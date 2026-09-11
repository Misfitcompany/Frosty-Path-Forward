# A Frosty Path Less Traveled By — a trustless bridge & a shared road to restitution — PROPOSAL DRAFT

**Status:** DRAFT (Den of Misfits / Misfit Company). **Not submitted.** **Not endorsed by Vottun, Qubic Incubation, or the computor set** — a path put forward for the people who own the decision to accept, reject, or reshape.
**Target:** Qubic core · the Vottun bridge contract (Qubic side) · an Ethereum‑side verifier — *all as proposals to be ratified, not decisions already made.*
**Stance:** neutral · forward‑looking · does **not** adjudicate the incident, its cause, or liability.

> 🙏 We are outside contributors, not core maintainers and not a party to the dispute. Please read this as a **worked‑out starting point, not a finished answer**. Every number labelled *estimate* needs a real measurement before anyone relies on it, and every mechanism below is a proposal the named parties must ratify.

---

## 0. The idea, in one paragraph (for everyone)

A bridge incident led to the loss of **~300 billion QUBIC**, and the conversation since has drifted toward who is at fault. This document does not answer that question — it offers a different one: *what does a path forward look like where everyone is better off if it works?* The shape is simple. **(A)** Rebuild the bridge as a **trustless** one that Vottun still runs, so the community trusts the cryptography rather than any single operator. **(B)** Make a small **Qubic core change** so that trustless bridge is cheap to operate, returning margin to Vottun. **(C)** Have Qubic **Incubation** stand up a transparent, capped **token supply** that routes redirected bridge fees to **affected users** until the loss is repaid, then steps back. The point that makes this more than a wish: **every stakeholder's payoff points the same way — toward the bridge earning trust and volume again.** That shared direction is the actual de‑escalation lever.

---

## 1. Summary

Three parts, meant to be adopted together but ratified independently:

| Part | What it is | Who decides |
|---|---|---|
| **A** | A **trustless bridge Vottun still operates** — the 451‑of‑676 computor quorum attests to cross‑chain events, so users trust the quorum + the code, not a multisig. Three ways to verify it on Ethereum, compared below. | Vottun + Qubic core |
| **B** | A **Qubic core change** (computors additionally co‑sign in an EVM‑friendly scheme) that makes Part A's on‑chain verification cheap — lowering operating cost and **returning margin to Vottun**. | Qubic core devs |
| **C** | An **Incubation‑managed token supply** that redirects the bridge's *shareholder* fee stream to **affected users** until the loss is repaid, then reverts to a 90/10 split. | Incubation + computor shareholders (consent) |

**The spine (please keep this in view while reading):** Parts B and C only pay out if the bridge has volume, and volume only returns if trust is restored — which is Part A. So **Part A is the keystone; C is downstream of A.** Nobody is made whole on redirected fees that a zero‑volume bridge never earns. This is stated plainly, up front, on purpose.

---

## 2. The situation, without blame (context)

What is not in dispute for the purposes of this document: an incident occurred, **~300B QU was lost**, the bridge is effectively idle, and the parties best positioned to fix it — **Vottun** (operator) and **Qubic Incubation** — are not currently aligned.

⚠️ This proposal takes **no position** on how the incident happened, whose controls were involved, or who bears liability. Those are questions for the parties, their auditors, and if necessary their counsel — not for a forward‑path document. Where we describe the bridge's current design, it is only to show what changes, never to assign fault.

We wrote this because a stalled recovery helps no one: not the affected users waiting, not Vottun's standing in the ecosystem, not Incubation's mandate, not Qubic. The rest of this document is entirely about the road ahead.

---

## 3. Part A — a trustless bridge Vottun still operates

Today's bridge relies on a small trusted signer set (a manager/multisig model). A **trustless** bridge replaces "trust the operator's attestation" with "verify the network's attestation":

1. A deposit/lock/burn event happens on one chain.
2. The **Qubic computor quorum (451 of 676)** independently observes it (inbound, via the Oracle read primitives `EvmLogRead` / `QubicLogRead`) or authorizes the counter‑action (outbound, via Outsourced Computation), producing a quorum‑signed attestation.
3. The destination chain verifies that attestation and mints/releases.

**Vottun keeps operating and managing the bridge** — the contracts, liquidity, relaying, and UX. What changes is *where the trust sits*: in the 451‑of‑676 quorum and audited code, not in a signer Vottun controls.

### 🛡️ What "trustless" does and does not mean here (stated honestly)

Being precise matters, because over‑claiming is how trust is lost twice:

- ✅ **Removed:** the need to trust a Vottun‑held key to *attest truthfully* to cross‑chain events. That becomes cryptographically verifiable against the quorum.
- ⚠️ **Still assumed:** the honesty of the computor quorum itself; the security of any **new keys** computors hold (see Part B); and Vottun's **liveness/censorship** role — Vottun still runs the machinery, so it can be slow or decline to act, even though it cannot forge. "Trustless" here means *reduced trust assumptions and defense‑in‑depth for everyone* — a general best practice — not a cure aimed at any one party's prior conduct.

### The real cost question is the *quorum*, not the curve

A common worry is that verifying Qubic signatures on Ethereum is prohibitively expensive. Our estimate says the opposite for a *single* signature, and pinpoints where the cost actually lives.

Qubic signs with **SchnorrQ** (Schnorr over the **FourQ** curve). FourQ has no Ethereum precompile, so it must be verified in contract code — but two properties make that far cheaper than the usual "non‑native curve on the EVM" horror stories:

- FourQ's prime `p = 2^127 − 1` is a **Mersenne prime below 2^128**, so a full field multiply is a **single `MULMOD` opcode (8 gas)** with no overflow.
- FourQ's **4‑D GLV** decomposition cuts the scalar‑multiplication loop to ~64 iterations (vs ~128–256).

**Estimated on‑chain cost of one FourQ SchnorrQ verify: ~350k–450k gas** (defensible range ~250k–600k). *Estimate — no public FourQ‑in‑EVM implementation exists yet; a measured Solidity/Yul verifier is what finalizes this.* For calibration it lands near an **optimized pure‑Solidity P‑256 verify (~200k–334k, measured)** and well below a **naive ed25519** (one scalar‑mult alone is ~1.25M gas, measured; a full verify higher).

The expensive part is that a bridge doesn't verify *one* signature — it must confirm the **whole 451‑signature quorum**, and Qubic computors sign **individually** (they don't natively aggregate):

| What you verify on Ethereum | Estimated gas | Fits in one L1 block? |
|---|---:|---|
| One FourQ SchnorrQ signature | ~350k–450k | Yes |
| **451 signatures verified individually (naive)** | **~180M (≈113M–226M)** | **No** — exceeds the block gas limit (tens of millions of gas) by multiples |
| Aggregate verify, epoch key **cached** (MuSig‑style) | ~0.4M–0.5M | Yes |
| Aggregate verify, decompressing 451 keys per call | ~8M–10M | Barely — *don't do this; cache the key* |

*(All quorum figures are estimates built on the single‑verify estimate, itself anchored on measured curves.)* The takeaway: the naive path is infeasible, so **the design must aggregate** — and *how* you aggregate is the real choice.

### The three ways to make the quorum verifiable — pros & cons

| Option | On‑chain verify (est.) | Off‑chain / infra cost | Qubic‑side change | Notes |
|---|---:|---|---|---|
| **1. Custom FourQ verifier + FourQ aggregation** | ~0.4M–0.5M (cached key) | none | Support **FourQ signature aggregation** (protocol change) + a hand‑written **FourQ verifier in Solidity** (large, audit‑heavy) | Keeps the native curve; the Solidity FourQ verifier is the cost |
| **2. zk‑wrapped verification** | ~181k–250k (Groth16, **measured** anchor) | **proving cost** — fixed standing prover **~$3k–5k/mo**, *or* **pay‑per‑proof ~$0 idle** | none (proves the existing FourQ path) | Cheapest, smallest on‑chain footprint; the cost moves to *proving* (see §4) |
| **3. secp256k1‑Schnorr / FROST (EVM‑native)** | **~3k–13k** via `ecrecover` (**measured** anchor) | none | Computors **additionally** co‑sign in secp256k1‑Schnorr (Part B) + aggregate/threshold | Cheapest verify, no custom verifier, but depends on the Part B core change |

📄 Anchors used above are measured: `ecrecover`‑Schnorr ~3k–13k gas; Groth16 ~181k–250k gas; optimized pure‑Solidity P‑256 ~200k–334k gas; naive ed25519 ~1.5–2.5M gas. The FourQ single‑verify (~350k–450k) is an estimate anchored on those.

---

## 4. The cost reality — zk vs no‑zk (with numbers)

Because "just use zk" is a common reflex, here is the honest cost model, and why it depends entirely on **volume**.

zk trades a **variable** cost for a **fixed** one:

| | Fixed monthly cost | Marginal (per‑checkpoint) cost |
|---|---:|---|
| **zk on a standing prover** | **~$3k–5k/mo** (one always‑on H100 / A100‑80GB; reserved floor ~$1,379/mo) | very low |
| **zk via pay‑per‑proof** | **~$0** (no requester minimum) | ~$0.04–$0.30/checkpoint (est.) |
| **Non‑zk aggregated (FourQ or secp256k1)** | **~$0** | on‑chain gas only (see below) |

**Why a standing prover is the wrong fit at low volume:** it bills the same at $0 or $1M of volume. Over a **~3‑month zero‑volume stretch it burns ~$4,100–$15,000 for zero proofs.** And because Qubic epochs are **weekly**, a per‑epoch bridge posts only **~4.33 checkpoints/month — a number that does not grow with volume.** For a standing prover to beat aggregated non‑zk on gas, you'd need **hundreds to thousands of checkpoints per month**; at ~4.33 that break‑even is effectively unreachable. Meanwhile a non‑zk aggregated verify at weekly cadence costs on the order of **$1–$2,200/month** — the low end is the cached‑key path (~0.4–0.5M gas), the high end a non‑cached Solidity‑FourQ path (5–10M gas), across plausible gas prices — all below the standing‑prover fixed cost, and it scales *down* toward $0 when idle.

**Conclusion for a nascent, low‑volume bridge:** do **not** run a standing prover. Use **aggregation** (Option 1 or 3), or if zk is wanted for other reasons, get it via **pay‑per‑proof**, never a standing instance. zk stops being a cost question and becomes an optional optimization.

⚠️ Every figure in this section is an **estimate** built from public GPU/proving prices and the gas anchors in §3. A measured Solidity FourQ verifier and a real cycle count would tighten them; nothing here should be quoted as a hard number without that.

---

## 5. Part B — a core change that returns margin to Vottun

Part A's cheapest, simplest form (Option 3) rests on one **additive** Qubic core change: **computors additionally co‑sign cross‑chain attestations in secp256k1‑Schnorr**, alongside their native FourQ signing.

- **Additive, never a replacement.** FourQ stays Qubic's universal identity and signature scheme for *everything* — every wallet, identity, transaction, and consensus signature. Replacing it is off the table (it would break the entire network). The secp256k1 key is a narrow, purpose‑scoped addition used *only* for EVM attestations, and can be **derived from each computor's existing seed** (no new secret to custody).
- **Fits Qubic's bare‑metal, no‑library ethos.** secp256k1 is a small, pairing‑free curve — hand‑implementable in the freestanding core, unlike a BLS pairing library. (Your team already hand‑rolled FourQ, the more intricate curve.)
- **The benefit:** Ethereum verifies the quorum with the native `ecrecover` precompile (~13k gas) instead of a custom FourQ verifier or a zk proof. **Lower operating cost → more margin to Vottun** — margin that (see Part C) creates room to fund restitution without anyone going underwater.

🛡️ New risks that belong in Part A's audit scope, stated up front: a second key per computor is a **new key‑management surface**; and the Ethereum‑side verifier must track the computor set as it **rotates each epoch** (the classic light‑client validator‑set‑rotation problem — real work, present for *any* trustless design, made cheap here by the native verify). None of this touches the consensus hot path.

---

## 6. Part C — a shared road to restitution (the token supply)

A framework, **not** a finished spec. The mechanism the parties would ratify and fill in:

1. **Incubation mints a capped, one‑time token supply** equal to the **assessed loss** — provably capped, mint key on a **multisig/timelock** (ideally renounced after mint) so affected users cannot be diluted.
2. **Affected users receive tokens** representing their verified claim.
3. The **Qubic‑side 0.5%** — the half the 676 computor‑shareholder smart contracts earn — is **diverted by autonomous on‑chain code** into a redemption pool that pays token holders. *(Vottun's half is the separate **ETH‑side 0.5%**, untouched unless Vottun chooses to share it — see §8.)*
4. When the pool has paid out the loss constant, the split **reverts to 90% shareholders / 10% retained by the token supply** — a **deliberate, lasting fund** that keeps ongoing resources and standing in the hands of affected parties, rather than dissolving once repayment ends.

### 🙏 Design decisions the parties must make (this is where honesty matters most)

We flag these because getting them wrong re‑escalates the very dispute this document tries to cool:

- **Who is an "affected user"?** Bridge‑locked holders at a specific **snapshot tick**, in‑flight transfers, wrapped‑token holders, LPs — these are different populations. Requires an agreed snapshot, an authoritative **pre‑incident** ledger, an **address‑ownership proof**, and rules for exchange‑custodied funds, lost keys, and partially‑withdrawn balances — plus a claims window and dispute process.
- **Denominate in what?** **Nominal QU (working figure 300B)** or **USD‑value‑at‑incident**? These diverge sharply with QU price; the on‑chain reversion counter must use the same unit and trigger on the agreed constant (300B QU serves as that number unless a more exact figure is set).
- **What does one token *represent*?** By design it is **both** a restitution claim (paid during repayment) **and** a lasting share of the retained **10% tail** — tokens **persist** rather than retire, deliberately keeping a standing resource with affected parties. ⚠️ This is also the feature most likely to give the token **investment‑contract** characteristics (see the legal note), so the perpetual tail is the key item for counsel. Decide the tail pool's **governance and permitted use** (e.g. ongoing distributions to holders or an affected‑party safety reserve).
- **Redemption ordering.** The pool is under‑funded throughout repayment, so ordering matters: **pro‑rata drip** is fairer than FIFO (rewards the fast) or lottery. Specify the token→QU rate and who pays claim gas.
- **Control and enforcement.** The diversion and the 90/10 reversion must be enforced by **autonomous contract code the operator cannot unilaterally alter** — otherwise restitution depends on trusting a party to the dispute, which contradicts the trustless goal. Because **Incubation is itself a disputant**, minting/governance should sit with a **multi‑party** multisig (user + Vottun + computor + Incubation representation), not Incubation alone.
- **⚠️ Securities / legal.** A token whose value derives from a **claim on future bridge revenue**, including the **deliberately retained perpetual 10% tail**, can resemble an **investment contract/note** in many jurisdictions. Distributing it *as restitution* mitigates this for the finite portion; the **perpetual tail carries the most risk** and is kept by design — so it is the key feature to clear with **regulatory counsel before finalizing**, alongside KYC/AML/sanctions/tax. The issuer (Incubation) likely inherits the liability. This is a flag for counsel, not a legal conclusion.
- **Transferable or soulbound?** Transferable = liquidity, but speculators can end up holding it (undercutting "help *affected* users" and worsening the securities optics). Soulbound/restricted‑for‑a‑period is the alternative. A real tradeoff to decide. Also decide what happens to **unclaimed** tokens.

---

## 7. The reality check (stated honestly)

The single most important section, because quiet optimism here is how stakeholders end up feeling misled — and re‑escalating.

**Repayment is funded only by redirected fees, and the bridge is at ~0 volume.** So affected users are made whole **in expectation, not in certainty**, and only *on paper* until volume returns — which requires trust, which is Part A. This is a circular dependency, and it makes **Part A the true keystone.**

**The arithmetic is sobering and belongs in daylight.** Repaying **300B QU** from a **0.5%‑of‑volume** diversion requires roughly:

> **~60 trillion QU of cumulative bridged volume** (`300,000,000,000 / 0.005`).

Illustrative years‑to‑repay (the parties must plug in real volume projections — these rows are hypothetical, not forecasts):

| Assumed monthly bridged volume | Cumulative volume / yr | Years to repay 300B QU |
|---:|---:|---:|
| ~0 (today) | ~0 | never |
| 100B QU/mo | 1.2T | ~50 |
| 1T QU/mo | 12T | ~5 |
| 5T QU/mo | 60T | ~1 |

Levers that shorten it (each a decision for the parties): a **temporary higher diversion** (e.g. Vottun voluntarily sharing part of its half — see §8), a **temporary higher bridge fee**, or **supplementary funding** so repayment isn't 100% volume‑contingent.

Two more honest caveats:
- **Secondary‑market haircut.** A long‑dated, uncertain claim will likely **trade well below face**, so users needing liquidity effectively take a discount. Acknowledge it rather than hide it.
- **Continuity risk.** The entire fee stream — and thus repayment — **dies if Vottun exits.** The plan needs an **assignment/continuity clause** so the obligation survives an operator change.

---

## 8. Why this works for everyone (the alignment)

The reason this is worth the effort: **every stakeholder's payoff points toward restoring volume**, which turns a blame fight into a shared growth objective.

| Stakeholder | What they get | The honest tension |
|---|---|---|
| **Affected users** | A funded, transparent claim and a real path toward being made whole | It's an illiquid, long‑dated, **volume‑contingent** instrument — better than a stalemate, short of instant restitution. Don't oversell it. |
| **Vottun** | Keeps its operator role; **higher margin** via Part B; trustless crypto that deflects future single‑point blame | Comes out clearly ahead — which is *good* for keeping Vottun engaged, but is also the plan's asymmetry (below). |
| **Computor shareholders** | Diversion costs ~**$0 today** (near‑zero volume); restored to **90%** of their share once the bridge thrives | They bear the whole cost and **must consent** (a quorum/governance vote — imposing it is neither fair nor likely feasible). The retained **10% tail** is a deliberate, permanent transfer to the affected‑party fund — the tradeoff they accept, and the main securities item for counsel. |
| **Incubation** | Clear direction and a constructive, visible role in the recovery | Inherits issuer/reputational risk; a sole‑Incubation‑controlled token also contradicts Part A's trustless ethos — hence multi‑party control. |

⚠️ **Neutrality is structural, not just lexical.** As drafted, shareholders fund 100% of repayment while Vottun's margin *rises* and its half is untouched — which, even with careful wording, can read as *"users paying for an incident they did not cause."* The clean way to defuse that is a **voluntary Vottun contribution** (even a temporary share of its **ETH‑side half**): it costs little at today's volume, materially shortens repayment when volume returns, and buys real goodwill and neutrality cover. We raise it as an option for Vottun to offer, not an obligation to impose.

---

## 9. Open questions — for Vottun, Incubation, and the computor set

This proposal **speaks for no one**. It becomes real only if the named parties ratify their parts. The questions that decide it:

**For Qubic core devs (Part A/B):**
1. Confirm the **per‑epoch checkpoint** cadence we've assumed as the logical default (vs. per‑transaction) — it drives the whole cost model.
2. Would the core add **additive secp256k1‑Schnorr co‑signing** for computors (derived from existing seeds), keeping FourQ native?
3. What is the **measured** gas of a Solidity FourQ verifier? (Replaces our ~350k–450k estimate with a fact.)
4. Who runs the outbound relayer / funds Ethereum gas, and what is the epoch computor‑set‑rotation update mechanism?

**For Vottun (Part A/C):**
5. Willing to continue operating a **trustless** bridge, and to consider a **voluntary contribution** to shorten repayment?
6. A **continuity/assignment** commitment so repayment survives an operator change?

**For Incubation + computor shareholders (Part C):**
7. Multi‑party (not sole‑Incubation) control of a **capped, autonomous** token/redemption contract?
8. A **shareholder consent** vote to divert the shareholder half, and the **governance and permitted use** of the retained **10% tail** (kept as a lasting affected‑party fund)?
9. **Regulatory counsel** engaged before the revenue‑claim design is locked?
10. The agreed **snapshot, ledger, denomination, and claims process** for defining affected users.

---

## 10. Scope reminder

🙏 We are outside contributors offering a **starting point, not a finished answer**. We take **no side** in the dispute and make **no claim** about how the incident occurred. Every mechanism here is a proposal for Vottun, Qubic Incubation, and the computor community to accept, reject, or reshape; every performance number is an **estimate** pending measurement. If a single idea survives contact with the people who own these decisions, this document did its job.

The through‑line is the only thing we'd ask everyone to hold onto: **a trustless bridge restores trust, restored trust restores volume, and restored volume is what makes affected users whole while keeping Vottun and the shareholders aligned.** That is a road less traveled — but it is one where the map points everyone in the same direction.
