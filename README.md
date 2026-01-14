This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.tsx`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Cloudflare Architecture

This project is built to run on Cloudflare's edge infrastructure:

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Request                              │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Cloudflare Pages (Frontend)                    │
│                     Next.js Application                          │
│              Serves UI, handles client routing                   │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│               Cloudflare Workflows (Orchestrator)                │
│                       TripWorkflow                               │
│    ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│    │ generate-   │→ │ fetch-      │→ │ curate-     │→ save      │
│    │ skeleton    │  │ options     │  │ itinerary   │            │
│    └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
┌───────────────────────────┐   ┌───────────────────────────────┐
│   Workers AI (Llama 3.3)  │   │     Cloudflare KV (State)     │
│                           │   │                               │
│ • Skeleton generation     │   │ • Trip plan caching           │
│ • Itinerary curation      │   │ • Persistent edits            │
│ • JSON structured output  │   │ • 7-day TTL                   │
└───────────────────────────┘   └───────────────────────────────┘
```

### Components

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Frontend** | Cloudflare Pages + Next.js | Server-side rendered React app with edge routing |
| **Orchestrator** | Cloudflare Workflows | Durable, retriable multi-step trip planning |
| **AI** | Workers AI (Llama 3.3 70B) | On-edge AI inference for skeleton and curation |
| **State** | Cloudflare KV | Key-value storage for trip plans and edits |

### API Routes

- `POST /api/plan-trip` - Triggers the TripWorkflow, returns `tripId`
- `GET /api/poll-trip?tripId=...` - Polls for completed trip plan
- `POST /api/update-trip` - Persists edits to KV

### Configuration

See `wrangler.toml` for Cloudflare bindings:
- `TRIP_CACHE` - KV namespace for caching
- `AI` - Workers AI binding
- `TRIP_WORKFLOW` - Workflow binding

## Deploy on Cloudflare

```bash
# Build for Cloudflare Pages
npm run build:cloudflare

# Preview locally
npm run preview

# Deploy (requires wrangler login)
wrangler pages deploy .vercel/output/static
```
