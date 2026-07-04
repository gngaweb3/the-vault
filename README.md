# the-vault

On-chain transparency layer for **The Vault** — the programmatic, multi-network container that holds the backing assets of the [GNGA.WEB3 Protocol](https://github.com/gngaweb3).

This repository does not contain protocol smart contracts. It contains the automation that reads The Vault's on-chain state and makes it publicly verifiable in real time: a live monitoring dashboard and an hourly historical snapshot pipeline.

For the protocol overview, tokenomics, and the Economic Value Floor (EVF) model, see the [organization profile](https://github.com/gngaweb3) and the [Whitepaper](https://gnga.tech/gngapaper).

---

## What's in this repo

```
the-vault/
├── scripts/
│   └── snapshot.js          # Hourly on-chain snapshot of The Vault's value
├── .github/
│   └── workflows/
│       └── snapshot.yml     # GitHub Actions automation (triggered by cron-job.org)
├── docs/
│   ├── vault-monitor-dashboard.md   # Live dashboard: architecture & calculation logic
│   └── vault-snapshot-system.md     # Historical snapshot pipeline: architecture & schema
├── package.json
├── SECURITY.md
├── LICENSE
└── README.md
```

| Component | What it does | Docs |
|---|---|---|
| **Vault Monitor Dashboard** | Real-time on-chain read of every asset in The Vault, no database | [`docs/vault-monitor-dashboard.md`](./docs/vault-monitor-dashboard.md) |
| **Vault Snapshot System** | Hourly historical capture, powers the Treasury Evolution Chart | [`docs/vault-snapshot-system.md`](./docs/vault-snapshot-system.md) |

Both read directly from public RPCs and CoinGecko — no proprietary data source, no manual entry.

---

## The Vault, in short

- **Multi-network by design** — backing assets are held across Ethereum, Polygon, BNB Chain, and HyperEVM, all under a Master Vault with dedicated Sub-Vaults per yield strategy.
- **No profit distribution** — The Vault does not pay dividends or yield to holders. Every flow it captures is recycled into buying more backing assets or buying back and burning $GNGA.
- **Programmatically restricted** — outgoing transfers are limited by code (Safe + Zodiac Scope Module) to a whitelist of authorized contracts. No path exists to a private wallet.
- **Fully auditable** — every capital movement is cryptographically sealed on Hedera, and every balance can be independently verified on-chain by anyone, at any time.

Full mechanics in the [Whitepaper](https://gnga.tech/gngapaper).

---

## Running the snapshot script locally

```bash
npm install
SUPABASE_URL=https://xxxx.supabase.co \
SUPABASE_SERVICE_KEY=your_service_role_key \
node scripts/snapshot.js
```

Requires no API keys beyond Supabase — all blockchain reads use public RPCs, and pricing uses the public CoinGecko API.

---

## Security

Found a vulnerability? Please **do not** open a public issue. See [`SECURITY.md`](./SECURITY.md) for responsible disclosure.

## License

This repository is released under the [MIT License](./LICENSE).

---

*GNGA.WEB3 Protocol*
