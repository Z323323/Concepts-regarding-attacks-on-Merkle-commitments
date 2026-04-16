# Concepts regarding attacks on Merkle commitments

We are going to attack the logic behind Merkle proofs and find out why the attested security is half the bits of the digest of the hash function chosen. We are also going to understand why Merkle trees and hash functions are so damn powerful so that we can't fucking attack (almost) anything.

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
e^{- \frac{1}{N}} \cdot e^{- \frac{2}{N}} \cdot \dots \cdot e^{- \frac{n - 1}{N}} = e^{- (\frac{1 + 2 + \dots + n - 1}{N})}
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

## The _paradox_ explained

The _birthday paradox_ calculates the probability anyone in the group of people has to be born the same day of anyone else, i.e. **for each person birthday, we should check all other people birthdays**. That's why a collision is so likely to happen.

## From the _paradox_ to the real world

Now that we know basically everything about the _birthday paradox_, we should ask ourselves **_"how would we check each hash with all the others in a real scenario?"_**. Well, there are algorithms like _turtoise and hare_ (Robert W. Floyd) which try to solve the problem of memory occupation. They try to find collisions building only 2 "paths" of hashes and trying to find the "period" of the function, and after that, they try to rebuild the point of collision. The main problem of these kind of algorithms is that the hash "paths" are only buildable using the form $H(H(H(\dots(x)\dots)))$, and for really secure stuff like SHA256 etc. it's not feasible because the "periods" are too long to compute. Even if we talk about an _hare_, we are talking about an _hare_ which needs to run from Earth to the end of the galaxy. I won't delve these kind of algorithms but it would be interesting to find some fusion mechanism between this approach and the next (trying to find the best compromise between CPU speed and memory occupation).

Another approach which I would follow is the pure one. Say we have SHA256, and so 256 bits digests (32 bytes). We exploit the $n$ value, that is, we use as much RAM as possible to compute as many hashes as possible starting from a long vector of random seeds. In the first step we simply ensure the random seeds are all different (using the same tactic we are going to see now). For each step we compute `H` on `prev_state`, that is, on every single 256 bits var (hyper parallelization), then, we check if, using the hash table mechanism, `H(prev_var)` (the result) is equal to the previous value at the address `H_{table}(prev_var)`. If it is, then we have a collision, otherwise we don't. If we have one then the process is over, otherwise we'll overwrite the memory location `H_{table}(prev_var)`. In order to check the last step, we should extract `prev_var` from memory at addr `H_{table}(prev_var)` [ $O(1)$ but I/O is slow ] and load it into CPU registers, then compute

```
H(x_{prev_state}) XOR y_{prev_state} <- read(H_{table}(y_{prev_state})) 
```

and check if the result is $0$. If it is, then we have a collision. Otherwise we don't and we write `H(x_{prev_state})` into `H_{table}(x_{prev_state}) = H_{table}(y_{prev_state})`.

Say we could afford the previous computations in negligible time; what's the main problem now? Having 1TB of RAM (kinda optimistic) we could handle

```math
\frac{1TB}{32B} = \frac{2^{40}B}{2^{5}B} = 2^{35}
```

hashes. Now, if we apply the _birthday paradox_ we get

```math
\text{Pr}(\text{collision}) = 1 - \frac{1}{e^{\frac{n^{2}}{2N}}} = 1 - \frac{1}{e^{\frac{(2^{35})^{2}}{2(2^{256})}}} = 1 - \frac{1}{e^{\frac{2^{70}}{2^{257}}}} = 1 - \frac{1}{e^{\frac{1}{2^{187}}}} = 1 - \frac{1}{\sqrt[2^{187}]{e}} = 1 - \frac{1}{1} = 1 - 1 = 0
```

probability of messing with SHA256 using some Google supercomputer.

## Tryharding in the sci-fi scenario

What would happen to probability if we kept repeating the previous computation?

Since we are literally erasing `prev_state` at each step (overwriting), once we compute `current_state` and overwrite, `next_state` won't be compared to `prev_state`. This breaks the powerful mechanism which would enable us to obtain more power on the collision problem. If we'd be able to create a linkage between states without adding space required for computation (the actual bottleneck is RAM and I/O speed) we'd be perhaps able to pose a threat to security. The problem is this is probably not possible. I think Claude Shannon wrote something about this.

>- **Probability if we'd be able to create a linkage**
>- $Pr(collision\\_step\\_1) = 1 - e^{- ((2^{35})^{2} / 2^{257})} = 1 - e^{- (2^{70} / 2^{257})}$
>- $Pr(collision\\_step\\_2) = 1 - e^{- ((2^{35} + 2^{35})^{2} / 2^{257})} = 1 - e^{- ((2^{36})^{2} / 2^{257})} = 1 - e^{- (2^{72} / 2^{257})}$
>- $\vdots$
>- $Pr(collision\\_step\\_2^{60}) = 1 - e^{- ((2^{35} + 2^{35} + 2^{35} + 2^{35} \dots)^{2} / 2^{257})} = 1 - e^{- ((2^{35 + 60})^{2} / 2^{257})} = 1 - e^{- ((2^{95})^{2} / 2^{257})} = 1 - e^{- (2^{190} / 2^{257})}$
>- $\vdots$
>- $Pr(collision\\_step\\_2^{93}) = 1 - e^{- ((2^{35} + 2^{35} + 2^{35} + 2^{35} \dots)^{2} / 2^{257})} = 1 - e^{- ((2^{35 + 93})^{2} / 2^{257})} = 1 - e^{- ((2^{128})^{2} / 2^{257})} = 1 - e^{- (2^{256} / 2^{257})} = 1 - e^{- (1 / 2)} = 1 - \frac{1}{\sqrt{e}} = 1 - \frac{1}{1,648721271} = 1 - 0,60653066 = 0,39346934 \approx 40\\%$

Basically, in order to break one single hash of SHA256 we'd need the computation power of the whole umanity for more than $1$ year + we should break the laws of reality. Gemini says the actual hash (SHA256) power of humanity in an year is around $2^{94}$ trials. I'm not checking this result but I'll believe it's correct since SHA256 and this particular stuff is quite famous because of Bitcoin. We should by the way restrict this result a lot because of I/O bottleneck. SHA256 is literally god's power proof.

## Tryharding in the real scenario

We could eventually try to exploit many Google supercomputers and use a central server only to check each hash result to check a collision between many $1TB$ RAMs. Having $256 = 2^{8}$ such computers we could reach

```math
\text{Pr}(\text{collision}) = 1 - e^{- ((2^{35} \cdot 2^{8})^{2} / 2^{257})} = 1 - e^{- ((2^{43})^{2} / 2^{257})} = 1 - e^{- (2^{86} / 2^{257})} = 0
```

This means we are either going to find some crazy agorithm which probably doesn't even exist or we can't simply break SHA256 using normal computers. Now the problem resides in QCs and/or particular math manipulations of hash functions.

## Brainless bruteforce

Say we just keep repeating trials. In this case, we would simply need $\approx 2^{186}$ attempts in the supercomputer scenario, and $\approx 2^{170}$ in the server based solution to achieve $50\\%$ probability. The actual problem is even worse because each trial is way more than $O(1)$ and I/O is slow.

## Quantum threats to SHA security

Listening to Gemini there exists one quantum attack called BHT (Brassard-Høyer-Tapp) which exploits the Grover algorithm and reduces the security to $O(\sqrt[3]{N})$. Now $256 / 3 \approx 85,3$ which means we'll probably need to jump to something like SHA512 at some point $(512 / 3 \approx 170,67)$. Since $170,67$ is quite far from being attackable, SHA384 could be the best next choice after SHA256 $(384 / 3 = 128)$. Also, BHT could literally be unfeasible for long time for certain reasons, in fact, usually everyone cites Grover only, which will be the actual next problem (as soon as QCs start really working) for hash functions. From what I'm perceiving, Grover splits $N$ in half, and so SHA256 will be hopefully secure for looong time.

## STARKs scenario

Say we find a way to get some hash collision pwning Poseidon, how would we exploit it in a Merkle commitment scenario (like the STARK one)?

STARKs require many Merkle proofs. **For each tree we should generate one single collision (it's not possible, but still, let's say it is) on the root to generate fake proofs**. Let's understand how we could try to use them.

Say we have `Malicious_Merkle_Trees = MMTs` and some leaves we obtain through random oracle challenges. **We fix honest leaves (we substitute malicious ones) and generate `Fake_Honest_Merkle_Trees = FHMTs`.** Now, in order to make our malicious trees to pass, we will need to use the previous section mechanism (or another one to get the collision; I wrote this one for clarity) but starting from `initial_state` **where each $(2^{35})$ hash on the vector we test are obtained varying some leaves (or better, some nodes, if we can) in `FHMT` and `MMT` and extracting the roots. Since we'll need to get a collision between the roots of `FHMT` and `MMT` we will lose another half probability of getting an useful collision**, thus obtaining

```math
\text{Pr}(\text{collision}) = 1 - e^{- ((2^{34})^{2} / 2^{257})} = 1 - e^{- (2^{68} / 2^{257})} = 1 - e^{- \frac{1}{2^{189}}}
```

in the first trial. Now, say we can break Poseidon. We obtain some **useful** collisions and after that substitute `MMTs` with `FHMTs`. Since we forged our trees after random challenges, we could try to pass a proof where our Merkle proofs and commitments mirror honest ones, **but we send malicious statediffs in order to make everyone update their states, then control the root and think _"ok the proof is safe"_ (while it isn't)**. This, in particular, works only if only the Merkle roots are controlled. It could be easy to further improve security by controlling some nodes too (and it wouldn't even generate computational overhead).

Gemini replied that the previous scenario could be prevented nonetheless if the verifier inserts `statediffs` data directly from blobs into AIR constraints. This is true only if our malicious `statediffs` are completely unrelated to our `FHMTs`. If, instead, we manipulate statediffs to be fake honest in the places corresponding to the random challenges (that is, the trace lines controlled by the verifier) then we should be able to pass a malicious Merkle tree.

**Iff what I just wrote is correct, then the security of STARKs is (also) restricted to the security of hash collisions** and finally should explain why Starkware wrote **BLAKE2** with 160 bits digests allows 80 bits of security (birthday paradox, +, more likely, Grover algorithm) in ethSTARK.
