# AI Model Manifest

> Canonical registry of every AI model, SDK binding, deployment path, and containment boundary across the 360 Magicians / MBTQ ecosystem.

---

## Model Registry

| HuggingFace ID / Model Slug | Type | Source | License | Assigned Platform(s) |
|-----------------------------|------|--------|---------|----------------------|
| `meta-llama/Meta-Llama-3-8B-Instruct` | LLM | HF Hub / Groq / Ollama | Llama 3 Community | mbtq.dev, Quinn AI |
| `meta-llama/Meta-Llama-3-70B-Instruct` | LLM | Groq API | Llama 3 Community | 360 Magicians hub, agents |
| `mistralai/Mistral-7B-Instruct-v0.3` | LLM | HF Hub / vLLM / Ollama | Apache 2.0 | PinkSync microservices, fallback |
| `mistralai/Mixtral-8x7B-Instruct-v0.1` | LLM | Groq API | Apache 2.0 | Fast inference layer |
| `openai/gpt-4o` | LLM | OpenAI API | Proprietary | Quinn AI, GROW stage |
| `openai/gpt-4` | LLM | OpenAI API | Proprietary | Railway template, mbtq-dev |
| `anthropic/claude-3-5-sonnet` | LLM | Anthropic API | Proprietary | BUILD stage, agent review |
| `anthropic/claude-3-haiku` | LLM | Anthropic API | Proprietary | Fast agent tasks |
| `google/gemini-1.5-pro` | LLM | Vertex AI / AI Studio | Proprietary | 360 Magicians hub, mbtq.dev |
| `google/gemini-1.5-flash` | LLM | Vertex AI / AI Studio | Proprietary | Gemini CLI bridge, real-time |
| `openai/gpt-4o` (vision) | VLM | OpenAI API | Proprietary | Content generation, media |
| `google/gemini-1.5-pro` (vision) | VLM | Vertex AI | Proprietary | Accessibility AI, sign-lang |
| `openai/whisper-large-v3` | ASR | HF Hub / OpenAI API | MIT | DeafAuth ASL/audio input |
| `openai/whisper-medium` | ASR | HF Hub / Ollama | MIT | PinkSync audio pipeline |
| `openai/clip-vit-large-patch14` | VLM / Embedding | HF Hub | MIT | Sign-language recognition |
| `sentence-transformers/all-MiniLM-L6-v2` | Embedding | HF Hub | Apache 2.0 | Semantic search, RAG |
| `BAAI/bge-large-en-v1.5` | Embedding | HF Hub | MIT | DeafAuth preferences RAG |
| `stabilityai/stable-diffusion-xl-base-1.0` | Image-gen | HF Hub / Fal | CreativeML OpenRAIL-M | Media, Deaf content gen |
| `black-forest-labs/FLUX.1-schnell` | Image-gen | Fal API | Apache 2.0 | Content generation, rapid |
| `codellama/CodeLlama-13b-Instruct-hf` | Code-gen | HF Hub / Ollama | Llama 2 Community | BUILD stage fallback |

---

## Inference Modes

| Model | API (Hosted) | Local (Ollama/vLLM) | Serverless (Edge/Deno) | Containerized (K8s) |
|-------|:---:|:---:|:---:|:---:|
| Llama 3 8B | Groq | ✓ | — | ✓ |
| Llama 3 70B | Groq | — | — | ✓ |
| Mistral 7B | HF Serverless | ✓ | — | ✓ |
| Mixtral 8x7B | Groq | — | — | — |
| GPT-4o | OpenAI | — | ✓ (via edge proxy) | — |
| GPT-4 | OpenAI | — | — | — |
| Claude 3.5 Sonnet | Anthropic | — | — | — |
| Claude 3 Haiku | Anthropic | — | ✓ (via edge proxy) | — |
| Gemini 1.5 Pro | Vertex AI / AI Studio | — | ✓ (Deno) | — |
| Gemini 1.5 Flash | Vertex AI / AI Studio | — | ✓ (Deno) | — |
| Whisper Large v3 | OpenAI / HF Endpoint | ✓ | — | ✓ |
| CLIP ViT-L/14 | HF Serverless | — | — | ✓ |
| all-MiniLM-L6-v2 | HF Serverless | ✓ | — | ✓ |
| BGE Large | HF Endpoint | — | — | ✓ |
| SDXL | Fal / HF | — | — | ✓ |
| FLUX.1-schnell | Fal | — | — | — |
| CodeLlama 13B | HF Serverless | ✓ | — | ✓ |

---

## AI SDK Bindings

| SDK | Package | Models / Providers | Layer |
|-----|---------|-------------------|-------|
| **Vercel AI SDK** | `@ai-sdk/openai`, `@ai-sdk/anthropic`, `@ai-sdk/google` | GPT-4o, Claude, Gemini | Edge + Server (streaming) |
| **Google Generative AI** | `@google/generative-ai` | Gemini 1.5 Pro/Flash | Server, Deno |
| **OpenAI SDK** | `openai` | GPT-4, GPT-4o, Whisper | Server, agent tasks |
| **Anthropic SDK** | `@anthropic-ai/sdk` | Claude 3.5, Claude 3 Haiku | BUILD stage agents |
| **HuggingFace Inference** | `@huggingface/inference` | Llama 3, Mistral, CLIP, Whisper, embeddings | Server, microservices |
| **LangChain** | `langchain` / `@langchain/community` | All (via adapters) | Orchestration, RAG chains |
| **oRPC** | `@orpc/server`, `@orpc/client` | All (RPC wrapper) | Type-safe API layer |
| **Groq SDK** | `groq-sdk` | Llama 3 70B, Mixtral | Fast inference, agents |
| **Fal** | `@fal-ai/client` | SDXL, FLUX.1 | Image/video generation |

### HuggingFace `@huggingface/inference` — Usage Pattern

```typescript
import { HfInference } from "@huggingface/inference";

const hf = new HfInference(process.env.HF_TOKEN);

// Text generation (Mistral 7B)
const result = await hf.textGeneration({
  model: "mistralai/Mistral-7B-Instruct-v0.3",
  inputs: "<s>[INST] Your prompt here [/INST]",
  parameters: { max_new_tokens: 512 },
});

// Feature extraction / embeddings
const embedding = await hf.featureExtraction({
  model: "sentence-transformers/all-MiniLM-L6-v2",
  inputs: "Text to embed",
});

// Automatic speech recognition
const transcription = await hf.automaticSpeechRecognition({
  model: "openai/whisper-large-v3",
  data: audioBlob,
});
```

**Endpoint URL pattern:** `https://api-inference.huggingface.co/models/<model-id>`  
**HF Token scope:** `Read` for public models; `Read` + Inference Endpoints permission for dedicated endpoints.

### HF Serverless vs. Dedicated Endpoints

| | HF Serverless Inference | HF Dedicated Endpoint |
|-|------------------------|----------------------|
| **Use when** | Dev, staging, low-traffic | Production, SLA required |
| **Cold start** | Yes (~5–30s) | No (always warm) |
| **Rate limits** | Yes (free tier) | No (dedicated GPU) |
| **Cost** | Free (quota) | Per-hour GPU billing |
| **Auth** | HF_TOKEN | HF_TOKEN + endpoint URL |

### oRPC — Type-Safe AI Layer

oRPC wraps all AI provider SDKs into a single type-safe RPC contract:

```typescript
import { os } from "@orpc/server";

export const inferRouter = os.router({
  llm: os
    .input(InferLLMSchema)
    .handler(async ({ model, prompt, stream }) => {
      // route to correct SDK based on model prefix
    }),
  embed: os
    .input(InferEmbedSchema)
    .handler(async ({ model, input }) => { ... }),
});
```

---

## Per-Platform Model Assignment

| Platform | Primary Models | Secondary / Fallback | Purpose |
|----------|---------------|----------------------|---------|
| **360magicians.com hub** | Gemini 1.5 Pro, Llama 3 70B | Mistral 7B | Creative AI, content, design |
| **mbtq.dev** | Gemini 1.5 Flash, GPT-4o | Llama 3 8B (Ollama) | Deaf-first dashboard, v0 bridge |
| **PinkSync microservices** | Whisper Large, CLIP, Mistral 7B | all-MiniLM (embeddings) | Accessibility transforms, ASL |
| **DeafAuth** | BGE Large (embeddings), Whisper | Gemini Flash | Identity, preferences, ASR input |
| **Quinn AI** | Claude 3.5 Sonnet, GPT-4o | Llama 3 8B | Dev assistant, BUILD agent |
| **MBTQUniverse** | GPT-4o | Gemini 1.5 Pro | DAO summaries, governance |
| **FibonRose** | Claude 3 Haiku | GPT-4o | Task validation, UI generation |
| **Municipal DAO** | Gemini 1.5 Flash | Mixtral | Real-time governance analysis |
| **Local dev (all)** | Ollama: Llama 3 8B, Mistral 7B | — | No API cost in development |

---

## Capabilities Matrix

| Model | Text-gen | Vision/VLM | ASL/Sign | ASR | Embedding | Image-gen | Code-gen |
|-------|:--------:|:----------:|:--------:|:---:|:---------:|:---------:|:--------:|
| Llama 3 8B | ✓ | — | — | — | — | — | ✓ |
| Llama 3 70B | ✓ | — | — | — | — | — | ✓ |
| Mistral 7B | ✓ | — | — | — | — | — | ✓ |
| Mixtral 8x7B | ✓ | — | — | — | — | — | ✓ |
| GPT-4o | ✓ | ✓ | ✓ (via vision) | — | — | — | ✓ |
| GPT-4 | ✓ | — | — | — | — | — | ✓ |
| Claude 3.5 Sonnet | ✓ | ✓ | — | — | — | — | ✓ |
| Claude 3 Haiku | ✓ | ✓ | — | — | — | — | ✓ |
| Gemini 1.5 Pro | ✓ | ✓ | ✓ (via vision) | ✓ (native) | — | — | ✓ |
| Gemini 1.5 Flash | ✓ | ✓ | — | ✓ (native) | — | — | ✓ |
| Whisper Large v3 | — | — | — | ✓ | — | — | — |
| Whisper Medium | — | — | — | ✓ | — | — | — |
| CLIP ViT-L/14 | — | ✓ | ✓ | — | ✓ | — | — |
| all-MiniLM-L6-v2 | — | — | — | — | ✓ | — | — |
| BGE Large | — | — | — | — | ✓ | — | — |
| SDXL | — | — | — | — | — | ✓ | — |
| FLUX.1-schnell | — | — | — | — | — | ✓ | — |
| CodeLlama 13B | ✓ | — | — | — | — | — | ✓ |

---

## Deployment Paths

### Containerized (Docker / Kubernetes)

Model servers running as dedicated K8s Deployments in GKE (`mbtq.dev` project):

```yaml
# Example: Ollama model server pod spec
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama-llama3
  namespace: inference
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ollama-llama3
  template:
    spec:
      containers:
        - name: ollama
          image: gcr.io/mbtq-dev/ollama:llama3-8b
          ports:
            - containerPort: 11434
          resources:
            requests:
              nvidia.com/gpu: "1"
```

Container image tag strategy:
- `:dev` — local dev build
- `:staging-<git-sha>` — staging deploy
- `:prod-<git-sha>` — production release

GCR image paths: `gcr.io/mbtq-dev/<model-server>:<tag>`

### Virtual / Serverless

| Platform | Models | Notes |
|----------|--------|-------|
| **HF Spaces (Gradio)** | Mistral, Whisper, CLIP | Demo and staging, not SLA-bound |
| **HF Spaces (Streamlit)** | SDXL, FLUX | Visual demos, Deaf media previews |
| **Deno Deploy** | Gemini Flash (via API) | Edge inference proxy, <100ms TTFB |
| **Vercel Edge** | GPT-4o, Claude Haiku (via AI SDK) | Streaming chat, Next.js routes |
| **Railway** | Ollama sidecar container | Dev/staging quick deploys |

### Local Development

```yaml
# docker-compose.yml — Ollama sidecar
services:
  ollama:
    image: ollama/ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama

  app:
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
      AI_MODEL_OVERRIDE: llama3  # override prod model in local dev
```

`.env` model override flags:
```
AI_MODEL_OVERRIDE=llama3        # use local Ollama instead of OpenAI/Gemini
HF_TOKEN=hf_...                 # HuggingFace token for serverless inference
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_AI_API_KEY=AIza...
GROQ_API_KEY=gsk_...
FAL_KEY=...
```

---

## Containment Boundaries

| Scope | Isolated | Shared Pool | Notes |
|-------|:--------:|:-----------:|-------|
| DeafAuth identity models (BGE, Whisper) | ✓ | — | Dedicated namespace; no cross-service access |
| 360 Magicians hub (Gemini, Llama 70B) | — | ✓ | Shared inference pool via oRPC gateway |
| PinkSync accessibility transforms (CLIP, Whisper) | ✓ | — | PII: sign-language frames stay in PinkSync namespace |
| Quinn AI assistant (Claude, GPT-4o) | ✓ | — | Agent context isolated per session |
| MBTQUniverse DAO (GPT-4o, Gemini) | — | ✓ | Shared via inference gateway |
| Image generation (SDXL, FLUX) | ✓ | — | GPU workload isolated, rate-limited per user |
| Embeddings (MiniLM, BGE) | — | ✓ | Shared vector service across platforms |

### Isolation Rules

1. **PII audio/video** (DeafAuth ASR, PinkSync sign frames) → never leave isolated namespaces.
2. **API keys** → per-service GCP Secret Manager entries; no shared `.env` in production.
3. **Model versions** → pinned in each service's deployment manifest; no `latest` tags in prod.
4. **HF tokens** → scoped to minimum required permissions (`read` only unless endpoint creation needed).

---

## HuggingFace Spaces (Demo / Staging)

| Space | Model | Purpose | Environment |
|-------|-------|---------|-------------|
| `360magicians/asl-recognizer` | CLIP ViT-L/14 + Whisper | ASL sign demo | Demo/staging |
| `360magicians/deaf-tts-demo` | Bark / SpeechT5 | Text-to-speech Deaf UI | Demo |
| `mbtq-dev/pinksync-a11y` | Mistral 7B + MiniLM | Accessibility transform preview | Staging |
| `mbtq-dev/image-gen-demo` | SDXL / FLUX | Deaf media content generator | Demo |

> HF Spaces are **not** used for production traffic. Production routes through dedicated HF Endpoints or GKE-hosted model servers.

---

## Related

- [AI Inference Architecture](./ai-inference.md)
- [Providers, Vendors & Resources](./providers.md)
- [Infrastructure / Ecosystem Architecture](./infrastructure.md)
- [Package Managers & Runtimes](./package-managers.md)
- [OpenAPI Inference Paths](../openapi/openapi.yaml)
