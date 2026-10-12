# Blink Frontend

Next.js application for **Blink** — wallet-owned short links on Stellar (Soroban).

Users create aliases with Freighter, prove ownership on-chain, and share portable `/{alias}` URLs. Destinations can change later without changing the public link.

| | |
| --- | --- |
| **Stack** | Next.js 16, React 19, TypeScript, Stellar SDK, Freighter |
| **Networks** | `testnet` (default) · `public` |
| **Related** | [blink-backend](https://github.com/blink-protocol/blink-backend) · [blink-contracts](https://github.com/blink-protocol/blink-contracts) |
| **Live** | [blink-frontend-lovat.vercel.app](https://blink-frontend-lovat.vercel.app) |

---

## What this app does

1. **Create** — Validate alias + destination, connect Freighter, simulate/sign/submit a Soroban `create`.
2. **Prove** — Surface a success receipt with transaction hash and StellarExpert link.
3. **Resolve** — Serve public `GET /{alias}` redirects for visitors (today via direct RPC; prefer the backend resolver in production).
4. **Brand** — Landing experience, product narrative, and Blink assets under `public/brand/`.

Mutations always require the end-user wallet. This app never holds private keys and never submits unsigned create transactions on behalf of users.

---

## Architecture

```text
Browser
  ├─ CreateLinkForm  → Freighter (sign) → Stellar RPC (simulate / submit)
  └─ /{alias}        → resolve (RPC today) → 307 redirect | missing UX

Env
  NEXT_PUBLIC_STELLAR_NETWORK     → passphrase (testnet | public)
  NEXT_PUBLIC_STELLAR_RPC_URL     → Soroban RPC
  NEXT_PUBLIC_CONTRACT_ID         → LinkRegistry contract
  NEXT_PUBLIC_APP_ORIGIN          → short-link preview host
```

| Path | Role |
| --- | --- |
| `app/page.tsx` | Marketing + create form |
| `app/[alias]/route.ts` | Public redirect handler |
| `components/create-link-form.tsx` | Wallet create flow |
| `lib/stellar.ts` | Network passphrase, build/submit/resolve helpers |

---

## Prerequisites

- Node.js **20+** (22 LTS recommended)
- npm 10+
- [Freighter](https://www.freighter.app/) for create flows
- A deployed **LinkRegistry** contract id (`C…`) on the target network  
  → see [blink-contracts](https://github.com/blink-protocol/blink-contracts)

---

## Quick start

```bash
git clone https://github.com/blink-protocol/blink-frontend.git
cd blink-frontend
npm install
cp .env.example .env.local
# fill NEXT_PUBLIC_CONTRACT_ID (and network/RPC if not using testnet defaults)
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Configuration

Copy `.env.example` → `.env.local`. All `NEXT_PUBLIC_*` values are embedded in the client bundle — **never** put secrets there.

| Variable | Required | Description |
| --- | --- | --- |
| `NEXT_PUBLIC_STELLAR_NETWORK` | yes | `testnet` or `public` (also accepts `mainnet` → public passphrase) |
| `NEXT_PUBLIC_STELLAR_RPC_URL` | yes | Soroban RPC HTTP endpoint |
| `NEXT_PUBLIC_CONTRACT_ID` | yes | Deployed LinkRegistry contract id (`C…`) |
| `NEXT_PUBLIC_APP_ORIGIN` | yes | Public origin used in short-link previews (no trailing slash) |

### Network passphrase

`lib/stellar.ts` exposes `getNetworkPassphrase()`. Create/sign/submit/resolve paths call it at use-time so unknown or empty network values **fail closed** before Freighter opens.

| Config value | Passphrase |
| --- | --- |
| `testnet` | Test SDF Network |
| `public` / `mainnet` | Public Global Stellar Network |

### Production checklist

- [ ] `NEXT_PUBLIC_STELLAR_NETWORK=public` only when the contract is on public
- [ ] RPC URL matches the network (do not mix testnet RPC with public passphrase)
- [ ] `NEXT_PUBLIC_APP_ORIGIN` is the real HTTPS origin (affects preview URLs)
- [ ] Contract id is the audited/intended deployment — not a scratch deploy
- [ ] Prefer routing `/{alias}` through [blink-backend](https://github.com/blink-protocol/blink-backend) for cache, rate limits, and stable errors

---

## Scripts

```bash
npm run dev      # local development
npm run lint     # ESLint
npm run build    # production build
npm run start    # serve .next (after build)
```

---

## Alias & destination rules (client)

These must stay aligned with the contract and backend:

| Rule | Value |
| --- | --- |
| Alias | `^[a-z0-9-]{3,32}$` |
| Destination | `http:` or `https:` URL |

Invalid input is rejected in the form **before** wallet prompts.

---

## Create flow (operator view)

1. Format validation (alias + destination)
2. Freighter `isConnected` / `requestAccess` / `getAddress`
3. `buildCreateTransaction` — simulate + assemble XDR with configured passphrase
4. `signTransaction` with the same passphrase
5. `submitTransaction` — poll until SUCCESS / FAILED / timeout
6. Success copy includes truncated hash + StellarExpert URL

Failure modes the UI must keep distinct over time: missing extension, rejected access/sign, alias taken, bad destination, misconfigured env, RPC/timeout.

---

## Security notes

- **No custodial keys** — signing is Freighter-only in the current release.
- **Network binding** — passphrase comes from deployment config; do not hardcode a second network in UI copy without matching env.
- **Public env** — treat all `NEXT_PUBLIC_*` as world-readable.
- **Redirects** — never send visitors to non-http(s) destinations; backend additionally rejects unsafe schemes with `422 unsafe_destination`.
- **XSS** — if echoing `?missing=` aliases into HTML, keep encoding strict (alias charset is already constrained).

This software has **not** received an independent security audit. Use public mainnet at your own risk.

---

## Deployment (Vercel)

1. Import `blink-protocol/blink-frontend`.
2. Set the four `NEXT_PUBLIC_*` variables for the target environment.
3. Build command: `npm run build` · Output: Next default.
4. Confirm Freighter network matches the deployment network after go-live.

`vercel.json` may be present for project defaults; prefer dashboard env for secrets-free public config.

---

## Repository map

```text
app/
  page.tsx              Landing + create
  [alias]/route.ts     Public redirect
  layout.tsx            Root layout / metadata
components/
  create-link-form.tsx  Freighter create UX
lib/
  stellar.ts            RPC + passphrase + contract calls
public/brand/           Logos and marks
```

---

## Contributing

1. Wait for maintainer assignment on Wave issues before starting work.
2. Branch from `main`: `feat/<short-slug>`.
3. Keep PRs scoped; include `Closes #<n>` when resolving an issue.
4. Run `npm run lint` and `npm run build` before requesting review.

Open issues: [github.com/blink-protocol/blink-frontend/issues](https://github.com/blink-protocol/blink-frontend/issues)

---

## License & status

Blink is under active development. APIs and UX may change between Wave cycles. See sibling repos for resolver API and on-chain policy source of truth.
