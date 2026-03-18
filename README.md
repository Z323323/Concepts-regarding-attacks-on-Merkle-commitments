# Concepts regarding attacks on Merkle commitments

We are going to attack the logic behind Merkle proofs and find out why the attested security is half the bits of the digest of the hash function chosen.

## Birthday paradox

Say we have $n$ people. What's the probability at least two of them have their birthdays on the same day [ we call this problem $A$ ]? Let's make a simple trick. We calculate the probability none of them are born the same day [ we call this problem $\bar{A}$ ]. The probability of the former problem will be

```math
\text{Pr}(A) = 1 - \text{Pr}(\bar{A})
```

