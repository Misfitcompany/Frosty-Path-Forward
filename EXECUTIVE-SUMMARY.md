# A Frosty Path Less Traveled By — executive summary

**Status:** DRAFT · Den of Misfits / Misfit Company · **not submitted** · **not endorsed by Vottun, Qubic Incubation, or the computor set** · neutral · does not adjudicate the incident
**Full proposal:** [`PROPOSAL.md`](PROPOSAL.md) · **This page:** ~2‑min read

---

## The situation (neutral)

A bridge incident led to the loss of **~300B QUBIC**, the bridge is idle, and the two parties best placed to fix it — **Vottun** (operator) and **Qubic Incubation** — are not aligned. Today the bridge is **operator‑authorized on both sides**: on Ethereum the gateway (`0x60D7…811Aa`) releases via a `MANAGER_ROLE`, with no computor‑signature check; on Qubic a manager completes and refunds orders. This document takes **no position** on cause or liability; it offers a path where the fix and the repayment reinforce each other.

## The path — two contracts to build, plus restitution

- **A — An Ethereum contract that verifies the computors.** So releases need **no operator**: the contract checks **≥451 of 676 computor signatures** against the epoch's computor set. Two ways, both **no zk**:
  - **(a) verify the current FourQ signatures** directly — no core change to the signature scheme (a bridge OC interface is still needed either way); heavy gas on L1, comfortable on an L2 (Arbitrum); a custom verifier.
  - **(b) verify a secp256k1 signature** via an additive computor co‑sign — cheap everywhere (native `ecrecover`, ~3k gas/sig), a simple verifier; needs a small core change.
- **B — A rewritten Qubic bridge contract** that reads Ethereum through the `EvmLogRead` oracle (inbound) and requests computor authorization through Outsourced Computation (outbound), with **no manager gate**. **Bob nodes** (~$10–40/mo) support smooth operation — front‑end order status, log reads, monitoring — while the settlement itself is secured by the computor quorum.
- **C — An Incubation‑managed token supply for restitution.** A capped, autonomous contract routes the Qubic‑side **shareholder** fee to **affected users** until the loss is repaid, then reverts to **90% shareholders / 10% permanently retained** as a lasting affected‑party fund.

## The one thing that makes it hold together

> Repayment is funded by bridge fees → fees require volume → volume requires restored trust → **trust is the two builds above.** So the trustless bridge is the keystone, and restitution is downstream of it. Every stakeholder's payoff points the same way — toward the bridge earning volume again.

## What each party gets — and the honest tension

| | Gets | Tension |
|---|---|---|
| **Affected users** | A funded, transparent claim; a real path toward being made whole | Illiquid, long‑dated, **volume‑contingent** |
| **Vottun** | Keeps a role (the front end); a bridge costing ~$10–40/mo, so nearly all its fee is **margin**; a design where no single party can be the point of failure | Comes out ahead — a **voluntary contribution** is suggested to keep it fair |
| **Shareholders** | Diversion costs ~$0 today; restored to **90%** once volume returns | Bear the cost and **must consent**; the retained **10% tail** is a permanent transfer to the affected‑party fund |
| **Incubation** | Clear direction and a constructive role | A party to the dispute → **multi‑party control**, not sole control |

## Stated honestly

⚠️ Trustlessness is **two contracts to build** on existing Qubic primitives — an Ethereum verifier and a rewritten Qubic contract (plus a bridge OC interface in core). Repayment is **not guaranteed** — it depends on restored volume (see the arithmetic). Every part is a **proposal to ratify**. **There is no zk in this design.**

## The ask

Each party ratifies its part. **Vottun:** run the front end for a bridge whose contracts ship *without* a manager gate (± a voluntary contribution). **Core devs:** build the Ethereum verifier + rewritten Qubic contract + a bridge OC interface, and make the L1 (secp256k1) vs L2 choice. **Incubation + shareholders:** stand up the capped, autonomous restitution contract with shareholder consent.

---

## 📄 Evidence & key numbers (compact)

| Item | Figure | Basis |
|---|---:|---|
| **Trust source** | The **451‑of‑676 computor quorum**, cryptographically verified — no operator, no zk | Cited (Qubic core) |
| **Security infrastructure cost to the bridge** | **$0** — the computors + their oracle/OC machines already exist, protocol‑funded | Cited (`contracts_oracles.md`) |
| Bridge's own opex | **~$10–40/month** (front‑end + optional bob node + optional relayer) | Estimate (VPS pricing) |
| Bob node's role | Qubic‑side **indexer** for front‑end status/logs & monitoring — an operational aid, not the settlement security | Cited (core) |
| Inbound (EVM→Qubic) | Secured on Qubic by `EvmLogRead` + 451/676 quorum reading a **finalized** Ethereum log | Cited (core) |
| Outbound (Qubic→EVM) | Secured by an **Ethereum verifier** of ≥451 computor signatures — **FourQ** (heavy on L1) or **secp256k1** (cheap via `ecrecover`) | Cited (core) + estimate |
| Current deployed bridge | **Trusted/role‑based on both sides** — Ethereum `executeOrder` gated by `MANAGER_ROLE` (`0x60D7…811Aa`), no quorum check; Qubic manager model | Front‑end config + Etherscan‑verified |
| Volume to repay 300B QU at 0.5% diversion | **~60 trillion QU** cumulative bridged | Arithmetic (`300B / 0.005`) |
| Years to repay (illustrative) | 0 vol → never · 100B/mo → ~50y · 1T/mo → ~5y · 5T/mo → ~1y | Illustrative, not a forecast |

*Fee (confirmed): **1% total = 0.5% Qubic‑side** (dividends to the holders of the bridge's Qubic‑side smart‑contract shares) **+ 0.5% Ethereum‑side** (Vottun); Part C diverts the Qubic‑side half. Loss: working figure **300B QU**, owed to the bridge's users.*

> 🙏 A worked‑out **starting point, not a finished answer**, from outside contributors who are not a party to the dispute. Every mechanism is a proposal to ratify; the outbound‑verification cost figures are estimates pending a measured benchmark.
