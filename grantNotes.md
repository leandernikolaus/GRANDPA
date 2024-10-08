# Formal Finality

*Goal: Formally specify and verify finality gadgets*

## Preliminary

*Why do we think it is interesting to study finality gadgets, e.g. opposed to consensus algorithms?*


## #1 Interface and modeling 

Build on tree from Hotstuff specification.
Build on safety properties from Hotstuff.

Try different modeling frameworks to build a model of simplified Granpa. 

### Modeling tools

Talk about advantages of different tools that we want to try:
- Ivy
- TLA+ and TLAPS
- Quint
- others?

Ethereum Beacon chain verified in K, Coq and Dafny.
Compare with verifications of Casper FFG.

- **Result:** Verification result safety.

- Properties needed from Fork choice rule and block generation.

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