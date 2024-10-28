# Formal Finality

*Goal: Formally specify and verify finality gadgets*

## Preliminary

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

At this stage we want to model mechanisms which have been omitted at the first stage. 
Additional mechanisms may include:
- Equivocation detection
- Voter set changes
- Catching up

## #3 Liveness

First approach liveness to safety reduction, only formally verify safety property.
Give example.

Investigate liveness condition, for different finality gadgets.

Second approach, tools suitable for liveness verification.

## Use

Describe our goal to put specifications to use, e.g. generate test cases, analyse traces against the model, ...

Erfaring: Test case generation (Hein), Twins implementation