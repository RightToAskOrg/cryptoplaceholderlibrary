# cryptoplaceholderlibrary
Crypto API placeholders with best-effort implementations.

We need code for generating a 0 or 1 vote, and proving that it contains either 0 or 1. ElectionGuard uses prime fields; not clear that we would also need to do so if we were using a different library, but I have left them in that form for now.

## Parts of ElectionGuard we need

Based on [Version 1.0 of the EG Spec](https://github.com/microsoft/electionguard/releases/download/v1.0/EG_spec_v1_0.pdf).

We need a special case of [Ballot Encryption](https://www.electionguard.vote/spec/web/6_Ballot_Encryption/) in which each ballot (which is an up-vote or dismiss-vote on a question) has only one entry, which is either 0 or 1.

Unfortunately, Microsoft uses an extra ciphertext as a 'placeholder' value, because this simplifies the whole ballot-validity proof - they only need to prove that the total number of ones is *equal* to the limit. So in order to be exactly compliant with the EG standard, we would effectively have a (1,0) ciphertext tuple for an up-vote and a (0,1) ciphertext tuple for a dismiss-vote.

This placeholder is not necessary for our actual ballot proofs, because the single Chaum-Pedersen proof suffices (it's either a zero or a one). 

TODO: Consider whether according with EG spec is worth doubling the work.

###El Gamal encryption, exponential form

An El Gamal encryption of a zero is computed from public key $K$ and parameters $(g,p,q)$ by selecting R with $`0 \leq R < q`$ and setting

```math
C_0 = (g^R \text{ mod } p, K^R \text{ mod } p).
```

Encryption of one is

$$ C_1 = (g^R \text{ mod } p, g . K^R \text{ mod } p).$$

**CJC: I thought it was supposed to be Exponential ElGamal?**

Implemented in EG [here](https://github.com/microsoft/electionguard-cpp/blob/main/bindings/netstandard/ElectionGuard/ElectionGuard.Encryption/ElGamal.cs).




###DisjunctiveChaumPedersenProof


C++ implementation [here](https://github.com/microsoft/electionguard-cpp/blob/main/src/electionguard/chaum_pedersen.cpp).

Like (the rest of) ElectionGuard, I don't think we actually need a proof for either disjunct, i.e. we never need to prove that it's a 0 or prove that it's a 1 - we only need the disjunctions.

###Summary of what EG would do
For a single-option ballot, EG would have two
ciphertexts and three Chaum-Pedersen proofs:

- a ciphertext C for the chosen selection: 1 for 'selected' and 0 for 'not selected.'
- a placeholder ciphertext P, the complement of the actual choice
- a Chaum-Pedersen proof, for each of C and P, that they contain either a 0 or a 1.
- a Chaum-Pedersen proof that the homomorphic combination C.P contains a 1.

###Possible simplification of ballot

The obvious simplification for our ballots, which would make us noncompliant with EG but save us more than a factor of two of work, storage and communication, is to have one ciphertext and one proof:

- a ciphertext C for the chosen selection: 1 for 'selected' and 0 for 'not selected.'
- a Chaum-Pedersen proof, for   C, that it contains either a 0 or a 1.

## Interface Requirements
In terms of defining the interface, what is required is the actual method signatures, as this will define where the parameters are coming from. For example:

`BallotObj createBallot(PublicKeyObject, Choice(Boolean))`

### Objects
```
BallotObject:
  - Ciphertext C
  - Ciphertext P
  - Proof C
  - Proof P
  - Proof C.P
Choice:
  - True for 1
  - False for 0
PublicKeyObject:
  - g
  - pk
  - p
  - q
 ```
 It needs to define what parameters will be passed in and what object/values will be returned. Depending on how it is currently implemented it might be easy to just replace with a single method as above, or if it currently calls multiple methods in ElectionGuard then the granularity of of the method calls could be increased to match.
