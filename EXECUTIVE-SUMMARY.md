# A Frosty Path Less Traveled By — executive summary

**Status:** DRAFT · Den of Misfits / Misfit Company · **not submitted** · **not endorsed by Vottun, Qubic Incubation, or the computor set** · neutral · does not adjudicate the incident
**Full proposal:** [`PROPOSAL.md`](PROPOSAL.md) · **This page:** ~2‑min read

---

## The situation (neutral)

A bridge incident led to the loss of **~300B QUBIC**, the bridge is idle, and the two parties best placed to fix it — **Vottun** (operator) and **Qubic Incubation** — are not aligned. This document takes **no position** on cause or liability. It offers one thing: a path forward where the fix and the repayment reinforce each other.

## The path, in three parts

- **A — A trustless bridge Vottun still operates.** The **451‑of‑676 computor quorum** attests to cross‑chain events, so users trust the *cryptography and the code*, not a signer. Vottun keeps running the bridge; only the trust model changes.
- **B — A small, additive Qubic core change.** Computors *additionally* co‑sign in an EVM‑friendly scheme (secp256k1‑Schnorr, **not** replacing FourQ), making Part A cheap to verify on Ethereum — which **returns margin to Vottun**.
- **C — An Incubation‑managed token supply for restitution.** A **capped, autonomous** contract routes the bridge's *shareholder* fee half to **affected users** until the loss is repaid, then reverts to **90% shareholders / 10% permanently retained** (a deliberate affected‑party fund).

## The one thing that makes it hold together

> Repayment is funded by bridge fees → fees require volume → volume requires restored trust → **trust is Part A.** So **Part A is the keystone, and C is downstream of A.** The payoff for *every* stakeholder points the same way — toward the bridge earning trust and volume again. **That shared direction is the de‑escalation lever.**

## What each party gets — and the honest tension

| | Gets | Tension |
|---|---|---|
| **Affected users** | A funded, transparent claim; a real path toward being made whole | Illiquid, long‑dated, **volume‑contingent** — better than a stalemate, short of instant restitution |
| **Vottun** | Keeps its role; **higher margin** (B); crypto that deflects future single‑point blame | Comes out ahead — so a **voluntary contribution** is suggested to keep it fair |
| **Shareholders** | Diversion costs ~$0 today; restored to **90%** of their share once volume returns | Bear the cost and **must consent**; the retained **10% tail** permanently funds affected parties — a deliberate transfer, and the main securities item for counsel |
| **Incubation** | Clear direction and a constructive role | Inherits issuer/legal risk → **multi‑party control**, not sole control |

## Stated honestly

⚠️ Repayment is **not guaranteed** — it depends on restored volume (see the arithmetic below). The FourQ/quorum verify figures are **estimates** pending a measured verifier (the `ecrecover` and Groth16 anchors are measured). A revenue‑claim token carries **securities/legal** questions → **counsel before finalizing**. Neutrality is *structural*, not just wording. Every part is a **proposal to ratify**, not a decision made for anyone.

## The ask

Ratify your part: **Vottun** — operate the trustless bridge (± a voluntary contribution); **Core devs** — add the additive secp256k1 co‑signing; **Incubation + shareholders** — stand up the capped, autonomous restitution contract with shareholder consent. Rebuild trust first; the rest follows.

---

## 📄 Evidence & key numbers (compact)

| Item | Figure | Basis |
|---|---:|---|
| Single FourQ (`SchnorrQ`) verify on‑chain | ~350k–450k gas | **Estimate** (anchored on measured P‑256) |
| Quorum — 451 verified **individually** | ~180M gas → **exceeds ETH block limit** | **Estimate** → must aggregate |
| Aggregated verify (epoch key cached) | ~0.4M–0.5M gas | **Estimate** |
| `ecrecover`‑Schnorr (secp256k1) anchor | ~3k–13k gas | **Measured** |
| Groth16 zk‑verify anchor | ~181k–250k gas | **Measured** |
| Standing zk prover | ~$3k–5k/mo **fixed** (1× H100/A100‑80GB; reserved floor ~$1,379/mo) | Measured GPU pricing |
| 3 months at ~0 volume (standing prover) | ~$4,100–$15,000 burned for 0 proofs | Derived |
| zk pay‑per‑proof | ~$0 idle; ~$0.04–$0.30/checkpoint | **Estimate** |
| Checkpoint cadence | ~4.33/month — one attestation per weekly epoch (assumed as the logical default; dev to confirm) | Qubic epoch = 7 days |
| Volume to repay 300B QU at 0.5% diversion | **~60 trillion QU** cumulative bridged | Arithmetic (`300B / 0.005`) |
| Years to repay (illustrative) | 0 vol → never · 100B/mo → ~50y · 1T/mo → ~5y · 5T/mo → ~1y | Illustrative, not a forecast |

*Fee (confirmed): **1% total = 0.5% Qubic‑side** (the 676 computor‑shareholder contracts) **+ 0.5% ETH‑side** (Vottun); rate is governable. Part C diverts the Qubic‑side half. Loss: working figure **300B QU**.*

> 🙏 A worked‑out **starting point, not a finished answer**, from outside contributors who are not a party to the dispute. Every mechanism is a proposal to ratify; the FourQ/quorum figures are estimates pending measurement (the measured anchors are labeled as such).
