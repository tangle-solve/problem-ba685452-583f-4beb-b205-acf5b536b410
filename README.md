<!-- solve-problem:ba685452-583f-4beb-b205-acf5b536b410 -->
# Endogenous timing-game participation

A [Tangle Solve problem](https://solve.tangle.tools/problems/ba685452-583f-4beb-b205-acf5b536b410).

## What is the problem?

In Ethereum's proof-of-stake consensus, validators (block proposers) choose *when* to sign and broadcast their block within their assigned slot (12 s, with attestations aggregating over ~4 s). Delaying the block proposal ("timing game") increases the time available for block builders — via relays under proposer-builder separation (PBS, e.g., MEV-Boost) — to stuff in more MEV, raising proposer rewards, but risks missed slots and weakens confirmation safety since attestations may land before the block. The 2023 baseline models treat timing as a static game with fixed participants and exogenous infrastructure.

This problem asks for a **dynamic/endogenous** theory: model a repeated timing game where validators can (i) *learn* about competitors' delay policies from observed on-chain timing data, (ii) *choose relays* (which enforce different timing deadlines/policies), and (iii) *invest in infrastructure* (low-latency hosting, colocated builder connections, faster signing) at a cost. Formally: a repeated game among a set of validators with heterogeneous costs cᵢ of latency reduction, action space of delay policies τᵢ: history → [0, slot length], relay selection rᵢ, and investment levels kᵢ; payoff = expected MEV share − infrastructure cost − slashing/missed-slot penalties. The target is to characterize **stable timing policies** — e.g., pure/Nash or evolutionary (congestion-game style) equilibria — and whether equilibrium delay is bounded or ratchets toward the attestation deadline ("everyone delays maximally, safety degrades").

Ambiguity note: "stable" is not formally defined in the census record; the solver must choose and justify a solution concept (Nash equilibrium of the repeated game, dynamically stable rest points, or evolutionary stability).

## Why does it matter?

If equilibrium timing games converge to maximal delay, Ethereum's effective confirmation safety margin shrinks (attestations land before blocks, enabling reorg pressure and 51%-style economical attacks), and reward distribution shifts to validators with the best infrastructure — a centralization force. Protocol designers currently tune relay deadlines (e.g., deneb-era proposer-safety constraints) without a predictive model; a solution would tell them which relay policy choices or protocol changes (honest-behavior incentives, deadline enforcement in clients) yield safe equilibria, and would let MEV researchers forecast PBS market structure rather than measure it after the fact.

## What's already known?

- Schwarz-Schelling et al. (2023) formalized static strategic timing games in PoS: proposers optimally delay; they identify equilibria and unsafe outcomes, and empirically document timing games in practice. This is the baseline — but timing choice is one-shot and infrastructure/relay choice is exogenous.
- Empirical work post-2023 documents widespread adoption of strategic delay by MEV-Boost proposers and relay deadline tightening; measurements exist of delay distributions by validator set. (Exact citations beyond the 2023 paper: verify.)
- Outside blockchain, the relevant machinery exists: **evolutionary congestion games** (players' strategy adjustment dynamics converge to equilibria under replication/best-response dynamics) and **endogenous speed/investment games** (e.g., literature on R&D racing and "faster horses" investment games, where equilibrium investment in speed can be socially excessive). Nobody has ported these to the PBS setting with relay choice as an intermediary.

Gap: no model where (a) timing policies are learned/adjusted over repeated slots, (b) relay deadline choice and validator participation feed back into timing incentives, and (c) infrastructure investment is endogenous.

## What would count as solving it?

A complete solution would be a model with proven equilibrium characterization. Concretely, any of:
1. **Existence + characterization**: prove existence of (dynamically stable) equilibria in a specified repeated timing game with relay choice and latency investment; characterize equilibrium delay and investment levels as functions of MEV intensity, infrastructure costs, and relay deadlines.
2. **Bounding result**: prove equilibrium delay is bounded away from the attestation deadline under some relay policy class (safety-preserving equilibria), or construct a counterexample showing delay ratcheting is unavoidable without protocol intervention.
3. **Comparative statics/policy**: prove which relay deadline policies or protocol changes yield delay-reducing equilibria, ideally with an evolutionary/congestion-game convergence proof for the learning dynamics.
4. **Empirical complement**: fit the model to observed validator timing data and test whether observed adoption dynamics match the predicted learning dynamics.

A referee wants: precise game definition, stated solution concept, theorems with proofs (or calibrated simulations with clear specification).

## How to start

1. Read Schwarz-Schilling et al. (2023) fully — Sections on the static game are the base model you extend; extract their payoff parametrization of delay (MEV accrual vs. miss probability) and reuse it.
2. Read the Ethereum PBS mechanics background (MEV-Boost architecture, relay deadline parameters, attestation timing in the consensus specs) so the timing/infrastructure action spaces are faithful — this is where blockchain-agnostic game theorists most often go wrong.
3. Start with a **two-stage reduction**: validators first choose relay + investment (one-shot), then play the Schwarz-Schilling timing game given those choices. Solve for subgame-perfect equilibria before attempting fully dynamic learning dynamics.
4. Then import machinery from evolutionary congestion games / best-response dynamics for the repeated-slot learning layer; the main obstacle is that delay payoffs are coupled through *others'* delay (congestion-like, since total slot MEV is roughly fixed and shares depend on relative timing), which may break standard potential-function arguments.

## Key references

- Schwarz-Schilling, C., Li, Y., Klausen, S., Pan, N., Song, Z., Kochat, A. (2023). "Time Is Money: Strategic Timing Games in Proof-of-Stake Protocols." *AFT 2023*. DOI: 10.4230/LIPIcs.AFT.2023.30. Formalizes static proposer timing games with equilibrium analysis and empirical evidence of strategic delay on Ethereum.
- Ethereum consensus / MEV-Boost documentation (relays, proposer deadlines, attestation timing) — needed to get the action spaces right; not a paper but the definitive source for protocol parameters. https://ethereum.org/en/roadmap/mev/ (verify URL currency)
- For the dynamics machinery: literature on evolutionary congestion games and dynamic stability (e.g., work following Sandholm's *Population Games and Evolutionary Dynamics*, 2010, MIT Press) — provides convergence tools for the learning layer. (Specific application papers: none yet — this is the open part.)

## Warnings

- "Stable" is undefined in the problem statement — pick a solution concept explicitly and defend it; reviewers will reject ambiguity here.
- Do not model infrastructure investment as simply reducing cost of delay; in reality it shifts the achievable delay frontier (colocation near builders/relays), which changes the game's structure.
- A common error is treating MEV as growing linearly in delay indefinitely; the 2023 paper's payoff structure caps this — reuse it.
- Relay behavior is itself strategic (relays compete for flow by offering later deadlines); a model with fixed relay policies may be criticized as incomplete — at minimum discuss this second-order effect.
- Empirical grounding matters: purely abstract results without Ethereum-realistic parameters are less likely to be accepted in this venue's literature.

---
*`BCGT-000314` · flagged for review*

## Acceptance guidance

A complete solution would be a model with proven equilibrium characterization. Concretely, any of:
1. **Existence + characterization**: prove existence of (dynamically stable) equilibria in a specified repeated timing game with relay choice and latency investment; characterize equilibrium delay and investment levels as functions of MEV intensity, infrastructure costs, and relay deadlines.
2. **Bounding result**: prove equilibrium delay is bounded away from the attestation deadline under some relay policy class (safety-preserving equilibria), or construct a counterexample showing delay ratcheting is unavoidable without protocol intervention.
3. **Comparative statics/policy**: prove which relay deadline policies or protocol changes yield delay-reducing equilibria, ideally with an evolutionary/congestion-game convergence proof for the learning dynamics.
4. **Empirical complement**: fit the model to observed validator timing data and test whether observed adoption dynamics match the predicted learning dynamics.

A referee wants: precise game definition, stated solution concept, theorems with proofs (or calibrated simulations with clear specification).

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/ba685452-583f-4beb-b205-acf5b536b410?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000314.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000314)
