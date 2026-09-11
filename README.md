# A Frosty Path Less Traveled By — a trustless bridge & a shared road to restitution

A neutral, forward‑looking proposal for the Qubic ↔ Ethereum bridge after the recent incident: how a low‑cost **trustless** bridge — one **Vottun still operates** — can restore confidence, and how the activity that trust brings back can fund an orderly, transparent **restitution for affected users**, with every stakeholder's incentive pointing the same way.

> The name is a small nod to Robert **Frost** ("the road less traveled"), to **FROST** (one of the signature schemes discussed inside), and to relations having gone a little *frosty*. It takes no side. It is a map of **one** path forward, offered to the people who actually own the decision — to accept, reject, or reshape.

**Status:** DRAFT · Den of Misfits / Misfit Company · **not submitted** · **not endorsed by Vottun, Qubic Incubation, or the computor set** · neutral · does not adjudicate the incident

**Target:** Qubic core · the Vottun bridge contract (Qubic side) · an Ethereum‑side verifier — *proposals to be ratified, not decisions already made.*

## Contents

| Document | What's in it |
|---|---|
| ⭐ [**Executive summary →**](EXECUTIVE-SUMMARY.md) | The ~2‑minute version — the path, the key numbers, and a compact evidence table |
| 📋 [**The proposal →**](PROPOSAL.md) | The full path — the three parts, the gas/cost numbers, the honest arithmetic, and the open questions for each party |
| 🌉 Part A — *Trustless bridge* | How the 451‑of‑676 quorum attests to cross‑chain events while Vottun keeps operating; the three ways to verify it on Ethereum, with pros/cons ([§3](PROPOSAL.md#3-part-a--a-trustless-bridge-vottun-still-operates)) |
| 💸 Part B — *Core change for margin* | An **additive** secp256k1‑Schnorr co‑signing option that makes the trustless bridge cheap to verify — returning margin to Vottun ([§5](PROPOSAL.md#5-part-b--a-core-change-that-returns-margin-to-vottun)) |
| 🎟️ Part C — *Restitution* | An Incubation‑managed, capped token supply that routes redirected fees to affected users until the loss is repaid, then reverts to 90% shareholders / a permanently retained 10% affected‑party fund ([§6](PROPOSAL.md#6-part-c--a-shared-road-to-restitution-the-token-supply)) |
| 🧮 Reality check | The zk‑vs‑no‑zk cost model and the sobering repayment arithmetic — stated honestly ([§4](PROPOSAL.md#4-the-cost-reality--zk-vs-no-zk-with-numbers), [§7](PROPOSAL.md#7-the-reality-check-stated-honestly)) |
| 🤝 Alignment | Why every stakeholder's payoff points toward restoring volume — the real de‑escalation lever ([§8](PROPOSAL.md#8-why-this-works-for-everyone-the-alignment)) |
| 🧭 Open questions | What each party (Vottun, Incubation, computor set) must ratify ([§9](PROPOSAL.md#9-open-questions--for-vottun-incubation-and-the-computor-set)) |

## The ask, in one paragraph (for everyone)

Rebuild the bridge as a **trustless** one that Vottun still runs (so the community trusts the cryptography, not a signer); make a small **additive** Qubic core change so verifying it on Ethereum is cheap (returning margin to Vottun); and have **Incubation** stand up a capped, autonomous **token supply** that redirects the bridge's shareholder‑fee stream to **affected users** until the ~300B QU loss is repaid, then reverts to 90% shareholders with a permanently retained 10% affected‑party fund. The one thing that makes it hold together: **repayment is funded by bridge volume, and volume only returns if trust does — so the trustless bridge is the keystone, and every stakeholder is better off pulling in the same direction.**

> 🙏 A worked‑out **starting point, not a finished answer**, from outside contributors who are not a party to the dispute. Every mechanism is a proposal to ratify; the FourQ/quorum figures are estimates pending measurement (the measured anchors are labeled as such).
