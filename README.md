# Blink Frontend

The Next.js web application for Blink, a wallet-owned link shortener built on Stellar.

## Responsibilities

- Landing page and link creation experience
- Freighter wallet connection and transaction signing
- Soroban transaction simulation and submission
- Public `/{alias}` redirect route
- Blink brand assets and metadata

## Setup

```bash
npm install
cp .env.example .env.local
npm run dev
```

Required configuration:

```dotenv
NEXT_PUBLIC_STELLAR_NETWORK=testnet
NEXT_PUBLIC_STELLAR_RPC_URL=https://soroban-testnet.stellar.org
NEXT_PUBLIC_CONTRACT_ID=C...
NEXT_PUBLIC_APP_ORIGIN=http://localhost:3000
```

## Checks

```bash
npm run lint
npm run build
```

## Related repositories

- [blink-backend](https://github.com/blink-protocol/blink-backend)
- [blink-contracts](https://github.com/blink-protocol/blink-contracts)
- [blink-docs](https://github.com/blink-protocol/blink-docs)

