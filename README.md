# Concepts regarding attacks on Merkle commitments

We are going to attack the logic behind Merkle proofs and find out why the attested security is half the bits of the digest of the hash function chosen. We are also going to understand why Merkle trees and hash functions are so damn powerful so that we can't fucking attack anything.

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

[ because $\frac{1}{u} \to 0$ ] where

```math
1 - \frac{1}{u} \rightarrow 1 - x
```

This finally means that whenever $x \to 0$ we can consider

```math
e^{- x} \approx 1 - x
```

Back to our hash collisions scenario, renaming $2^{160} - 1 = N$ we can finally see that

```math
\text{Pr}(\text{no\_collision\_on\_n\_hashes}) = (1 - \frac{1}{N}) \cdot (1 - \frac{2}{N}) \cdot \dots \cdot (1 - \frac{n - 1}{N}) \approx e^{- \frac{1}{N}} \cdot e^{- \frac{2}{N}} \cdot \dots \cdot e^{- \frac{n - 1}{N}} = \prod_{i = 1}^{n - 1} e^{- \frac{i}{N}}
```

and

```math
e^{- \frac{1}{N}} \cdot e^{- \frac{2}{N}} \cdot \dots \cdot e^{- \frac{n - 1}{N}} = e^{- (\frac{1 + 2 + \dots + n - 1)}{N})}
```

Recalling the Gauss formula

```math
\sum_{i = 1}^{n} i = \frac{n(n + 1)}{2}
```

we have

```math
1 + 2 + \dots + n - 1 = \sum_{i = 1}^{n - 1} i = \frac{n(n - 1)}{2}
```

and

```math
e^{- (\frac{1 + 2 + \dots + n - 1)}{N})} = e^{- (\frac{n(n - 1)}{2N})} \approx e^{- (\frac{n^{2}}{2N})} = \frac{1}{e^{\frac{n^{2}}{2N}}}
```

so that

```math
\text{Pr}(\text{no\_collision\_on\_n\_hashes}) = \frac{1}{e^{\frac{n^{2}}{2N}}}
```

Extracting now

```math
\frac{n^{2}}{2N}
```

we can see that the probability of having no hash collision **decreases quadratically on $n$ attempts**. This is finally why it is called a _"paradox"_. Let's now calculate **the number of attempts ($n$) needed in order to achieve the $80\\%$ probability of a collision (we would be almost sure to hit one)**. This means we need to compute **the $20\\%$ probability of not having an hash collision**. Now

```math
20\% = e^{- (\frac{n^{2}}{2N})}
```
```math
\log_{e}(0.2) = - \frac{n^{2}}{2N}
```
```math
--
```
```math
\log_{e}(0.2) = \log_{e}(1/5) = \log_{e}(1) - \log_{e}(5) = 0 - 1.609437912 = - 1.609437912
```

For these last equations refer to [https://github.com/Z323323/Logarithm-rules].

```math
--
```
```math
- 1.609437912 = - \frac{n^{2}}{2N}
```
```math
n^{2} = 2N(1.609437912)
```
```math
n = \sqrt{2N}\sqrt{1.609437912}
```

Again, this clearly shows how the number of attempts required to be almost sure [ $80\\%$ ] to hit a collision is roughly the square root of all the possible values $N$ we could check. If $N = 2^{160}$ then

```math
\sqrt{N} = \sqrt{2^{160}} = 2^{160 / 2} = 2^{80}
```

which finally somehow justifies why people write the security of hash functions is half the bits of the digest.

Now we need to understand why it is so simple to hit one collision, and therefore what actually the paradox states. This is important in order to think of some collision attack.

## The _"paradox"_ explained

In order to understand why it is so simple to obtain a collision we need to understand that the _birthday paradox_ calculates the probability anyone in the group of people has to be born the same day of anyone else, i.e. **for each person birthday, we should check all other people birthdays**. That's why a collision is so likely to happen.

## From the _"paradox"_ to the real world

Now that we know basically everything about the _birthday paradox_, we should ask ourselves **_"how the actual fuck would I check each hash with all the others in a real scenario?"_**. Well, there are algorithms like _turtoise and hare_ (Robert W. Floyd) which try to solve the problem of memory occupation. They try to find collisions building only 2 "paths" of hashes and trying to find the "period" of the function, and after that, they try to rebuild the point of collision. The main problem of these kind of algorithms is that the hash "paths" are only buildable using the form $H(H(H(\dots(x)\dots)))$, and for really secure stuff like SHA256 etc. it's not feasible because the "periods" are too long to compute. Even if we talk about an _hare_, we are talking about an _hare_ which needs to run from Earth to the end of the galaxy.

Another approach which I would follow is the pure one. Say we have SHA256, so 256 bits digests, so 32 bytes. We exploit the $n$ value. That is, we use as much RAM as possible to compute as many hashes as possible starting from a crazy long vector of random seeds. For each step we compute $H$ on every seed, then, we use the fastest way possible to compute `next_state - prev_state = state_diff` (hyper parallelization required here). After that, we could try `r = 1; r = r * each_256_bits_element_of_the_state_diff_vector`. If we get $r = 0$ then we have a collision, otherwise we don't. But ig there are faster methods available (like autoflag a $0$ buffer from HW). So, say we could afford the previous 2 problems in O(1), what's the main problem now? Having 1TB of RAM (kinda optimistic) we could handle

```math
\frac{1TB}{(2 \cdot 32)B} = \frac{2^{40}B}{2^{6}B} = 2^{34}
```

hashes for each vector (`next_state, prev_state`). Now, if we apply the _"birthday paradox"_ we get

```math
\text{Pr}(\text{collision}) = 1 - \frac{1}{e^{\frac{n^{2}}{2N}}} = 1 - \frac{1}{e^{\frac{(2^{34})^{2}}{2(2^{256})}}} = 1 - \frac{1}{e^{\frac{2^{68}}{2^{257}}}} = 1 - \frac{1}{e^{\frac{1}{2^{189}}}} = 1 - \frac{1}{\sqrt[189]{e}} \approx 1 - \frac{1}{1} = 1 - 1 = 0
```

