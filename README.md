# A Frosty Path Less Traveled By — a trustless bridge & a shared road to restitution

A neutral, forward‑looking proposal for the Qubic ↔ Ethereum bridge after the recent incident: how a **trustless** bridge — one where the authority to move funds is the Qubic computor quorum and the operator **just runs a front end** — can be built on primitives Qubic already has, run for **~$10–40/month** with **no zero‑knowledge machinery**, restore confidence, and let the activity that trust brings back fund an orderly **restitution for affected users**, with every stakeholder's incentive pointing the same way.

> The name nods to Robert **Frost** ("the road less traveled"), to a couple of the signature schemes discussed inside, and to relations having gone a little *frosty*. It takes no side — a map of **one** path forward, offered to the parties who own the decision, to accept, reject, or reshape.

**Status:** DRAFT · Den of Misfits / Misfit Company · **not submitted** · **not endorsed by Vottun, Qubic Incubation, or the computor set** · neutral · does not adjudicate the incident

**Target:** Qubic core · a new Qubic‑side bridge contract · a new Ethereum‑side verifier — *proposals to be ratified, not decisions already made.*

## Contents

| Document | What's in it |
|---|---|
| ⭐ [**Executive summary →**](EXECUTIVE-SUMMARY.md) | The ~2‑minute version — the two builds, the key numbers, and a compact evidence table |
| 📋 [**The proposal →**](PROPOSAL.md) | The full path — the current design, the two contracts to build, restitution, the honest arithmetic, and the open questions |
| 🔎 Where it stands today | The current bridge, stated factually — operator‑authorized on both sides (Ethereum `executeOrder` role‑gated at `0x60D7…`, Qubic manager model) ([§2](PROPOSAL.md#2-where-the-bridge-stands-today-context-no-blame)) |
| 🔗 Build 1 — *Ethereum verifier of the computors* | An Ethereum contract that checks ≥451 computor signatures — shown with current **FourQ** and with a **secp256k1** co‑sign alternative (`ecrecover`) ([§4](PROPOSAL.md#4-build-1--an-ethereum-contract-that-verifies-the-computors)) |
| 🌉 Build 2 — *Qubic contract, made trustless* | The Qubic bridge contract reading Ethereum via the oracle inbound and OC outbound, no manager gate — with **bob nodes** supporting smooth operation ([§5](PROPOSAL.md#5-build-2--the-qubic-bridge-contract-made-trustless)) |
| 🎟️ Part C — *Restitution* | A capped, autonomous token supply routing the Qubic‑side shareholder fee to affected users until repaid, then 90% shareholders / a permanently retained 10% affected‑party fund ([§7](PROPOSAL.md#7-part-c--a-shared-road-to-restitution-the-token-supply)) |
| 🧮 Cost & reality check | The real **~$10–40/mo** cost (no zk) and the sobering repayment arithmetic ([§6](PROPOSAL.md#6-what-it-costs-no-zk), [§8](PROPOSAL.md#8-the-honest-reality-check)) |
| 🤝 Alignment | Why every stakeholder's payoff points toward restoring volume ([§9](PROPOSAL.md#9-why-this-works-for-everyone-the-alignment)) |
| 🧭 Open questions | What each party (Vottun, Incubation, computor set) must ratify ([§10](PROPOSAL.md#10-open-questions--for-vottun-incubation-and-the-computor-set)) |

## The ask, in one paragraph (for everyone)

Evolve the bridge into a **trustless** one where the authority to move funds is the **451‑of‑676 Qubic computor quorum**, verified cryptographically, instead of a company's keys. It takes **two contracts**: an **Ethereum verifier** that checks the computor signatures (with the current FourQ, or with a cheaper secp256k1 co‑sign via `ecrecover`), and a **rewritten Qubic bridge contract** that reads Ethereum through the oracle inbound and requests computor authorization outbound, with no manager gate — with **bob nodes** keeping operation smooth. The operator's role shrinks to a **front end**, the security is the network's (**$0** to the bridge, **no zk**), and the bridge runs for **~$10–40/month**. Then have **Incubation** stand up a capped, autonomous **token supply** that routes the Qubic‑side shareholder fee to **affected users** until the ~300B QU loss is repaid, then reverts to 90% shareholders with a permanently retained 10% affected‑party fund. The one thing that makes it hold together: **repayment is funded by bridge volume, and volume only returns if trust does — so the trustless bridge is the keystone, and every stakeholder is better off pulling in the same direction.**

> 🙏 A worked‑out **starting point, not a finished answer**, from outside contributors who are not a party to the dispute. The trustless bridge is a **build on existing Qubic primitives**; every mechanism is a proposal to ratify.
