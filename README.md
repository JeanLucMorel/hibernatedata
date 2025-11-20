## Ignition Protocol

https://pump.fun/coin/6GyELfnEX6HAVgYWuvB1ZpmQzrF3PhafSBvJnC2Zpump

Solana-backed data labeling exchange where requesters upload CSVs, fund escrow accounts, and mobile-first workers submit entry-by-entry labels. Amplify Storage stores payloads, Upstash Redis tracks task state, and payment events run through custodial wallets managed on-chain.

### Stack

- **Next.js 16 / App Router** with Tailwind v4 design tokens
- **AWS Amplify** (file uploads + signed reads)
- **Upstash Redis** (tasks, submissions, disputes, wallet metadata)
- **Solana Web3** (custodial wallets, escrow actions)
- **Vitest** for unit tests

## Getting Started

1. **Install dependencies**
   ```bash
   npm install
   ```

2. **Copy environment template**
   ```bash
   cp env.example .env.local
   ```
   Fill the variables (see table below).

3. **Run the dev server**
   ```bash
   npm run dev
   ```
   Navigate to `http://localhost:3000`.

4. **Lint & test**
   ```bash
   npm run lint
   npm test
   ```

### Environment Variables

| Variable | Description |
| --- | --- |
| `NEXT_PUBLIC_AWS_REGION` | Amplify identity pool + bucket region |
| `NEXT_PUBLIC_AMPLIFY_IDENTITY_POOL_ID` | Cognito identity pool for anonymous Storage access |
| `NEXT_PUBLIC_AMPLIFY_FILE_BUCKET` | S3 bucket used by Amplify Storage |
| `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` | Redis REST endpoint + token |
| `SOLANA_RPC_ENDPOINT` | RPC endpoint (mainnet or devnet) |
| `SOLANA_ESCROW_PROGRAM_ID` | Program controlling escrow logic (for reference) |
| `PLATFORM_CUSTODY_SECRET_KEY` | Base58/JSON secret for platform custody wallet |
| `NEXT_PUBLIC_SOLANA_CLUSTER` | Cluster label (e.g. `mainnet-beta`) |
| `NEXT_PUBLIC_PLATFORM_FEE_BPS` | Platform fee basis points (client display) |
| `REQUESTER_WEBHOOK_SECRET` | Shared secret for outbound hooks (future use) |
| `ADMIN_API_KEY` | Salt for wallet derivation + privileged APIs |

Refer to `env.example` for a full list.

## Key Commands

| Command | Action |
| --- | --- |
| `npm run dev` | Launch Next.js dev server |
| `npm run build && npm start` | Production build & serve |
| `npm run lint` | ESLint across the repo |
| `npm test` | Vitest unit tests (`tests/`) |

## Architecture Notes

- `app/(dashboard)` contains requester, worker, and admin surfaces. Routes stay mobile-first with reusable cards + buttons in `components/ui`.
- `app/(workspace)/tasks/[taskId]` is the worker labeling surface. It streams CSV rows from Amplify, tracks per-entry labels, and saves drafts/submissions through `/api/submissions`.
- `app/api/**` exposes Next.js route handlers for tasks, submissions, disputes, wallets, and payments.
- `lib/redisTasks.ts` + `lib/disputes.ts` encapsulate Upstash persistence. All escrow events are tracked in `lib/solanaEscrow.ts`.
- `components/requester/escrow-card.tsx` hands requesters a custodial wallet + button to fund the Solana escrow, while approvals invoke `/api/payments` to release lamports to workers.

## Flows

1. **Requester**
   - Upload CSV via Amplify (`/requester/upload`)
   - Fund escrow from custodial wallet (`/requester/review/[taskId]`)
   - Review submissions, approve (releases escrow) or open disputes

2. **Worker**
   - Browse open tasks (`/worker`)
   - Label entries in the workspace (`/tasks/[taskId]`)
   - Track disputes + wallet address inside worker dashboards

3. **Admin**
   - Resolve disputes and trigger refunds/releases from `/admin/disputes`

Feel free to extend the API routes or UI sections following the existing patterns. All state mutations live server-side to keep wallets and escrow actions authoritative.
