# Concepts regarding attacks on Merkle commitments

We are going to attack the logic behind Merkle proofs and find out why the attested security is half the bits of the digest of the hash function chosen.

## Birthday paradox

Say we have $n$ people. What's the probability at least two of them share their birthdays [ we call this problem $A$ ] ? Let's make a simple trick. We calculate the probability none of them are born the same day [ we call this problem $\bar{A}$ ]. The probability of the former problem will be

```math
\text{Pr}(A) = 1 - \text{Pr}(\bar{A})
```

Assuming every year is made of $365$ days we have

```math
\text{Pr}(\bar{A}) = \frac{365}{365} \cdot \frac{364}{365} \cdot \frac{363}{365} \cdot \dots \cdot \frac{365 - (n - 1)}{365} = \frac{365!}{365^{n}(365 - n)!}
```

because:

>- $1st$ guy enters the room, the probability he doesn't share his birthday with anyone is $1$ because he's alone.
>- $2nd$ guy enters the room, the probability he doesn't share his birthday with the $1st$ one is $364$ days over $365$ [ $\frac{364}{365}$ ].
>- $3rd$ guy enters the room, the probability he doesn't share his birthday with the $1st$ and the $2nd$ ones is $363$ days over $365$ [ $\frac{363}{365}$ ].
>- $\dots$
>- $n-th$ guy enters the room, the probability he doesn't share his birthday with the others is $365 - (n - 1)$ days over $365$ [ $\frac{366 - n}{365}$ ].

Now

```math
\text{Pr}(A) = (1 - \frac{365}{365}) \cdot (1 - \frac{364}{365}) \cdot (1 - \frac{363}{365}) \cdot \dots \cdot (1 - \frac{365 - (n - 1)}{365}) = \frac{1}{365} \cdot \frac{2}{365} \cdot \dots \cdot \frac{n - 1}{365} = \frac{(n - 1)!}{365^{n - 1}}
```

because:

>- $1st$ guy enters the room, the probability he shares his birthday with anyone is $1 - 1 = 0$ because he's alone.
>- $2nd$ guy enters the room, the probability he shares his birthday with the $1st$ one is $1$ day over $365$, that is $1 - \frac{364}{365} = \frac{365 - 364}{365} = \frac{1}{365}$.
>- $3rd$ guy enters the room, the probability he shares his birthday with the $1st$ and the $2nd$ ones is $2$ days over $365$, that is $1 - \frac{363}{365} = \frac{365 - 363}{365} = \frac{2}{365}$.
>- $\dots$
>- $n-th$ guy enters the room, the probability he shares his birthday with the others is $n - 1$ days over $365$, that is $1 - \frac{366 - n}{365} = \frac{365 - 366 + n}{365} = \frac{n - 1}{365}$.

## Linking the birthday paradox to hash-collisions

Back to

```math
\text{Pr}(\bar{A}) = \frac{365}{365} \cdot \frac{364}{365} \cdot \frac{363}{365} \cdot \dots \cdot \frac{365 - (n - 1)}{365}
```

we can observe that

```math
\text{Pr}(\bar{A}) = (1 - \frac{1}{365}) \cdot (1 - \frac{2}{365}) \cdot \dots \cdot (1 - \frac{n - 1}{365})
```

and that $\text{Pr}(\bar{A})$ means "probability of having no collisions on birthdays having $n$ guys". This means that the former, in an hash function scenario where we handle $160$ bits digests, can be rewritten as

```math
\text{Pr}(\text{no\_collision\_on\_n\_hashes}) = (1 - \frac{1}{2^{160} - 1}) \cdot (1 - \frac{2}{2^{160} - 1}) \cdot \dots \cdot (1 - \frac{n - 1}{2^{160} - 1})
```

**Theorem**

```math
\lim_{x \to \infty} (1 + \frac{1}{x})^{x} = e
```

**Proof**

Let's expand $(1 + \frac{1}{x})^{x}$ using Newton's binom.

```math
\lim_{x \to \infty} (1 + \frac{1}{x})^{x} = \sum_{k = 0}^{x} \binom{x}{k} 1^{k}(\frac{1}{x})^{x - k} = \sum_{k = 0}^{x} \binom{x}{k} (\frac{1}{x})^{x - k}
```

We can rewrite the last one as

```math
\lim_{x \to \infty} \sum_{k = 0}^{x} \frac{x!}{k!(x - k)!}(\frac{1}{x})^{k} = \sum_{k = 0}^{x} \frac{x!}{k!(x - k)!}(\frac{1}{x^{k}}) = \sum_{k = 0}^{x} \frac{1}{k!}\frac{x(x - 1)(x - 2) \dots (x - k + 1)}{x^{k}}
```
