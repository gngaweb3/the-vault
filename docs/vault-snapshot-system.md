# Vault Snapshot System

**Repository:** `gngaweb3/the-vault`
**Files:** `scripts/snapshot.js` · `.github/workflows/snapshot.yml`
**Last updated:** July 5, 2026

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture](#2-architecture)
3. [Repository Structure](#3-repository-structure)
4. [How It Works](#4-how-it-works)
5. [Snapshot Script — scripts/snapshot.js](#5-snapshot-script--scriptssnapshotjs)
6. [Scope — What Is and Isn't Captured](#6-scope--what-is-and-isnt-captured)
7. [Database — Supabase](#7-database--supabase)
8. [GitHub Actions Workflow](#8-github-actions-workflow)
9. [Automation — cron-job.org](#9-automation--cron-joborg)
10. [Data Volume](#10-data-volume)
11. [Security Overview](#11-security-overview)
12. [Maintenance](#12-maintenance)
13. [Independent Verification](#13-independent-verification)

---

## 1. Overview

The Vault Snapshot System captures and stores hourly snapshots of The Vault's economic value. It is the data backbone that powers the Treasury Evolution Chart on the GNGA.WEB3 Protocol dashboard.

The system runs fully autonomously every hour without human intervention, reading directly from the blockchain and storing verifiable on-chain data in Supabase. The snapshot methodology is fully aligned with the live Vault Monitor Dashboard, including idle USDC held across Master Vaults and Sub-Vaults.

---

## 2. Architecture

```
cron-job.org (hourly trigger at :00)
        │
        ▼
GitHub Actions (gngaweb3/the-vault)
        │  runs scripts/snapshot.js
        ▼
Blockchain RPCs + CoinGecko API
(same logic and scope as the live Vault Monitor Dashboard)
        │
        ▼
Supabase → table: treasury_snapshots
        │
        ▼
Treasury Evolution Chart (Framer)
reads Supabase and renders historical chart
```

---

## 3. Repository Structure

```
the-vault/
├── scripts/
│   └── snapshot.js          ← Core snapshot logic (Node.js)
├── .github/
│   └── workflows/
│       └── snapshot.yml     ← GitHub Actions workflow
├── package.json             ← Dependencies (ethers.js 5.7.2)
└── docs/
    ├── vault-snapshot-system.md    ← This document
    └── vault-monitor-dashboard.md
```

---

## 4. How It Works

Every hour, cron-job.org sends an HTTP POST to the GitHub Actions API, which triggers `snapshot.js`. The script:

1. Fetches real-time prices from CoinGecko for all Vault assets, including USDC.
2. Reads on-chain balances from all Safe wallets across 4 networks using public RPCs — Master Vault and all 4 Sub-Vaults.
3. Calculates `base_assets_usd`, `yield_usd`, and `treasury_total_usd`.
4. Inserts a new row into the `treasury_snapshots` table in Supabase.

The calculation logic is identical in scope to the live Vault Monitor Dashboard, ensuring full consistency between real-time and historical data.

---

## 5. Snapshot Script — scripts/snapshot.js

### Assets calculated

**Base Assets** (read from the Master Vault on each network):

| Asset | Network | Method |
|---|---|---|
| PAXG | Ethereum | `balanceOf(masterVault)` on ERC-20 contract |
| QNT | Ethereum | `balanceOf(masterVault)` on ERC-20 contract |
| LINK | Polygon | `balanceOf(masterVault)` on ERC-20 contract |
| POL | Polygon | `getBalance(masterVault)` — native token |
| WXRP | BNB Chain | `balanceOf(masterVault)` on ERC-20 contract |
| HYPE | HyperEVM | `getBalance(masterVault)` — native token |
| USDC | Ethereum, Polygon, BNB Chain, HyperEVM | `balanceOf(masterVault)` on each network's USDC contract, summed |

**Yield Positions** (read from Sub-Vaults):

| Asset | Network | Method |
|---|---|---|
| stPOL | Ethereum | `balanceOf(subEthereum)` on stPOL contract |
| vXRP | BNB Chain | `balanceOf(subBnb) × exchangeRateStored() / 10^36` on Venus vXRP contract |
| USDC | Ethereum, Polygon, BNB Chain, HyperEVM | `balanceOf(subVault)` on each network's USDC contract, summed across all 4 Sub-Vaults |

### Calculation

```javascript
usdc_master_usd =
  (usdc_eth_master  × 1) +
  (usdc_pol_master  × 1) +
  (usdc_bnb_master  × 1) +
  (usdc_hype_master × 1)

usdc_sub_usd =
  (usdc_eth_sub  × 1) +
  (usdc_pol_sub  × 1) +
  (usdc_bnb_sub  × 1) +
  (usdc_hype_sub × 1)

base_assets_usd =
  (paxg  × paxg_price)  +
  (qnt   × qnt_price)   +
  (link  × link_price)  +
  (pol   × pol_price)   +
  (wxrp  × xrp_price)   +
  (hype  × hype_price)  +
  usdc_master_usd

yield_usd =
  (stpol × spol_price) +
  (vxrp  × xrp_price) +
  usdc_sub_usd

treasury_total_usd = base_assets_usd + yield_usd
```

### Dependencies

```json
{
  "dependencies": {
    "ethers": "5.7.2"
  }
}
```

ethers.js is required for `BigNumber` arithmetic when calculating the Venus vXRP position — prevents precision loss with large integers from `exchangeRateStored()`.

### RPCs used

| Network | RPC |
|---|---|
| Ethereum | `https://ethereum-rpc.publicnode.com` |
| Polygon | `https://polygon-bor-rpc.publicnode.com` |
| BNB Chain | `https://bsc-rpc.publicnode.com` |
| HyperEVM | `https://rpc.hyperliquid.xyz/evm` |

All public RPCs — no API key required.

---

## 6. Scope — What Is and Isn't Captured

The snapshot is designed to reflect the Vault's committed economic value — the assets and positions that make up `treasury_total_usd` in the Vault Monitor Dashboard. Two categories are intentionally excluded, matching the dashboard's own methodology:

| Category | Included in snapshot? | Rationale |
|---|---|---|
| Base Assets (Master Vault) | ✅ Yes | Core reserve holdings of the protocol |
| Idle USDC — Master Vaults | ✅ Yes | Capital in transit before being deployed to a Sub-Vault or swapped into a base asset; economically part of the Vault |
| Yield Positions (Sub-Vaults) | ✅ Yes | Active, yield-generating positions |
| Idle USDC — Sub-Vaults | ✅ Yes | Capital in transit before being deployed into a yield position |
| Gas reserves (ETH, BNB held for transaction fees) | ❌ No | Operational float, not treasury value |
| Residual Assets (dust balances pending accumulation) | ❌ No | Below deployment threshold, not yet an active position |

This scope is intentional and mirrors the live dashboard exactly, so that `treasury_total_usd` in Supabase always matches what a user sees on the Vault Monitor Dashboard at the moment of capture.

---

## 7. Database — Supabase

**Table:** `treasury_snapshots`

```sql
CREATE TABLE treasury_snapshots (
  id                  bigserial PRIMARY KEY,
  captured_at         timestamptz NOT NULL DEFAULT now(),
  treasury_total_usd  float8 NOT NULL,
  base_assets_usd     float8 NOT NULL,
  yield_usd           float8 NOT NULL
);

CREATE INDEX idx_treasury_snapshots_captured_at
  ON treasury_snapshots (captured_at DESC);
```

No schema changes were required — the USDC balances are absorbed into the existing `base_assets_usd` and `yield_usd` columns, consistent with how the Vault Monitor Dashboard reports them.

### Row Level Security

```sql
CREATE POLICY "Allow insert from service"
ON treasury_snapshots
FOR INSERT
WITH CHECK (auth.role() = 'service_role');
```

| Key | Permission | Used by |
|---|---|---|
| `anon` | SELECT only (read) | Treasury Evolution Chart in Framer |
| `service_role` | INSERT (write) | GitHub Actions snapshot script |

---

## 8. GitHub Actions Workflow

The workflow is triggered **exclusively** by cron-job.org via `workflow_dispatch`. The native GitHub Actions cron scheduler is intentionally disabled — it caused irregular execution timing with delays of up to several hours.

```yaml
name: Treasury Snapshot

on:
  workflow_dispatch:

jobs:
  snapshot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4.2.2
      - run: npm install
      - run: node scripts/snapshot.js
        env:
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_SERVICE_KEY: ${{ secrets.SUPABASE_SERVICE_KEY }}
```

### GitHub Secrets

| Secret | Description |
|---|---|
| `SUPABASE_URL` | Supabase project URL (`https://xxxx.supabase.co`) |
| `SUPABASE_SERVICE_KEY` | Supabase `service_role` key (write permissions only) |

---

## 9. Automation — cron-job.org

cron-job.org guarantees exact hourly execution (1:00, 2:00, 3:00 AM UTC...), producing uniform data points in the chart. The GitHub native cron was replaced with this after testing showed irregular timing.

| Field | Value |
|---|---|
| Title | `Treasury Snapshot` |
| URL | `https://api.github.com/repos/gngaweb3/the-vault/actions/workflows/snapshot.yml/dispatches` |
| Method | `POST` |
| Header `Authorization` | `Bearer ghp_YOUR_TOKEN` |
| Header `Accept` | `application/vnd.github.v3+json` |
| Body | `{"ref":"main"}` |

> ⚠️ **Action required in 2027:** The GitHub Personal Access Token (scope: `workflow`) expires in 1 year. Generate a new one at GitHub → Settings → Developer settings → Personal access tokens, and update it in cron-job.org before expiration.

---

## 10. Data Volume

| Filter | Approximate snapshots |
|---|---|
| 1 Day | ~24 |
| 7 Days | ~168 |
| 30 Days | ~720 |
| 90 Days | ~2,160 |
| 1 Year | ~8,760 |

---

## 11. Security Overview

| Element | Status | Notes |
|---|---|---|
| RLS on Supabase | ✅ Active | Only `service_role` can insert |
| Service key in GitHub Secrets | ✅ | Never exposed in code |
| Anon key in Framer | ✅ | Read-only, no write access |
| Personal Access Token | ✅ | Expires 2027 — renew before expiration |
| Repository visibility | ✅ Public | Code fully auditable — no sensitive data in codebase |

---

## 12. Maintenance

### Normal operation

The system is fully autonomous. No manual intervention required under normal conditions.

### When to intervene

| Situation | Action |
|---|---|
| Chart shows no new data points | Check cron-job.org execution history and GitHub Actions logs |
| GitHub Actions workflow failing | Review logs in the Actions tab of this repository |
| CoinGecko rate limit error | Script will fail that snapshot and resume automatically next hour |
| Public RPC temporarily down | Script retries next hour — public RPCs have high uptime |
| Personal Access Token expired | Generate new token with `workflow` scope, update in cron-job.org |
| New yield position added | Update `scripts/snapshot.js` to include the new position in `yield_usd` |
| New Sub-Vault or Master Vault network added | Update `scripts/snapshot.js` to include its USDC balance in the corresponding total |

### Adding a new asset to the snapshot

Add the asset balance calculation in `scripts/snapshot.js` following the same pattern as existing assets. Then include it in the `base_assets_usd` or `yield_usd` totals before the Supabase insert.

---

## 13. Independent Verification

All snapshot data can be independently verified:

- **On-chain balances** — Etherscan, Polygonscan, BscScan, HyperEVM Explorer
- **Asset prices** — CoinGecko public API
- **Script logic** — `scripts/snapshot.js` in this repository (open source)
- **Stored snapshots** — readable via Supabase anon key (SELECT only)

Snapshot values must match the live Vault Monitor Dashboard values at the time of capture (±minor price movement within the same minute of execution).

---

*GNGA.WEB3 Protocol — Vault Snapshot System*
