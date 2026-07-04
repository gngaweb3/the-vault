# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in this repository — the snapshot pipeline, the GitHub Actions workflow, or the on-chain read logic — please report it **privately**. Do not open a public GitHub issue or disclose it on social media before it has been addressed.

**Report to:** security.gnga@proton.me

Please include:
- A description of the vulnerability and its potential impact
- Steps to reproduce it
- Any relevant logs, transaction hashes, or code references

We will acknowledge your report as soon as possible and keep you informed as the issue is investigated and resolved. Once a fix is deployed, we're happy to credit you publicly if you'd like.

## Scope

This policy covers the code in this repository (`the-vault`): the snapshot script, GitHub Actions workflow, and any automation that reads or reports on The Vault's on-chain state.

It does not cover the GNGA.WEB3 Protocol's smart contracts (deployed and audited separately) or third-party infrastructure (Supabase, Polygonscan, public RPCs, CoinGecko). If you believe you've found an issue in one of those, please still reach out — we'll help route it to the right place.

## Out of Scope

- Social engineering attacks
- Denial-of-service attacks against public RPCs or third-party APIs this repo depends on
- Issues that require physical access to a maintainer's device
