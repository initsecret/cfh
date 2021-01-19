A *cryptographic fuzzy hash* with respect to a distance metric `D: {0,1}^* x
{0,1}^* -> [0,1]` is a tuple `(H, S)` of functions, termed the *fuzzy hash
function* and *similarity function* respectively. satisfying the following
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

**Note.** The above definition is clearly not formal, I used `256` instead of
`n` and use phrases like "are about the same", but it shouldn't be too hard to
formalize it with a bunch of parameters and a couple of epsilons.

### Explicit Examples (or Why I Care)

#### 1. CSAM Detection

Cloudflare talks about this in [their blog post on CSAM
scanning](https://blog.cloudflare.com/the-csam-scanning-tool/amp/) in which they
say the following.

> The challenge was if criminal producers and distributors of CSAM knew exactly
> how such tools worked then they might be able to craft how they alter their
> images in order to beat it.

Looks like a classic case of [security by
obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity) to me. An
explicit construction would help us here.

#### 2. The Right to Be Forgotten

The GDPR includes a [Right to Erasure](https://gdpr-info.eu/art-17-gdpr/) but
that doesn't stop people from re-uploading content you deleted. Suppose you
uploaded some rude pictures of yourself as a public post on Facebook, and then
deleted them. There does not seem to be an easy way for Facebook to prevent
someone else uploading the same images back. Indeed, the threat model seems
similar to the one for CSAM.

#### 3. Fingerprint Authentication

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

A lot of people seem to be using
[PhotoDNA](https://en.wikipedia.org/wiki/PhotoDNA) for CSAM Scanning but
PhotoDNA is not public: [this NYTimes
article](https://www.nytimes.com/interactive/2019/11/09/us/internet-child-sex-abuse.html)
contains the best description of it that I've come across.

### Some Observations

1. If you only care about bitflip errors, then piecewise hashing with a
   cryptographic hash works.
2. Bitflip errors doesn't seem to be a good model. For instance, an image and an
   image shifted by one are kinda the same but they are very far in Hamming
   distance.

This model might be a bit too rigid for the CSAM case, so let's consider
relaxations.

1. If we assume that the client has access to the set they are comparing
   against, then we can use an idea from [traceback paper by Tyagi, Miers, and
   Ristenpart (ia.cr/2019/981)](https://ia.cr/2019/981) and ask the client to
   produce a zero-knowledge proof that they are sufficiently far away from any
   element in this set. In this setting, the zero-knowlege proofs plays the role
   of the hash. However, this has the downside of requiring the client to have
   plaintext access to the target set (which we really don't want in the CSAM
   setting!)
2. Building on the above setting, if we assume that we have a fuzzy hash that is
   not cryptographically hiding but has the trivial similarity function (that
   is, `H(x)` and `H(y)` are similar if and only if `H(x)==H(y)`), then we can
   use the OPRF-based private keyword search scheme of [Freedman et al.
   (2005)](https://www.iacr.org/archive/tcc2005/3378_304/3378_304.pdf) to query
   a database of CSAM indexed by the fuzzy hash. In this setting the database
   can be on a server and if we force all queries to use the private keyword
   search protocol, then we don't leak the plaintext hashes (assuming the hashes
   are sufficiently hard to guess.)

### Failed Constructions

#### 1. Construction from Fuzzy Hash for Numbers

Suppse we had a cryptographic fuzzy hash for numbers, that is a hash `H` which
maps a number to a hash and a similarity function `S` which takes two numbers
and outputs a similarity score. Then we can consider the following naive
construction.

1. Pixelate the image to say 1920x1080 or fix some other common resolution
2. Hash every pixel using the number hash

The similarity score between two images could be the sum of the similarity
scores for each of the pixels.

Unfortunately, this doesn't work. For starters, each pixel is typically
represented by 24 bits (24-bit color; 8 bits each for R, G, and B) or 32 bits
(24-bit color plus 8 bits for alpha) and is easy to bruteforce. In other words,
with approximately `(2^32) + (image size)` work one can recover the original
image from the hash (`2^32` work to build a lookup table and `image size` work
to query each pixel hash.)

The second issue is that the similarity function can be used as a side-channel
to leak information about the preimage. For example, if we had a 256-bit
preimage, we can do binary search by hashing a value and using the similarity
score to inform the branch we should take, and this attack should take
approximately `256 * (image size)` work (`256` is `log(2^256)` and we need do
the binary search for each pixel.)

In conclusion, it seems like even with a fuzzy hash for numbers, it might be
hard to construct this primitive.

**Conclusion 1.** It doesn't seem easy to bootstrap a fuzzy hash for images from
a fuzzy hash for numbers.

**Conclusion 2.** We might need some sort of a secret to prevent brute force
attacks. Like, if we had a secret, then we can FHE to compute `Enc(|a-b|)` from
`Enc(a)` and `Enc(b)` where `Enc(a)` and `Enc(b)` are binding and irreversable
but it doesn't seem trivial to get `|a-b|` from `Enc(|a-b|)`. Also, we don't
need FHE or even multi-linear maps here, just linear maps.

#### 2. Construction from Obufuscation

Building on Conclusion 2 from Failed Construction 1, how about we embed a secret
key into the hash and similarity programs using obfuscation. So, let's try that.

1. Pick a random secret key `sk`.
2. Define hash `x` to be `Enc(x)` under `sk`; in other words, expose an
   obfuscated function which maps `x` to `Enc(x)` under `sk`.
3. Define the similarity between `Enc(x)` and `Enc(y)` to be `D(x,y)`; in other
   words, expose an obfuscated function which takes `Enc(x)` and `Enc(y)`
   encrypted under key `sk`, decrypts them using `sk`, and then returns the
   similarity between the plaintexts, that is, `D(x,y)`.

This construction seems to work [citation required] but it requires iO!

**Conclusion 1.** iO rocks!

#### 3. Construction from Functional Encryption

Duh! But, again, one needs functional encryption. There seems to be a way to do
functional encryption efficiently using SGX due [Fisch, Vinayagamurthy, Boneh,
and Gorbunov (ia.cr/2016/1071)](https://ia.cr/2016/1071) but it requires SGX and
I don't like or trust SGX (maybe I should, I dunno.)

### Moving Forward

Here are some research questions.

1. What is a good distance metric? As we observed above, Hamming distance
   doesn't seem to be a good candidate. Edit distance seem to get over the issue
   mentioned in the above section but it seems expensive to compute.
   Furthermore, there might be better domain-specific hashes (like one for text,
   PNGs, JPEGs, etc.) On a different note, the [NYTimes article linked
   above](https://www.nytimes.com/interactive/2019/11/09/us/internet-child-sex-abuse.html)
   suggests that PhotoDNA uses "visual features to generate the image’s
   fingerprint." perhaps we can do something similar but this metric might make
   things harder to prove.
2. Can we relax the threat model for the explicit use cases? (See, for instance,
   the discussion in the above section.)
3. What are the performance and storage considerations? If we wanted to, for
   example, scan images server-side then we could take a couple of seconds per
   image, but if we wanted to scan real-time video client-side then we need to
   be faster. Similarly, if we are storing the database server-side then we can
   take a few terabytes for the corpus, but we are storing it client-side we
   probably can't use more than a few megabytes.
4. And, obviously, how the fuck do we construct one of these?!

### Get in Touch

If you make progress on or are just interested in any of these problems, I would
love to hear about it. You can find my contact details on [my
homepage](https://snkth.com).

### License

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img
alt="Creative Commons License" style="border-width:0"
src="assets/by.svg"/></a><br />This work is
licensed under a <a rel="license"
href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution
4.0 International License</a>.
