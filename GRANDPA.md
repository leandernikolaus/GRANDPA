
Notes:
- $B' > B$ means B' descendant of B.
- $H_i(B)$ is the level a Block has in the tree.

## Questions:

- What is $H_n$ used in Alg16 and Alg17 (also used in [Alg1](https://spec.polkadot.network/chap-state#algo-validate-transactions), but not defined in [Def10](https://spec.polkadot.network/chap-state#algo-validate-transactions))

- Are the changes to the Voter set fundamental to the algorithm, or can we assume a fixed voter set for simplification?
- [Def82](https://spec.polkadot.network/sect-finality#defn-sign-round-vote) Why is the stage not part of the message. If you get a signature, can you identify what stage it is?
- [Def84](https://spec.polkadot.network/sect-finality#defn-finalizing-justification) Why would you have different rules for finalization for non-voter and voter nodes. 
- [Def85](https://spec.polkadot.network/sect-finality#defn-voter-equivocation) $M^{r,stage}_v$ is not defined. What is it?
- [Def88](https://spec.polkadot.network/sect-finality#defn-total-potential-votes) 
- The min term is strange. Do they count equivocation as potential votes?
- [Def90](https://spec.polkadot.network/sect-finality#defn-grandpa-completable) I do not understand this one. Wondering if one of the inequalities is turned the wrong way. 
A vote for a descendant B' is also a vote for B. 
- [Alg16](https://spec.polkadot.network/sect-finality#algo-grandpa-best-candidate) I cannot find a definition for $H_n$, should this be $H_i$? Same counts in Alg17.
- [Alg18](https://spec.polkadot.network/sect-finality#algo-best-prevote-candidate) Is $L$ the BestFinalCandidate(r-1) from Alg17?
