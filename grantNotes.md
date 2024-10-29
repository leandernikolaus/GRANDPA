# Formal Finality

*Goal: Formally specify and verify finality gadgets*

- [Formal Finality](#formal-finality)
  - [About us](#about-us)
  - [Preliminary](#preliminary)
    - [Why this project](#why-this-project)
  - [#1 Interface and modeling](#1-interface-and-modeling)
    - [Interface and properties](#interface-and-properties)
    - [Model and REPL runs](#model-and-repl-runs)
    - [Safety in finite models](#safety-in-finite-models)
    - [Safety proof](#safety-proof)
    - [Other tools](#other-tools)
    - [Comparison](#comparison)
  - [#2 Refinement](#2-refinement)
  - [#3 Liveness](#3-liveness)
    - [Liveness to safety reduction](#liveness-to-safety-reduction)
    - [Fairness conditions](#fairness-conditions)
  - [#4 Use](#4-use)
  - [References](#references)


## About us

The idea for this project is driven by Leander Jehl, a researcher and associate professor at the University of Stavanger, Norway.
Leander did his PhD on fault tolerant distributed algorithms. His research interests include decentralized systems, their algorithms and incentives.
He has a background in mathematical logic and experience applying formal methods to distributed algorithms [Hotstuff, Hotstuff spec, Splitbt].

Daniel Dirdal is a novel PhD student. He did his master setting up and evaluating a local testnet of Ethereum. 
He is an enthusiastic about web3 and hopes to make a contribution with his work.

Leander and Daniel are part of the reliable systems group [RELAB](https://relab.website/) at the University of Stavanger and will be supported by other members of the group.

## Preliminary
The following presents different steps of the project.
The first step is an initial study. 
Steps 2 to 4 can be done concurrently or in any order.
The description of steps 2 to 4 is intentionally not as details as step 1. 
We plan to extend these, based on experiences made during step 1.

### Why this project

*Why do we think it is interesting to study finality gadgets, e.g. opposed to consensus algorithms?*

- Finality gadgets like GRANDPA are used in real systems at large scale. Formal verification of such algorithms can make a relevant contribution in practice.
- Finality gadgets present a novel interface and application of the classical consensus problem. 
A formalization of that interface together with the requirements that finality gadgets pose to other system components, like the fork choice rule and block production, can lead to better understanding of this class of algorithms.
- We believe studying GRANDPA is well suited to understand the properties and requirements of finality gadgets in general, since it is decoupled from block production. 
- This very likely allows to define a simpler interface with block production, than for example Casper FFG used in Ethereum.

## #1 Interface and modeling 

The goal for this first step is to create a simplified model of GRANDPA, that allows both to produce live traces and a proof of safety properties.


For this part we will heavily rely on our experience from the formal verification of the Hotstuff algorithm [1].
In our work on the Hotstuff algorithm, we did model consensus on a tree of blocks, rather than in the classical view instance matrix used in algorithms like Paxos, PBFT and others. 
Similarly we adjusted safety properties the tree structure. 
Instead of requiring that no two different values are decided in one instance, we require that any two finalized blocks are ancestors, i.e. due not lie on different forks.

We found that a main challenge in the formal treatment of consensus algorithms working on a tree-model is that they heavily rely on the ancestor relation between blocks in the tree. 
This is even more the case for Grandpa, where a vote on one block counts as a vote for the blocks ancestors. 
In our work, we already proved lemmas and defined auxiliary invariants about the ancestor relation, that will be similarly useful in the formalization of Grandpa.


### Interface and properties

The goal for this step is to define the interface of a finality gadget.
The description should include both properties which need to hold for the interface, as well as a description how the finality module fits in to the overall system.

### Model and REPL runs

The first step is to formalize a first variant the algorithm and produce several runs, that show the model behaves as expected. 
Many tool have a built-inn read evaluate print loop for this purpose [Ivy, Quint].

It is especially important to produce traces that show the algorithm is live, e.g. a block can be finalized. 
Produced runs should be maintained as test cases, to be run iteratively, when changing the model.

### Safety in finite models

The second step is to verify safety in finite models. 
This means to show for example that in a system with 4 validators, up to 3 blocks and running at most 3 rounds of Grandpa, no two blocks on different forks are finalized. 
Such analysis can be done by finite model checkers like TLC or Apalache.
Starting with a minimal viable example, the system size can be successively increased.

### Safety proof

The last step in this part of the project is to verify safety in a variable system, i.e. without binding the number of validators or blocks or rounds.
In our previous work, we have derived such proofs both automatically, by using the Ivy tool, and by writing a manual, automatically verified, proof in TLAPS.
Other popular tools to derive such proofs are Coq and Dafny.

Ivy can helps a user to find an inductive invariant and thus prove safety properties. 
This requires little manual work. 
However, for a specification to work well with Ivy, it is advantageous to rewrite the specification, e.g. introducing free variables instead of existential quantification or replacing functions with predicates.
While this technique can allows to prove safety properties quite fast, the rewriting slightly obfuscates the specification.

TLAPS on the other hand allows to write manual proofs for safety properties and automatically verify the individual proof steps.
While this may require more effort than a proof in Ivy, it does not require rewriting of the specification.

We note that the tools can also be combined. An inductive invariant found with Ivy can be used in a manual proof in TLAPS. 
We have done this is our previous work on Hostuff.

### Other tools

We hope to use our experience with TLAPS and Ivy and use these tools to verify Grandpa.
However, we will also consider other tools.
First, K, Coq, and Dafny have all be used to verify Casper FFG. 
We want to investigate if building on these existing proofs would simplify verifying Grandpa.

Further, we want to investigate using Quint alongside TLA+. 
While Quint does not support TLAPS or a full safety proof, it could be used for specification, REPL and finite model steps. 
Quint claims to provide more developer friendly tooling than TLA+ and thus, having the specification available in Quint may be beneficial.

### Comparison

We hope to develop the specification of Grandpa alongside a specification of Casper FFG, reproducing the REPL runs and Finite proofs also for Casper FFG. 
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
Since Grandpa is designed to be *more live* than other finality gadgets like Casper FFG, we plan to focus on these requirements needed to ensure liveness. 

### Liveness to safety reduction

A liveness property may be formulated along the following:
> *If the fork-choice rule evaluated at different correct nodes eventually only proposes to vote on one branch, blocks on that branch will be finalized.*

On the other hand, a safety property can be formulated along the following:
> *If during one round, the fork-choice rule evaluated at different correct nodes all points to a branch extending block `B`, then the votes cast during that round are sufficient to finalize block `B`.*

A liveness to safety reduction implies that for each liveness property, we formulate safety properties like the above, which imply the liveness property.

### Fairness conditions

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

Another possible use of our specification would be to update the existing Grandpa specification. 
Relying on a language like Quint, it would be possible that the specification indeed is the formal model.

Our research group has made good experience, integrating formal tools for testing. 
For example, our open-source Hotstuff framework includes an instance of the twin testing framework, to execute complex scenarios.
The group also has experience with test case generation [Hein].

## References

- [Hotstuff] *Formal Verification of HotStuff*. In:  FORTE 2021 [https://doi.org/10.1007/978-3-030-78089-0_13](https://doi.org/10.1007/978-3-030-78089-0_13) 
- [Hotstuff spec] *Specification and proof in Ivy and TLAPS* [https://github.com/leandernikolaus/hotstuff-ivy](https://github.com/leandernikolaus/hotstuff-ivy)
- [SplitBFT] *SplitBFT: Improving Byzantine fault tolerance safety using trusted compartments* [Paper](https://doi.org/10.1145/3528535.3531516) and [Proof](https://github.com/leandernikolaus/splitbft-proofs)
- 