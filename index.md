A *cryptographic fuzzy hash* with respect to a distance metric `D: {0,1}^* x
{0,1}^* -> [0,1]` is a tuple `(H, S)` of functions satisfying the following
properties.

1. `H: {0,1}^* -> {0,1}^256`.
2. `S: {0,1}^256 x {0,1}^256 -> [0,1]`.
3. `H` is *preimage resistant*; that is, for a randomly chosen string `y`, it
   is hard to compute a string `x` such that `H(x)=y`.
4. `(H, S)` is *fuzzy* with respect to `D`; that is, for randomly chosen `x1`
   and `x2`, it holds that `D(x1, x2)` and `S(H(x1), H(x2))` are about the
   same.
5. `(H, S)` is *tamper resistant* with respect to `D`; that is, for a randomly
   chosen string `x`, there is not efficient modification `M` such that `D(x,
   M(x))` is small but `S(H(x), H(M(x)))` is large.

**Note.** A pedantic person, who for once is not me, can write the above
definition more formally using two epsilons.

### Explicit Examples (or Why I Care)

#### 1. Fingerprint Authentication

One can think of such a scheme being used to hash fingerprints. Currently, to
the best of my knowledge, there is no open standard for fingerprint hashing. If
such a scheme existed, we could use that. This is more generally applicable to
biometric authentication.

**Note.** Fingerprint authentication is easier as we can assume that the noise
or the "fuzziness" is random and not adversarial. If we assume this, then we
can use ideas from error correcting codes---namely, *decoding* error correcting
codes---to construct a fuzzy hash. The key idea is to decode the input (under a
suitable error correcting code) before hashing it with a cryptographic hash.
These ideas, to the best of my knowledge, were first discussed in [Juels and
Wattenberg (1999)](https://dl.acm.org/doi/10.1145/319709.319714).

#### 2. CSAM Detection

Cloudflare talks about this in [their blog post on CSAM
scanning](https://blog.cloudflare.com/the-csam-scanning-tool/amp/) in which they
say the following.

> The challenge was if criminal producers and distributors of CSAM knew exactly
> how such tools worked then they might be able to craft how they alter their
> images in order to beat it.

Looks like a classic case of [security by
obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity) to me. An
explicit construction would help us here.

### Literature Review

None of the articles I could find on the internet (e.g.,
[TLSH](https://github.com/trendmicro/tlsh),
[ssdeep](https://ssdeep-project.github.io/ssdeep/index.html),
[sdhash](https://github.com/sdhash/sdhash), and
[Nilsimsa](https://en.wikipedia.org/wiki/Nilsimsa_Hash)) seem to talking about
this definition or these properties explicitly. In fact, I found an article

> Using Randomization to Attack Similarity Digests; Jonathan Oliver, Scott
> Forman, and Chun Cheng; DOI:
> [10.1007/978-3-662-45670-5_19](https://doi.org/10.1007/978-3-662-45670-5_19)

which shows how most fuzzy hashes fail under the naive random noise attack.

### Simple Constructions for Restricted Cases

1. If you only care about bitflip errors, then piecewise hashing with a
   cryptographic hash works.

### Moving Forward

It would be nice to formally define an attack model and construct a
cryptographic fuzzy hash for it.

