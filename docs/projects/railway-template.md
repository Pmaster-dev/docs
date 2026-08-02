# Railway Next.js Template

Source: [`360magicians/railway_nextjs_with_shadcn`](https://github.com/360magicians/railway_nextjs_with_shadcn)

> Production-ready Next.js starter with Screaming Architecture, Server Actions, and zero-config Railway deployment.

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/nextjs-155-server-actions-with-shadcn?referralCode=fvqeo8)

---

## Key Features

- **Screaming Architecture** — Business logic organized by domain
- **Server Actions** — Zero API routes for auth; uses Next.js 15 server actions
- **Amber Minimal Theme** — Pre-configured shadcn/ui design system
- **Complete Auth Flows** — Signup, login, forgot/reset password, email verification
- **Railway-Native** — Health checks, graceful shutdown, zero-config deploy
- **Type Safety** — Zod runtime validation, TypeScript throughout

---

## Tech Stack

| Technology          | Version  | Purpose                  |
|---------------------|----------|--------------------------|
| Next.js             | 15.5.4   | React framework          |
| React               | 19.2.0   | UI library               |
| TypeScript          | 5.9.3    | Type safety              |
| Tailwind CSS        | 4.1.14   | Styling                  |
| shadcn/ui           | Latest   | Component library        |
| Zod                 | 4.1.11   | Runtime validation       |
| React Hook Form     | 7.64.0   | Form management          |
| bcryptjs            | 3.0.2    | Password hashing         |
| Sonner              | 2.0.7    | Toast notifications      |

---

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/             # Auth pages
│   ├── dashboard/          # Protected dashboard
│   └── api/health|system/  # Railway health checks
├── features/               # Business logic by domain
│   ├── auth/               # Authentication
│   │   ├── actions/        # Server actions (NO API routes)
│   │   ├── components/     # Auth UI
│   │   ├── lib/            # Utilities (password hashing)
│   │   └── types/          # Zod schemas & TS types
│   ├── dashboard/
│   ├── home/
│   └── health/
├── components/ui/          # shadcn/ui components
├── core/config/            # Environment validation
└── infrastructure/logging/ # Logging utilities
```

---

## Quick Start

```bash
# Clone and install
git clone https://github.com/360magicians/railway_nextjs_with_shadcn
cd railway_nextjs_with_shadcn
npm install

# Run dev server
npm run dev
```

Demo credentials: `demo@example.com` / `password123`  
Email verification code: `123456`

---

## Server Actions Pattern

```typescript
// 1. Shared Zod schema
export const signupSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword);

// 2. Server Action
'use server';
export async function signupAction(_prevState, formData: FormData) {
  const result = signupSchema.safeParse({ ... });
  if (!result.success) return { success: false, errors: result.error };
  // Hash password, create user, etc.
  return { success: true, message: 'Account created!' };
}

// 3. Client form hooks into the action
const [state, formAction] = useFormState(signupAction, null);
```

---

## Documentation

| Guide | Description |
|-------|-------------|
| [SETUP.md](https://github.com/360magicians/railway_nextjs_with_shadcn/blob/main/docs/SETUP.md) | Detailed setup instructions |
| [ARCHITECTURE.md](https://github.com/360magicians/railway_nextjs_with_shadcn/blob/main/docs/ARCHITECTURE.md) | Why it's structured this way |
| [SERVER_ACTIONS_GUIDE.md](https://github.com/360magicians/railway_nextjs_with_shadcn/blob/main/docs/SERVER_ACTIONS_GUIDE.md) | Complete server actions guide |
| [DEPLOYMENT.md](https://github.com/360magicians/railway_nextjs_with_shadcn/blob/main/docs/DEPLOYMENT.md) | Railway deployment guide |
| [MIGRATION_CHECKLIST.md](https://github.com/360magicians/railway_nextjs_with_shadcn/blob/main/docs/MIGRATION_CHECKLIST.md) | Step-by-step migration guide |
| [API_ROUTES_CLEANUP_GUIDE.md](https://github.com/360magicians/railway_nextjs_with_shadcn/blob/main/docs/API_ROUTES_CLEANUP_GUIDE.md) | API routes vs server actions |
| [CHANGELOG.md](https://github.com/360magicians/railway_nextjs_with_shadcn/blob/main/CHANGELOG.md) | Version history |

---

## Deployment (Railway)

1. Click the deploy button above
2. Set environment variables (optional)
3. Deploy — Railway auto-detects Next.js via Railpack

Railway provides:
- Health checks on `/api/health`
- System monitoring on `/api/system`
- Graceful SIGTERM shutdown
- `HOSTNAME=0.0.0.0` for container networking
