# AI Inference Architecture

> Inference server options, routing strategy, fallback chains, and latency tiers for the 360 Magicians / MBTQ ecosystem.

---

## Inference Server Options

| Server | Type | Best For | GPU Required | Supported Models |
|--------|------|----------|:------------:|-----------------|
| **vLLM** | Self-hosted | High-throughput production LLM serving | ✓ | Llama 3, Mistral, Mixtral, CodeLlama |
| **Ollama** | Self-hosted | Local dev, low-config single-GPU | Optional | Llama 3, Mistral, CodeLlama, Whisper |
| **TGI (Text Generation Inference)** | Self-hosted / HF | Production HF models, continuous batching | ✓ | Llama 3, Mistral, Mixtral, BLOOM |
| **llama.cpp** | Self-hosted | CPU-only / edge inference, quantized models | — | GGUF-format models (Llama, Mistral, Phi) |
| **HF Inference API** | HF-hosted | Dev, staging, low-traffic | — | All HF Hub models |
| **HF Dedicated Endpoints** | HF-hosted | Production, SLA-bound | ✓ | All HF Hub models |
| **Groq API** | Cloud API | Ultra-low-latency LLM (LPU hardware) | — | Llama 3, Mixtral |
| **Vertex AI (Gemini)** | Cloud API | Google-integrated production workloads | — | Gemini 1.5 Pro/Flash |
| **OpenAI API** | Cloud API | GPT models, Whisper ASR | — | GPT-4, GPT-4o, Whisper |
| **Anthropic API** | Cloud API | Claude agents, code review | — | Claude 3.5, Claude 3 Haiku |
| **Fal** | Cloud API | Async image/video generation | — | SDXL, FLUX.1, video models |

---

## Inference Routing

Requests are routed to the appropriate inference tier based on **model requirement**, **latency SLA**, and **environment**:

```
Client Request
     │
     ▼
oRPC Inference Gateway  (api.360magicians.com/inference/*)
     │
     ├─ model prefix "gpt-*"      ──► OpenAI API
     ├─ model prefix "claude-*"   ──► Anthropic API
     ├─ model prefix "gemini-*"   ──► Vertex AI / AI Studio
     ├─ model prefix "groq/*"     ──► Groq API
     ├─ model prefix "fal/*"      ──► Fal API
     ├─ model prefix "hf/*"       ──► HF Inference API / Dedicated Endpoint
     └─ model prefix "local/*"    ──► Ollama (dev) / vLLM/TGI (prod K8s)
```

### Routing Rules by Stage

| Environment | LLM Route | Embedding Route | ASR Route | Image-gen Route |
|-------------|-----------|----------------|-----------|----------------|
| **Local dev** | Ollama (Llama 3 8B) | Ollama (MiniLM) | Ollama (Whisper) | HF Serverless (SDXL) |
| **Staging** | Groq (Llama 3 70B) | HF Serverless (BGE) | HF Serverless (Whisper) | Fal |
| **Production** | Vertex AI / OpenAI / Anthropic | HF Dedicated Endpoint (BGE) | HF Dedicated Endpoint (Whisper) | Fal |

### Per-Service Routing

| Service | Default LLM | Override Env Var |
|---------|------------|-----------------|
| 360magicians hub | `gemini-1.5-pro` | `HUB_LLM_MODEL` |
| mbtq.dev | `gemini-1.5-flash` | `MBTQ_LLM_MODEL` |
| PinkSync | `mistral-7b-instruct` | `PINKSYNC_LLM_MODEL` |
| DeafAuth | `gemini-1.5-flash` | `DEAFAUTH_LLM_MODEL` |
| Quinn AI | `claude-3-5-sonnet` | `QUINN_LLM_MODEL` |
| Local dev (all) | `AI_MODEL_OVERRIDE` | `AI_MODEL_OVERRIDE` |

---

## Fallback Chains

Fallback order is enforced by the oRPC inference gateway. If a provider returns an error or exceeds quota, the next provider in the chain is tried.

### LLM Fallback (Text Generation)

```
Production:  Vertex AI (Gemini 1.5 Pro)  →  OpenAI (GPT-4o)  →  Groq (Llama 3 70B)  →  HF Dedicated (Mistral 7B)
Staging:     Groq (Llama 3 70B)          →  HF Serverless (Mistral 7B)
Local dev:   Ollama (Llama 3 8B)         →  HF Serverless (Mistral 7B)
```

### VLM Fallback (Vision / Multimodal)

```
Production:  Vertex AI (Gemini 1.5 Pro)  →  OpenAI (GPT-4o vision)  →  HF (CLIP + LLM)
```

### ASR Fallback (Speech-to-Text)

```
Production:  OpenAI Whisper API          →  HF Dedicated (Whisper Large v3)  →  HF Serverless (Whisper Medium)
Local dev:   Ollama (Whisper)            →  HF Serverless (Whisper Medium)
```

### Embedding Fallback

```
Production:  HF Dedicated (BGE Large)   →  HF Serverless (MiniLM)
Local dev:   Ollama (nomic-embed-text)   →  HF Serverless (MiniLM)
```

### Image Generation Fallback

```
Production:  Fal (FLUX.1-schnell)       →  Fal (SDXL)  →  HF Serverless (SDXL)
```

---

## Latency Tiers

| Tier | Target Latency | Use Cases | Providers |
|------|---------------|-----------|-----------|
| **Real-time** | < 500 ms (TTFB) | Streaming chat, live captions, sign-lang overlay | Groq, Gemini Flash (Deno Edge), Claude Haiku (Vercel Edge) |
| **Interactive** | 500 ms – 3 s | UI generation, idea drafts, code suggestions | Gemini 1.5 Pro, GPT-4o, Vertex AI |
| **Batch** | 3 s – 30 s | Market research, compliance reports, long-form | OpenAI, Anthropic, Vertex AI (batch mode) |
| **Async / Background** | > 30 s (queued) | Image generation, full report generation, model fine-tuning | Fal, HF Spaces, GKE batch jobs |

### Streaming

All real-time and interactive endpoints support **Server-Sent Events (SSE)** streaming via the Vercel AI SDK `streamText` / `streamObject` helpers, or the `ReadableStream` Deno API:

```typescript
// Next.js Edge Route (Vercel AI SDK)
import { streamText } from "ai";
import { openai } from "@ai-sdk/openai";

export const runtime = "edge";

export async function POST(req: Request) {
  const { prompt } = await req.json();
  const result = await streamText({
    model: openai("gpt-4o"),
    prompt,
  });
  return result.toDataStreamResponse();
}
```

```typescript
// Deno Edge (Gemini Flash via @google/generative-ai)
import { GoogleGenerativeAI } from "npm:@google/generative-ai";

const genAI = new GoogleGenerativeAI(Deno.env.get("GOOGLE_AI_API_KEY")!);
const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

const result = await model.generateContentStream(prompt);
for await (const chunk of result.stream) {
  // stream to client
}
```

---

## vLLM Production Setup (GKE)

For high-throughput production inference of open-source models:

```bash
# Deploy vLLM server for Mistral 7B
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vllm-mistral-7b
  namespace: inference
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vllm-mistral-7b
  template:
    metadata:
      labels:
        app: vllm-mistral-7b
    spec:
      containers:
        - name: vllm
          image: vllm/vllm-openai:latest
          args:
            - "--model"
            - "mistralai/Mistral-7B-Instruct-v0.3"
            - "--port"
            - "8000"
          ports:
            - containerPort: 8000
          resources:
            limits:
              nvidia.com/gpu: "1"
          env:
            - name: HUGGING_FACE_HUB_TOKEN
              valueFrom:
                secretKeyRef:
                  name: hf-token
                  key: token
EOF
```

vLLM exposes an OpenAI-compatible API at `/v1/completions` and `/v1/chat/completions`, so any `openai`-SDK-compatible client can target it by setting `OPENAI_BASE_URL`.

---

## TGI (Text Generation Inference) Setup

For HF-native model serving with continuous batching:

```bash
docker run --gpus all \
  -e HUGGING_FACE_HUB_TOKEN=$HF_TOKEN \
  -p 8080:80 \
  ghcr.io/huggingface/text-generation-inference:latest \
  --model-id mistralai/Mistral-7B-Instruct-v0.3 \
  --max-concurrent-requests 128
```

TGI endpoint: `POST http://localhost:8080/generate`

---

## Ollama Local Dev Setup

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull models
ollama pull llama3
ollama pull mistral
ollama pull nomic-embed-text  # embeddings
ollama pull whisper            # ASR

# Start server (default: http://localhost:11434)
ollama serve
```

Set `AI_MODEL_OVERRIDE=llama3` and `OLLAMA_BASE_URL=http://localhost:11434` in `.env.local` to redirect all inference calls to Ollama in development.

---

## Related

- [AI Model Manifest](./ai-model-manifest.md)
- [Providers, Vendors & Resources](./providers.md)
- [Environments (dev → prod)](./environments.md)
- [OpenAPI Inference Paths](../openapi/openapi.yaml)
