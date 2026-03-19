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

and that $\text{Pr}(\bar{A})$ means "probability of having no collisions on birthdays, having $n$ guys". This means that the former, in an hash function scenario where we handle $160$ bits digests, can be rewritten as

```math
\text{Pr}(\text{no\_collision\_on\_n\_hashes}) = (1 - \frac{1}{2^{160} - 1}) \cdot (1 - \frac{2}{2^{160} - 1}) \cdot \dots \cdot (1 - \frac{n - 1}{2^{160} - 1})
```

Now we can observe that (if you don't understand why $e^{x}$ equals that, go to [https://github.com/Z323323/Complex-numbers-background]), recalling

```math
e^{x} = \sum_{n = 0}^{\infty} \frac{x^{n}}{n!} = \frac{x^{0}}{0!} + \frac{x^{1}}{1!} + \frac{x^{2}}{2!} + \frac{x^{3}}{3!} + \dots = \frac{1}{1} + \frac{x}{1} + \frac{x^{2}}{2} + \frac{x^{3}}{3!} + \dots = 1 + x + \frac{x^{2}}{2} + \frac{x^{3}}{3!} + \dots
```

if we set $x = \frac{1}{u}$ where $\lim_{u \to \infty}$, then

```math
e^{- x} = \lim_{u \to \infty} e^{- \frac{1}{u}} = \sum_{n = 0}^{\infty} \frac{(- \frac{1}{u})^{n}}{n!} = \frac{(- \frac{1}{u})^{0}}{0!} + \frac{(- \frac{1}{u})^{1}}{1!} + \frac{(- \frac{1}{u})^{2}}{2!} + \frac{(- \frac{1}{u})^{3}}{3!} + \dots = \frac{1}{1} - \frac{\frac{1}{u}}{1} + \frac{(\frac{1}{u})^{2}}{2} - \frac{(\frac{1}{u})^{3}}{3!} + \dots = 1 - \frac{1}{u} + \frac{1}{2u^{2}} - \frac{1}{3!u^{3}} + \dots \approx 1 - \frac{1}{u}
```

[ because $\frac{1}{u} \to 0$ ]. This finally means that we can consider

```math
\text{Pr}(\text{no\_collision\_on\_n\_hashes}) = (1 - \frac{1}{2^{160} - 1}) \cdot (1 - \frac{2}{2^{160} - 1}) \cdot \dots \cdot (1 - \frac{n - 1}{2^{160} - 1}) \approx (1 - x) \cdot (1 - \frac{2}{2^{160} - 1}) \cdot \dots \cdot (1 - \frac{n - 1}{2^{160} - 1})
```
