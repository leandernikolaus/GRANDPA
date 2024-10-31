# Formal Finality

_Goal: Formally specify and verify finality gadgets_

- [Formal Finality](#formal-finality)
  - [About us](#about-us)
  - [Preliminary](#preliminary)
    - [Why This Project](#why-this-project)
  - [#1 Interface and Modeling](#1-interface-and-modeling)
    - [Interface and Properties](#interface-and-properties)
    - [Model and REPL Runs](#model-and-repl-runs)
    - [Safety in Finite Models](#safety-in-finite-models)
    - [Safety Proof](#safety-proof)
    - [Other Tools](#other-tools)
    - [Comparison](#comparison)
  - [#2 Refinement](#2-refinement)
  - [#3 Liveness](#3-liveness)
    - [Liveness to Safety Reduction](#liveness-to-safety-reduction)
    - [Fairness Conditions](#fairness-conditions)
  - [#4 Use](#4-use)
  - [References](#references)
  - [Our work](#our-work)
  - [Tools](#tools)

## About us

The project is led by associate professor Leander Jehl, a researcher at the University of Stavanger, Norway.
Leander did his PhD on fault tolerant distributed algorithms.
His research interests include decentralized systems, their algorithms and incentives.
He has a background in mathematical logic and has experience applying formal methods to analyzing distributed algorithms [HotStuff, HotStuff spec, Splitbt].

Daniel Dirdal is a PhD student just starting his graduate studies.
He did his master's thesis on setting up and evaluating a local testnet of Ethereum.
He is an enthusiastic student interested in web3 and is very keen to make a contribution with his work.

Leander and Daniel are part of the reliable systems group [RELAB](https://relab.website/) at the University of Stavanger and will be supported by other members of the group.

## Preliminary

The following presents different stages of the project.
Stage 1 is an initial study, serving as a foundation for the following stages.
Stages 2 to 4 can be done concurrently or in any order.
The description of stages 2 to 4 is intentionally less detailed compared to stage 1.
We plan to extend the description of these stages, based on our experience during stage 1.

### Why This Project

This project focuses on the study of finality gadgets, such as GRANDPA, in contrast to traditional consensus algorithms, for several key reasons:

- **Practical Relevance:** Finality gadgets are increasingly used in real-world decentralized systems.
  Formal verification of these algorithms will provide valuable insights, with potential for enhanced reliability and robustness.
- **Novel Interface to Consensus:** Finality gadgets offers a unique approach to the classic consensus problem.
  Formalizing their interface and defining their interactions with other system components, such as the fork-choice rule and block production, offers a new perspective on this class of algorithms and may lead to a deeper theoretical understanding.
- **Focus on GRANDPA:** We believe GRANDPA is well-suited for studying the properties and requirements of finality gadgets in general, because it operates decoupled from block production.
- **Interface Simplicity:** GRANDPA's decoupled design may allow a simpler interface for block production than other approaches, such as Ethereum's Casper FFG, potentially contributing to more efficient system designs.

## #1 Interface and Modeling

The goal of the first stage is to create a simplified model of GRANDPA that supports both generation of live traces and proving safety properties.

This stage will build on our prior experience with the formal verification of the [HotStuff algorithm](https://dl.acm.org/doi/pdf/10.1145/3293611.3331591s).
In our work on the HotStuff algorithm  [Hotstuff], we modeled consensus on a tree of blocks rather than the classical view instance matrix used by algorithms like Paxos and PBFT.
We adjusted the safety properties to fit this tree structure.
Instead of requiring that no two distinct values are decided in a single instance, we defined safety such that any two finalized blocks must be ancestors, preventing divergence into separate forks.

The main challenge we encountered when formally verifying tree-based consensus algorithms was the reliance on the ancestor relationship among blocks.
This is even more pronounced in GRANDPA, where a vote on one block counts as a vote for all of its ancestors.
In our previous work, we established lemmas and auxiliary invariants around this ancestor relationship, which will be instrumental in our formalization of GRANDPA.

### Interface and Properties

Our first goal is to define the interface of a finality gadget.
The description should include both properties which need to hold for the interface, as well as a description how the finality module fits in to the overall system.

### Model and REPL Runs

Second, we aim to formalize a first variant the algorithm and produce several runs, that show the model behaves as expected.
Many tools have a built-in read evaluate print loop for this purpose [Ivy, Quint].

It is especially important to produce traces that show the algorithm is live, e.g. a block can be finalized.
Produced runs should be maintained as test cases, to be run iteratively, when changing the model.

### Safety in Finite Models

The third goal is to verify safety in finite models.
This means to show for example that in a system with 4 validators, up to 3 blocks and running at most 3 rounds of GRANDPA, no two blocks on different forks are finalized.
Such analysis can be done by finite model checkers like TLC or Apalache.
Starting with a minimal viable example, the system size can be successively increased.

### Safety Proof

The last goal in this stage of the project is to verify safety in a parametrized system, i.e. without binding the number of validators or blocks or rounds.
In our previous work, we have derived such proofs both automatically, by using the Ivy tool, and by writing a manual, automatically verified, proof in TLAPS.
Other popular tools to derive such proofs are Coq and Dafny.

Ivy can helps a user to find an inductive invariant and thus prove safety properties.
This requires little manual work.
However, for a specification to work well with Ivy, it is advantageous to rewrite the specification, e.g. introducing free variables instead of existential quantification or replacing functions with predicates.
While this technique can allow to prove safety properties quite fast, the rewriting slightly obfuscates the specification.

TLAPS on the other hand allows to write manual proofs for safety properties and automatically verify the individual proof steps.
While this may require more effort than a proof in Ivy, it does not require rewriting of the specification.

We note that the tools can also be combined.
An inductive invariant found with Ivy can be used in a manual proof in TLAPS.
We have done this is our previous work on HotStuff.

### Other Tools

We hope to use our experience with TLAPS and Ivy and use these tools to verify GRANDPA.
However, we will also consider other tools.
First, K, Coq, and Dafny have all be used to verify Casper FFG.
We want to investigate if building on these existing proofs would simplify verifying GRANDPA.

Further, we want to investigate using Quint alongside TLA+.
While Quint does not support TLAPS or a full safety proof, it could be used for specification, REPL and finite model goals.
Quint claims to provide more developer friendly tooling than TLA+ and thus, having the specification available in Quint may be beneficial.

### Comparison

We hope to develop the specification of GRANDPA alongside a specification of Casper FFG, reproducing the REPL runs and Finite proofs also for Casper FFG.
Specifying the two alongside, following the same interface, allows us to better compare the two finality gadgets and understand their differences.

## #2 Refinement

We consider it useful to focus on the core functionality in the first stage, to more quickly produce a working model and proof.
At this stage we want to extend the model with additional mechanisms which are relevant to its correctness.

Additional mechanisms include:

- Equivocation detection
- Voter set changes
- Catching up

## #3 Liveness

Liveness is typically difficult to prove.
Tools like TLAPS and Ivy have been used to show liveness properties, but the support must be considered limited and experimental.
While we aim to explore the tools support for liveness, we plan to fall back on a liveness to safety reduction.

For must algorithms, liveness does not hold unconditionally, in an asynchronous system. Instead, certain fairness properties are needed for liveness to hold.
In practice, sufficiently many processes need to take steps to achieve liveness.
Since GRANDPA is designed to be _more live_ than other finality gadgets like Casper FFG, we plan to focus on these requirements needed to ensure liveness.

### Liveness to Safety Reduction

A liveness property may be formulated along the following:

> _If the fork-choice rule evaluated at different correct nodes eventually only proposes to vote on one branch, blocks on that branch will be finalized._

On the other hand, a safety property can be formulated along the following:

> _If during one round, the fork-choice rule evaluated at different correct nodes all points to a branch extending block `B`, then the votes cast during that round are sufficient to finalize block `B`._

A liveness to safety reduction implies that for each liveness property, we formulate safety properties like the above, which imply the liveness property.

### Fairness Conditions

As indicated by the examples above, liveness properties always require some fairness or preconditions.
Similarly, the safety properties gained after a safety to liveness reduction capture conditions which necessitate liveness.
Our goal is to investigate how much we can extend these conditions.
As shown by the example above, we believe these conditions also show the requirements a finality gadget poses to the fork-choice rule.
Comparing requirements given by the different finality gadgets will give useful insights into the properties and advantages of the different algorithms.

We note that the formal tools used in this project typically only allow preconditions to be formulated as a boolean predicate and do not include a probabilistic analysis.
However, a thorough analysis of the weakest precondition needed for a round to produce new finalizations, may be used in a future analysis quantifying the probability for a round to be live, based on the fork-choice rule.

## #4 Use

We want our project to be useful to the web3 community and would like our model to be connected to the real implementation.
We would like to use the model to generate test cases, which can be run on a local testnet deployment.
Similarly, it would be interesting to cover traces from a real deployment and verify that they correspond to possible executions of the specification.

Another possible use of our specification would be to update the existing GRANDPA specification.
Relying on a language like Quint, it would be possible that the specification indeed is the formal model.

Our research group has made good experience, integrating formal tools for testing.
For example, our open-source HotStuff framework includes an instance of the twin testing framework, to execute complex scenarios [Hotstuff impl].
The group also has experience with test case generation [Hein].

## References

## Our work
- [HotStuff] _Formal Verification of HotStuff_. In: FORTE 2021 [Paper](https://uis.brage.unit.no/uis-xmlui/bitstream/handle/11250/3054769/1968160_Jehl.pdf)
- [HotStuff spec] _Specification and proof in Ivy and TLAPS_ [HotStuff Spec](https://github.com/leandernikolaus/HotStuff-ivy)
- [Hotstuff impl.] _An Extensible Framework for Implementing and Validating
Byzantine Fault-tolerant Protocols_ [Paper](https://uis.brage.unit.no/uis-xmlui/bitstream/handle/11250/3076643/Pnr%252B2160906.pdf) and [Code](https://github.com/relab/hotstuff/)
- [SplitBFT] _SplitBFT: Improving Byzantine fault tolerance safety using trusted compartments_ [Paper](https://doi.org/10.1145/3528535.3531516) and [Proof](https://github.com/leandernikolaus/splitbft-proofs).

## Tools
- [Ivy] McMillan, Kenneth L., and Oded Padon. _Ivy: A multi-modal verification tool for distributed algorithms._ CAV 2020. [Paper](https://link.springer.com/content/pdf/10.1007/978-3-030-53291-8_12.pdf)
- [Quint] *A modern and executable specification language* https://quint-lang.org
- [TLA+ and TLC] Leslie Lamport. _The TLA+ Home Page_ https://lamport.azurewebsites.net/tla/tla.html
- [TLAPS] _The TLA+ Proof System_ https://proofs.tlapl.us


---
