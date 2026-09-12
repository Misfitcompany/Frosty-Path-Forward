# A Frosty Path Less Traveled By — a trustless bridge & a shared road to restitution — PROPOSAL DRAFT

**Status:** DRAFT (Den of Misfits / Misfit Company). **Not submitted.** **Not endorsed by Vottun, Qubic Incubation, or the computor set** — a path put forward for the parties who own the decision to accept, reject, or reshape.
**Target:** Qubic core · a new Qubic‑side bridge contract · a new Ethereum‑side verifier contract — *all proposals to be ratified, not decisions already made.*
**Stance:** neutral · forward‑looking · does **not** adjudicate the incident, its cause, or liability.

> 🙏 Written by outside contributors, not core maintainers and not a party to the dispute — a worked‑out starting point, not a finished answer. Figures marked *estimate* need real measurement; every mechanism below is a proposal the named parties must ratify.

---

## 0. The idea, in one paragraph (for everyone)

An incident led to the loss of **~300 billion QUBIC**, and the conversation since has drifted toward who is at fault. This document asks a different question: *what does a path forward look like where everyone is better off if it works?* In short — evolve the bridge into a **trustless** one, where the authority to move funds is the **Qubic computor quorum** (a two‑thirds‑plus supermajority of the 676 computors), verified cryptographically, rather than a company's operator keys. That takes two contracts: an **Ethereum contract that verifies the computors**, and a **Qubic bridge contract that reads the other chain through the network** instead of through a manager. The operator's role then shrinks to running a front end, the security is provided by the network, and the bridge costs almost nothing to run — no zero‑knowledge machinery required. Because the bridge is trusted and cheap again, the fees it earns can fund an **orderly, transparent restitution** for affected users. Every stakeholder's incentive ends up pointing the same way — toward the bridge working again.

---

## 1. Summary

A trustless bridge is **two contracts to build plus a restitution mechanism**, all resting on primitives Qubic already ships:

| Part | What it is | Who decides |
|---|---|---|
| **A** | **An Ethereum verifier contract** that checks the Qubic computor quorum's signatures, so releases need no operator. Shown two ways: verifying the current **FourQ** signatures, or verifying a **secp256k1** signature via a small additive co‑sign. | Vottun + Qubic core |
| **B** | **A rewritten Qubic bridge contract** that reads Ethereum through the oracle (inbound) and requests computor authorization through Outsourced Computation (outbound), with no manager gate — with **bob nodes** supporting smooth operation. | Vottun + Qubic core |
| **C** | An **Incubation‑managed, capped token supply** that autonomously routes the Qubic‑side shareholder fee to **affected users** until the loss is repaid, then reverts to 90% shareholders / 10% retained. | Incubation + the bridge's shareholders (consent) |

**The spine:** repayment is funded by bridge fees → fees need volume → volume needs restored trust → **the trustless bridge is the keystone; Part C is downstream of A and B.** No one is made whole on fees a zero‑volume bridge never earns.

---

## 2. Where the bridge stands today (context, no blame)

An incident occurred, **~300B QU was lost**, the bridge is effectively idle, and the parties best placed to fix it — **Vottun** (operator) and **Qubic Incubation** — are not currently aligned. This proposal takes **no position** on how the incident happened or who bears liability; it describes the current design only to make clear what changes.

The bridge is a non‑custodial lock‑and‑mint bridge (Qubic ↔ Ethereum and Arbitrum). On both sides, cross‑chain settlement is authorized by an **operator role**:

- **Ethereum side.** The front end's configuration points at a bridge gateway contract (`VITE_EVM_BRIDGE_CONTRACT = 0x60D7…811Aa`) and the Wrapped QUBIC token (`0xa989…F225`), both verified on Etherscan. The release function, `executeOrder`, takes plain order data (`originOrderId`, `originAccount`, `destinationAccount`, `amount`) and is gated by a `MANAGER_ROLE`, following a `confirmOrder` → `executeOrder` two‑step, with a 2‑of‑3 admin multisig for configuration (managers, fees, pausing). The contract contains no computor list and no signature/quorum verification.
- **Qubic side.** The bridge contract completes and refunds orders through a manager role, with a 2‑of‑3 multisig for administrative changes.

This is the common federated‑bridge model — releases are authorized by the operator's roles. It is the starting point this proposal evolves: moving the authority to move funds onto the computor quorum is what makes the bridge trustless, and — as Part C explains — what makes a fee‑funded restitution credible.

---

## 3. How a trustless bridge works (overview)

The authority to move funds becomes the **451‑of‑676 computor quorum**, verified cryptographically, in both directions:

- **Ethereum → Qubic (inbound):** the Qubic bridge contract issues an on‑chain oracle query (`EvmLogRead`) for the exact Ethereum log that recorded the lock/burn; every computor's oracle machine reads that log from a **finalized** Ethereum block, and the result is accepted only once **≥451 of 676 computors agree byte‑for‑byte**, after which the contract mints. *(Verified on the Qubic side; no Ethereum verifier involved.)*
- **Qubic → Ethereum (outbound):** the Qubic contract emits an **Outsourced‑Computation (OC)** request; each computor signs it, and the resulting bundle — the request plus **≥451 computor signatures** — is delivered to an **Ethereum verifier contract** that checks those signatures against the epoch's computor list before releasing wrapped QUBIC.

Reaching that state is two builds: the **Ethereum verifier** (§4) and the **rewritten Qubic contract** (§5). The operator runs only a front end and, optionally, a permissionless relayer that posts the outbound bundle — neither a trust point, because the verifier checks the computor signatures itself and anyone can relay.

---

## 4. Build 1 — an Ethereum contract that verifies the computors

This is the contract that lets Ethereum release funds on cryptographic proof of Qubic's quorum rather than on an operator's instruction.

### What it must do

1. **Hold the current epoch's computor set** (the 676 computors' public keys) and **update it each epoch** as the set rotates — verifiably derived from Qubic on‑chain data, not set by an admin.
2. **Accept the authorization bundle** — the pinned request parameters (`chainId`, `recipient`, `amount`, `target`), the `epoch`, `interfaceIndex` and `invocationId`, and the **451 `(computorIndex, signature)` pairs**.
3. **Reconstruct the exact signed message** — recompute `paramsDigest = KangarooTwelve(request parameters)`, then rebuild the domain‑separated authorization message (`epoch`, `interfaceIndex`, `invocationId`, `paramsDigest`) — so a signature is only valid for these precise parameters.
4. **Verify ≥451 valid signatures from distinct computors** in the current set.
5. **Enforce replay protection** — each `invocationId` usable once.
6. **Release only on success — with no operator override.** The sole path to mint/release is a valid quorum bundle; there is no role‑based mint.

Step 4 is the one that depends on the signature scheme, which gives two implementations.

### (a) Verifying the current FourQ (SchnorrQ) signatures

Qubic computors sign natively with **SchnorrQ over the FourQ curve**. The verifier implements FourQ signature verification directly in Solidity/Yul and checks the 451 signatures as‑is.

- **Qubic core change:** none — it consumes the native signatures.
- **Ethereum gas:** heavy on **L1** — Ethereum has no FourQ precompile, so each verification is custom elliptic‑curve arithmetic in contract code, and 451 of them is expensive. On an **L2 such as Arbitrum** (already a supported chain) gas is low enough that this is comfortable.
- **Build effort:** a substantial, custom, audit‑heavy verifier (hand‑written curve math).
- **Best fit:** an L2, or L1 where the per‑transfer gas is acceptable.

### (b) Verifying a secp256k1 signature (the additive alternative)

Computors **additionally** produce a **Schnorr signature on the secp256k1 curve** (an additive Qubic core change — a second signature alongside FourQ, from a secp256k1 key each computor derives from its existing seed — a new key, not a replacement). The Ethereum verifier then checks that signature with Ethereum's native `ecrecover` precompile.

- **Qubic core change:** yes — computors adopt an additive secp256k1 co‑signature for cross‑chain authorizations.
- **Ethereum gas:** low everywhere — `ecrecover` is a native precompile (~3k gas per signature), so verification is cheap even on **L1**.
- **Build effort:** small — the verifier leans on a battle‑tested precompile instead of custom curve math.
- **Best fit:** any chain, and the cheapest path for **Ethereum L1**.

### Choosing between them

| | (a) FourQ, verified directly | (b) secp256k1, via `ecrecover` |
|---|---|---|
| Qubic core change | None | Additive co‑sign |
| L1 gas per transfer | Heavy | Low |
| L2 (Arbitrum) gas | Fine | Fine |
| Verifier complexity | High (custom FourQ) | Low (native precompile) |

Both produce the **same trust property** — Ethereum verifies the 451‑of‑676 quorum — and **neither uses zero‑knowledge proofs.** The choice is a trade between avoiding a core change (a) and minimizing on‑chain cost and verifier complexity (b). *(A gas benchmark of a FourQ verifier versus an `ecrecover` path is the measurement that finalizes this decision, especially for L1.)*

---

## 5. Build 2 — the Qubic bridge contract, made trustless

The Qubic‑side contract changes from authorizing settlement by a manager role to authorizing it by the computor quorum, using primitives that already exist in Qubic core.

### What changes

- **Inbound (Ethereum → Qubic).** In place of a manager completing an order, the contract calls `QUERY_ORACLE(EvmLogRead, {chainId, txHash, logIndex})` for the Ethereum lock/burn; the computor quorum confirms the finalized log; the contract checks the log came from the bridge's own Ethereum contract and event, then mints. No manager approval.
- **Outbound (Qubic → Ethereum).** When a user burns wrapped QUBIC, the contract calls `INVOKE_OC` to request a computor‑signed authorization; the resulting 451‑signature bundle is what the Ethereum verifier (§4) checks. No manager approval.
- **Remove the settlement gate.** The manager/multisig path to complete or refund a transfer is removed, so no privileged party can move funds; only the quorum can.
- **Add a bridge OC interface.** Qubic core currently ships only a `Mock` outsourced‑computation interface; a bridge‑specific OC interface (carrying the release parameters) is added so the outbound path has a real target.

### Bob nodes — supporting smooth operation

Bob nodes are lightweight Qubic‑side **indexer** nodes that keep a readable copy of Qubic ticks, transactions and log events (and validate what they hold against the computor quorum, so they serve verified data). They make the bridge run smoothly:

- the **front end reads Qubic order status and logs** through a bob node, so users see the live state of their Qubic‑side transfer without waiting on a heavy node;
- the on‑chain `QubicLogRead` oracle is served from bob nodes (all configured bobs must agree byte‑for‑byte), so a contract can read a specific Qubic log event when needed; and
- they give operators a fast, self‑hosted source for **monitoring and reconciliation** of orders.

They are inexpensive — roughly **$10–40/month** per node — and Qubic's Network Guardians program rewards their uptime, which keeps them plentiful and well‑distributed. Bob nodes support operation and user experience; the cross‑chain settlement itself is secured by the computor quorum (inbound reads and outbound signatures), so a bob node is an operational aid, not part of the trust path.

---

## 6. What it costs (no zk)

The security infrastructure costs the bridge nothing: the 676 computors, their oracle/OC machines, and their bob nodes already exist and are funded by Qubic's protocol. The bridge's **own** recurring cost is a front end (static hosting), optionally one bob node for its own display and monitoring, and optionally a small relayer for the outbound leg — on the order of **~$10–40/month**, plus per‑withdrawal Ethereum gas (user‑payable). There is no zero‑knowledge proving anywhere in the design, and none is needed: Qubic's supermajority quorum is itself the trust mechanism the verifier checks. The one cost that varies is the outbound gas discussed in §4 — low on an L2 or with the secp256k1 path, heavier on L1 with direct FourQ verification.

---

## 7. Part C — a shared road to restitution (the token supply)

A framework, **not** a finished spec — and one that fits the no‑operator design cleanly, because the fee routing becomes autonomous contract logic with no human discretion.

1. **Incubation mints a capped, one‑time token supply** equal to the assessed loss — provably capped, mint key on a **multisig/timelock** (ideally renounced after mint) so affected users can't be diluted.
2. **Affected users receive tokens** representing their verified claim.
3. The new Qubic bridge contract **autonomously diverts the Qubic‑side 0.5% shareholder fee** (the dividend that holders of the bridge's Qubic‑side smart‑contract shares earn) into a redemption pool that pays token holders. *(The operator's half is the separate Ethereum‑side 0.5%, untouched unless voluntarily shared — see §9.)*
4. When the pool has paid out the loss constant, the split **reverts to 90% shareholders / 10% retained by the token supply** — a **deliberate, lasting fund** that keeps ongoing resources and standing in the hands of affected parties.

### 🙏 Design decisions the parties must make

- **Who is an "affected user," and in what unit?** Affected users are **the bridge's users** whose funds were lost in the incident; defining the set needs an agreed **snapshot tick**, an authoritative **pre‑incident** ledger, and an **address‑ownership proof** — with rules for exchange‑custodied funds, lost keys, and partial balances, plus a claims window and dispute process. Denominate in **nominal QU (working figure 300B)** or **USD‑value‑at‑incident**, and use that same unit for the on‑chain reversion counter.
- **What one token represents.** By design, **both** a restitution claim (paid during repayment) **and** a lasting share of the retained **10% tail** — tokens persist rather than retire. Decide the tail pool's **governance and permitted use** (e.g. ongoing distributions to holders or an affected‑party safety reserve), whether tokens are **transferable** (liquidity, but speculators can hold them) or **soulbound**, and what happens to unclaimed tokens.
- **Redemption and control.** While the pool is under‑funded, **pro‑rata drip** is fairer than FIFO or lottery; specify the token→QU rate and who pays claim gas. Because Incubation is itself a party to the dispute, the **capped mint and governance sit with a multi‑party multisig** (user + Vottun + computor + Incubation), not any single party.

---

## 8. The honest reality check

Repayment is funded only by redirected fees, and the bridge is at ~0 volume — so affected users are made whole **in expectation, not in certainty**, and only once volume returns, which requires restored trust (the two builds above). Repaying **300B QU** from a **0.5%‑of‑volume** diversion requires roughly **~60 trillion QU of cumulative bridged volume** (`300B / 0.005`). Illustrative years‑to‑repay (plug in real projections — these are hypothetical):

| Assumed monthly bridged volume | Years to repay 300B QU |
|---:|---:|
| ~0 (today) | never |
| 100B QU/mo | ~50 |
| 1T QU/mo | ~5 |
| 5T QU/mo | ~1 |

Levers that shorten it: a **voluntary operator contribution** from its Ethereum‑side half (§9), a temporary higher fee, or supplementary funding. Two further caveats: a long‑dated claim will likely trade **below face** (a secondary‑market haircut), and a **continuity/assignment** plan should ensure repayment survives any change in who runs the front end.

---

## 9. Why this works for everyone (the alignment)

Every stakeholder's payoff points toward **restoring volume**, which turns a blame fight into a shared growth objective.

| Stakeholder | What they get | The honest tension |
|---|---|---|
| **Affected users** | A funded, transparent claim; a real path toward being made whole | Illiquid, long‑dated, **volume‑contingent** — better than a stalemate, short of instant restitution |
| **Vottun** | Keeps a role (the front end); a bridge costing ~$10–40/mo to run, so nearly all its fee is **margin**; a design where no single party can be the point of failure or blame | Comes out ahead — so a **voluntary contribution** from its Ethereum‑side half is suggested to keep it fair |
| **Bridge‑contract shareholders** | Diversion costs ~$0 today; restored to **90%** of their share once volume returns | Bear the cost and **must consent** (a governance vote); the retained **10% tail** permanently funds affected parties |
| **Incubation** | A clear, constructive role in the recovery | As a party to the dispute, it shouldn't control the funds alone → **multi‑party control**, not sole control |

⚠️ **Neutrality is structural, not just lexical.** As drafted, shareholders fund 100% of repayment while the operator's Ethereum‑side half is untouched — which can read as *"users paying for an incident they did not cause."* A **voluntary operator contribution** (a temporary share of its Ethereum‑side half) defuses that: it costs little at today's volume, materially shortens repayment when volume returns, and is raised as an option to offer, not an obligation to impose.

---

## 10. Open questions — for Vottun, Incubation, and the computor set

**For Qubic core devs (Builds A/B):**
1. **L1 or L2 for outbound** — and, if L1, support for an **additive secp256k1 co‑signature** so the verifier can use `ecrecover`? (A measured L1 gas benchmark of both paths settles this.)
2. Build the three pieces — the **Ethereum verifier**, the **rewritten Qubic contract** (oracle inbound / OC outbound, no manager gate), and a **bridge OC interface** in core — and confirm how the epoch computor‑set is delivered to and rotated in the verifier.

**For Vottun:**
3. Willing to run the **front end** for a bridge whose contracts ship **without** a manager/settlement gate, with a **continuity/assignment** plan — and open to a **voluntary contribution** to shorten repayment?

**For Incubation + the bridge's shareholders (Part C):**
4. **Multi‑party** (not sole‑Incubation) control of the **capped, autonomous** restitution contract, and a **shareholder‑consent** vote to divert the shareholder half?
5. Agreement on the **affected‑user snapshot, ledger, denomination, and claims process** (per §7)?

---

## 11. Scope reminder

🙏 This is a **starting point, not a finished answer**, from outside contributors who take **no side** in the dispute and make **no claim** about how the incident occurred. Every mechanism here is a proposal for Vottun, Qubic Incubation, and the computor community to accept, reject, or reshape; every performance figure is an **estimate** pending measurement.

The through‑line is the only thing worth holding onto: **a trustless bridge restores trust, restored trust restores volume, and restored volume is what makes affected users whole while keeping every party aligned** — and it is built on primitives Qubic already has, with no operator in the trust path and no zero‑knowledge machinery. That is a road less traveled, but the map points everyone in the same direction.
