# Vault Monitor Dashboard

**Repository:** `gngaweb3/the-vault`
**File:** `TreasuryDashboard_v1_2.tsx`
**Platform:** Framer Code Component
**Version:** v1.0 · July 2026

---

## Table of Contents

1. [Overview](#1-overview)
2. [What It Displays](#2-what-it-displays)
3. [Vault Assets](#3-vault-assets)
4. [Idle USDC — Master Vaults & Sub-Vaults](#4-idle-usdc--master-vaults--sub-vaults)
5. [Safe Wallets — Vault Structure](#5-safe-wallets--vault-structure)
6. [Networks & RPCs](#6-networks--rpcs)
7. [Value Calculation](#7-value-calculation)
8. [Scope — What Counts Toward Total Vault Value](#8-scope--what-counts-toward-total-vault-value)
9. [Yield Positions — Technical Detail](#9-yield-positions--technical-detail)
10. [Original Deposits — Historical Constants](#10-original-deposits--historical-constants)
11. [Asset Prices — CoinGecko](#11-asset-prices--coingecko)
12. [Technical Dependencies](#12-technical-dependencies)
13. [Adding a New Base Asset](#13-adding-a-new-base-asset)
14. [Adding a New Yield Position](#14-adding-a-new-yield-position)
15. [Changelog](#15-changelog)
16. [Independent Verification](#16-independent-verification)

---

## 1. Overview

The Vault Monitor Dashboard is a real-time monitoring interface for The Vault of the GNGA.WEB3 Protocol. It reads all asset balances directly from the blockchain on every page load — no intermediaries, no databases, no manually entered data.

The dashboard reflects the current state of The Vault as defined in the GNGA.WEB3 Whitepaper: a multi-network, programmatic value container holding a basket of zero-derivative, on-chain verifiable assets. The dashboard accounts for idle USDC held in Master and Sub-Vaults — capital in transit between deposit and its final destination (a base asset swap or a yield position) — closing a gap where this capital was technically part of the Vault but was not yet reflected in the displayed totals.

---

## 2. What It Displays

| Section | Description |
|---|---|
| **Total Vault Value** | Combined USD value of all assets across all networks, including idle USDC |
| **Base Assets** | Fixed asset positions with real-time balance and USD value, including idle USDC in Master Vaults |
| **Ecosystem Value Positions** | Active yield-generating positions with current balance, including idle USDC in Sub-Vaults |
| **Value Generated** | Per-position yield breakdown in tokens and USD, plus total |
| **Vault Composition** | Donut chart: Base Assets vs Yield Positions breakdown |
| **Multichain Status** | Visual indicator of the 4 connected networks |
| **Gas — Master/Sub-Vaults** | Native gas reserves held for transaction fees (excluded from totals) |
| **Residual Assets** | Dust balances pending accumulation for future positions (excluded from totals) |
| **Verification Resources** | Links to Audit Framework, Document Seals, Treasury Policy, and Onchain Registry |

---

## 3. Vault Assets

### Base Assets

| Token | Full Name | Network | Type | Contract Address |
|---|---|---|---|---|
| **PAXG** | PAX Gold (Tokenized Gold) | Ethereum | ERC-20 | `0x45804880De22913dAFE09f4980848ECE6EcbAf78` |
| **QNT** | Quant Network | Ethereum | ERC-20 | `0x4a220E6096B25EADb88358cb44068A3248254675` |
| **LINK** | Chainlink | Polygon | ERC-20 | `0x53E0bca35eC356BD5ddDFebbD1Fc0fD03FaBad39` |
| **POL** | Polygon Ecosystem Token | Polygon | Native | — |
| **WXRP** | XRP Wrapped (BEP-20) | BNB Chain | ERC-20 | `0x1D2F0da169ceB9fC7B3144628dB156f3F6c60dBE` |
| **HYPE** | Hyperliquid | HyperEVM | Native | — |

### Ecosystem Value Positions

| Token | Protocol | Network | Type | Contract Address |
|---|---|---|---|---|
| **stPOL** | Polygon Liquid Staking (sPOL) | Ethereum | ERC-20 | `0x3B790d651e950497c7723D47B24E6f61534f7969` |
| **vXRP** | Venus Protocol | BNB Chain | vToken | `0xB248a295732e0225acd3337607cc01068e3b9c10` |

---

## 4. Idle USDC — Master Vaults & Sub-Vaults

USDC entering the Vault does not always map to a fixed asset or a yield position immediately. It typically follows one of two paths:

1. **Master Vault path:** USDC is deposited into a Master Vault and awaits conversion (swap) into a Base Asset (PAXG, QNT, LINK, POL, WXRP, HYPE).
2. **Sub-Vault path:** USDC is routed to the Sub-Vault corresponding to a yield strategy and awaits deployment into that strategy (e.g. staking, lending).

During this window, the USDC is neither a "position" in the traditional sense nor idle capital outside the protocol — it is committed Vault capital in transit. This balance is read directly on-chain and included in the dashboard's totals:

| USDC Location | Counted toward | Contract per network |
|---|---|---|
| Master Vaults (Ethereum, Polygon, BNB Chain, HyperEVM) | `Base Assets` | See table below |
| Sub-Vaults (Ethereum, Polygon, BNB Chain, HyperEVM) | `Ecosystem Value Positions` | See table below |

**USDC contract addresses per network:**

| Network | USDC Contract Address |
|---|---|
| Ethereum | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |
| Polygon | `0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174` |
| BNB Chain | `0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d` |
| HyperEVM | `0xb88339CB7199b77E23DB6E890353E22632Ba630f` |

USDC uses 6 decimals on all supported networks and is valued at a fixed $1.00, consistent with its role as a stable, fully-collateralized asset.

---

## 5. Safe Wallets — Vault Structure

| Safe | Address | Networks | Purpose |
|---|---|---|---|
| **Master Vault** | `0x315A47b154AA253F5660eDe42b64b4acD8402280` | Ethereum, Polygon, BNB Chain, HyperEVM | Primary vault — holds all base assets and idle USDC pending swap |
| **Sub-Ethereum** | `0xb9448016187B4DB709d24cC01AbaDbF3C654E175` | Ethereum | stPOL liquid staking position + idle USDC |
| **Sub-Polygon** | `0x7f61728427dEa4D188805275C6BcB607d005DFD0` | Polygon | Reserved for future positions + idle USDC |
| **Sub-BNB** | `0x9Fff9979A425AbD28c825406A64a31c524b7e403` | BNB Chain | Venus vXRP position + idle USDC |
| **Sub-HyperEVM** | `0x3f92214A1558E6b2d1734dFCD6DD105D37f454ab` | HyperEVM | Reserved for future positions + idle USDC |

> All addresses are publicly verifiable on their respective blockchain explorers.

---

## 6. Networks & RPCs

| Network | RPC Endpoint | Assets |
|---|---|---|
| **Ethereum** | `https://ethereum-rpc.publicnode.com` | PAXG, QNT, stPOL, USDC |
| **Polygon** | `https://polygon-bor-rpc.publicnode.com` | LINK, POL, USDC |
| **BNB Chain** | `https://bsc-rpc.publicnode.com` | WXRP, vXRP, USDC |
| **HyperEVM** | `https://rpc.hyperliquid.xyz/evm` | HYPE, USDC |

Public RPCs — no API key required.

---

## 7. Value Calculation

```
Total Vault Value = Base Assets USD + Yield Positions USD

USDC Master Vaults USD =
  (USDC_eth_master  × 1) +
  (USDC_pol_master  × 1) +
  (USDC_bnb_master  × 1) +
  (USDC_hype_master × 1)

USDC Sub-Vaults USD =
  (USDC_eth_sub  × 1) +
  (USDC_pol_sub  × 1) +
  (USDC_bnb_sub  × 1) +
  (USDC_hype_sub × 1)

Base Assets USD =
  (PAXG  × PAXG_price)  +
  (QNT   × QNT_price)   +
  (LINK  × LINK_price)  +
  (POL   × POL_price)   +
  (WXRP  × XRP_price)   +
  (HYPE  × HYPE_price)  +
  USDC Master Vaults USD

Yield Positions USD =
  (stPOL_balance × sPOL_price) +
  (vXRP_underlying × XRP_price) +
  USDC Sub-Vaults USD

Value Generated (per position) =
  (current_balance × current_price) − (original_deposit × current_price)
```

Balances are read directly from the blockchain using `balanceOf()` for ERC-20 tokens (including USDC) and `getBalance()` for native assets.

---

## 8. Scope — What Counts Toward Total Vault Value

To keep `Total Vault Value` an accurate representation of committed protocol capital, two categories are deliberately displayed separately and **excluded** from the total:

| Category | Included in Total Vault Value? | Where it's shown |
|---|---|---|
| Base Assets (Master Vault) | ✅ Yes | `Base Assets` section |
| Idle USDC — Master Vaults | ✅ Yes | `Base Assets` section |
| Yield Positions (Sub-Vaults) | ✅ Yes | `Ecosystem Value Positions` section |
| Idle USDC — Sub-Vaults | ✅ Yes | `Ecosystem Value Positions` section |
| Gas reserves (ETH, BNB) | ❌ No | `Gas — Master/Sub-Vaults` section, informational only |
| Residual Assets (dust balances) | ❌ No | `Residual Assets` section, informational only |

Gas reserves exist to pay for network transaction fees and are operational float rather than treasury value. Residual Assets are token dust below the threshold the protocol deploys into a position — they are disclosed for full transparency but do not represent committed capital yet.

---

## 9. Yield Positions — Technical Detail

### stPOL — Polygon Liquid Staking

POL deposited into the Polygon liquid staking protocol on Ethereum. The protocol issues **sPOL** tokens representing a share of the staking pool. The sPOL balance remains fixed but its underlying POL value increases over time as staking rewards accrue through the exchange rate.

**Balance reading:** `sPOL.balanceOf(subEthereum)` — returns raw sPOL shares
**Displayed as:** sPOL token balance (e.g. `36.754464 sPOL`)
**Value calculated as:** `sPOL_balance × sPOL_price`

**Value Generated formula:**
```
(36.754464 × sPOL_price) − (37 × POL_price)
```
This reflects the difference in USD between the current sPOL position valued at sPOL's market price and the original 37 POL deposited valued at POL's market price. The result may be negative in the short term since sPOL trades at a slight discount to POL — it turns positive as staking rewards accumulate and the exchange rate rises.

**Original deposit:** `37 POL` — confirmed on-chain at block `25,303,175` (Ethereum, Jun 12 2026)

### vXRP — Venus Protocol (BNB Chain)

XRP deposited in Venus Protocol (a Compound fork on BNB Chain). Venus issues **vXRP** tokens representing the position plus accrued interest.

**Balance reading:** Uses Venus `exchangeRateStored()` to convert vToken balance to underlying XRP:
```
underlying = (vXRP_balance × exchangeRate) / 10^36

Where:
  vXRP_balance   = raw vToken balance (8 decimals)
  exchangeRate   = stored exchange rate from the contract
  10^36          = 10^18 (Venus mantissa) × 10^18 (WXRP BEP-20 decimals)
```

`BigNumber` arithmetic from ethers.js is used to prevent precision loss.

**Original deposits:** `2 XRP` (block `103,924,340`, Jun 13 2026) + `2 XRP` (block `106,927,548`, Jun 28 2026) = **4 XRP total**

### USDC — Master & Sub-Vaults

USDC is not a yield-generating or price-appreciating position; it is valued 1:1 to the US Dollar and carries no "Value Generated" calculation, since it has no original deposit basis to compare against — it is transient capital, not a held position.

**Balance reading:** `USDC.balanceOf(vaultAddress)` on each network's USDC contract
**Displayed as:** aggregated total across the 4 networks, split into "Master Vaults" and "Sub-Vaults" line items

---

## 10. Original Deposits — Historical Constants

Original deposits are protocol decisions recorded on-chain. They are stored as constants in `getDepositoOriginal()` and only updated when a new deposit is made into an existing position. USDC is intentionally excluded from this function, since it has no original-deposit basis (see Section 9).

```javascript
function getDepositoOriginal(label: string): number {
    const depositos: Record<string, number> = {
        "stPOL": 37, // 37 POL — block 25,303,175 Ethereum (Jun 12 2026)
        "vXRP":  4,  // 2 XRP (block 103,924,340) + 2 XRP (block 106,927,548) BNB Chain
    }
    return depositos[label] ?? 0
}
```

> ⚠️ When adding a new deposit to an existing position, update this function with the new cumulative total.

---

## 11. Asset Prices — CoinGecko

Prices are fetched from the CoinGecko public API on every page load.

**Endpoint:**
```
GET https://api.coingecko.com/api/v3/simple/price?ids={ids}&vs_currencies=usd
```

| Token | CoinGecko ID |
|---|---|
| PAXG | `pax-gold` |
| QNT | `quant-network` |
| LINK | `chainlink` |
| POL | `polygon-ecosystem-token` |
| sPOL (stPOL) | `spol` |
| WXRP / vXRP | `ripple` |
| HYPE | `hyperliquid` |
| USDC | `usd-coin` (fixed at $1.00 in calculations) |

---

## 12. Technical Dependencies

| Library | Version | Purpose |
|---|---|---|
| `ethers.js` | 5.7.2 | Blockchain reads, unit formatting, BigNumber arithmetic |
| `chart.js` | 4.4.0 | Vault composition donut chart |
| `framer` | built-in | Property controls, canvas integration |

Imported directly from CDN — no installation required:

```javascript
import { ethers } from "https://esm.sh/ethers@5.7.2"
import { Chart, registerables } from "https://esm.sh/chart.js@4.4.0"
```

---

## 13. Adding a New Base Asset

Add a new entry to the `POSICIONES_FIJAS` array:

```javascript
{
    label:     "TOKEN",           // Token symbol (e.g., "AAVE")
    nombre:    "Full Name",       // Full name (e.g., "Aave Protocol")
    red:       "ethereum",        // "ethereum" | "polygon" | "bnb" | "hyperevm"
    safe:      SAFES.masterEthereum,
    tipo:      "erc20",           // "erc20" | "nativo"
    contrato:  "0x...",           // Contract address (erc20 only)
    decimales: 18,
    cgId:      "aave",            // CoinGecko ID
},
```

Also add the same asset to `scripts/snapshot.js` in this repository so it is included in hourly snapshots.

---

## 14. Adding a New Yield Position

**Step 1 — Add to `YIELD_POSITIONS`:**

```javascript
{
    label:           "stHYPE",
    nombre:          "Staked Hyperliquid",
    red:             "hyperevm",
    subVault:        SAFES.subHype,
    tokenContrato:   "0x...",       // token deposited
    yieldContrato:   "0x...",       // yield protocol contract
    depositoTipo:    "transfer",    // "transfer" (Venus-style) or "mint" (sPOL-style)
    decimales:       18,
    cgId:            "hyperliquid",
    deployBlock:     0,             // block near the first deposit date
    balanceContrato: "0x...",
    balanceTipo:     "erc20",       // "erc20" | "venus"
    esVenus:         false,
},
```

**Step 2 — Add original deposit to `getDepositoOriginal()`:**

```javascript
"stHYPE": 100, // 100 HYPE deposited — block XXXXXXX (date)
```

**Step 3 — Update `scripts/snapshot.js`** to include the new position balance in `yield_usd`.

**Deposit type reference:**
- `"transfer"` — sub-vault sends tokens directly to the yield contract (Venus pattern)
- `"mint"` — yield contract mints receipt tokens directly to the sub-vault (sPOL pattern)

---

## 15. Changelog

| Version | Date | Change |
|---|---|---|
| v1.0 | July 2026 | Added idle USDC tracking across the 4 Master Vaults (into `Base Assets`) and the 4 Sub-Vaults (into `Ecosystem Value Positions`). Added `Residual Assets` and `Gas — Master/Sub-Vaults` sections, explicitly excluded from `Total Vault Value`. |


---

## 16. Independent Verification

Every value displayed in the dashboard can be independently verified:

| What to verify | How |
|---|---|
| PAXG balance | Etherscan → `0x45804880...` → `balanceOf(0x315A47b1...)` |
| QNT balance | Etherscan → `0x4a220E60...` → `balanceOf(0x315A47b1...)` |
| LINK balance | Polygonscan → `0x53E0bca3...` → `balanceOf(0x315A47b1...)` |
| POL balance | Polygonscan → address `0x315A47b1...` → Native Balance |
| WXRP balance | BscScan → `0x1D2F0da1...` → `balanceOf(0x315A47b1...)` |
| HYPE balance | HyperEVM Explorer → address `0x315A47b1...` → Native Balance |
| stPOL balance | Etherscan → `0x3B790d65...` → `balanceOf(0xb9448016...)` |
| vXRP balance | BscScan → `0xB248a295...` → `balanceOf(0x9Fff9979...)` + `exchangeRateStored()` |
| USDC — Master Vaults | Etherscan / Polygonscan / BscScan / HyperEVM Explorer → USDC contract per network → `balanceOf(0x315A47b1...)` |
| USDC — Sub-Vaults | Same explorers → USDC contract per network → `balanceOf()` on each Sub-Vault address |
| Asset prices | CoinGecko public API or any price aggregator |
| stPOL deposit | Etherscan tx `0x0a079a2f...` block 25,303,175 |
| vXRP deposit 1 | BscScan tx `0x7549986b...` block 103,924,340 |
| vXRP deposit 2 | BscScan tx `0x2f4fac13...` block 106,927,548 |

---

*GNGA.WEB3 Protocol — Vault Monitor Dashboard v1.0 · July 2026*
