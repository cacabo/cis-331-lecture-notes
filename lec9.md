# Lecture 9 (Tuesday September 24, 2019)

## On the board

Last lecture: Review of probability + Perfect secrecy
Announcements:

- HW 2 due next Tuesday

Agenda for Today

1. Recap perfect secrecy definition

2. One-time pad

   - Recap
   - Proof that it guarantees perfect secrecy
   - Does not guarantee integrity

3. Stream ciphers

   - PRG (informal)
   - Stream cipher construction
   - PRG (formal)
   - Negligible function

4. Computational security
   - PRG definition: indistinguishability from random
   - Computational adversary

---

1. See [lec8.txt]

2. See [lec8.txt]

Major issue:
  
 Perfect secrecy implies |K| >= |M| ... Why?
  
 Suppose we had fewer keys. For example K = {0, 1}^n-1.
Suppose E(k, m) = (1 || k) XOR m.

      Then, Pr[E(k, m) = c] will no longer be a constant! Because there is
      some probability that for some m and c, there is no key that can
      encrypt m into c.

      For example. Suppose we are given:
        m = 1001
        c = 1101

        and we are using the above encryption algorithm

        Observe there does not exist a k such that
        E(k, 1001) = (1 || k) XOR 1001 = 1101.

    As a result, we require that key length >= message length

    This makes OTP hard to use in practice... We need keys as big as the
    message. And we need a new key for every message...

Note: OTP does not guarantee integrity (in fact ciphertexts are _malleable_)

    Suppose Alice generates:

    m = "Hi, let's meet a 3"
    c = Enc(k, m)

    and sends c to Bob.

    Suppose adversary intercepts message before Bob receives it.
    Suppose adversary knows the meeting time goes at the very end (adversary
    doesn't know the actual value, since OTP guarantees perfect secrecy).

    However, adversary can produce:

    c' = c XOR 0000000000....0001

    Now, c' encrypts message: "Hi, let's meet at X", where X is not 3.

    So now adversary forwards c to Bob.

    Bob decrypts and misses his meeting with Alice!

    (You can imagine worst attacks with banking transactions, etc.)

---

3. Welcome to the 21st century: Stream ciphers

Bad news with perfect secrecy: Any cipher that has perfect secrecy
must have really long keys (i.e., key length >= message length).

Proposed solution: let's take the idea of OTP, and make it practical
by introducing some assumptions that we believe hold in practice.

Key idea: Replace the "random" key in OTP by a "pseudorandom" key.

PRG: Pseudorandom generator

A PRG G is a function that takes a random "seed" (a value that
initializes the PRG) and generates "pseudorandom" bits (more than the
number of bits in the seed).

G: {0, 1}^s -> {0, 1}^n n >> s  
 -------
Seed space

For example, we take a seed that's 256-bits, and expand it into billions of
bits.

Key properties of PRG:

    (1) Given a small random seed, one can compute output of
    PRG efficiently with a deterministic algorithm (the seed is random
    but the algorithm is deterministic).

    (2) the output of the PRG "looks" random (we will revisit this)

---

Stream cipher: Making OTP practical

    Suppose that we are given a PRG G with the above two properties.


    [Draw picture of key expansion]


    We then build the following cipher (E, D):

    E(k, m):
      k' = G(k)     // the "small" key becomes the seed!
      c = m XOR k'


    D(k, m):
      k' = G(k)
      m = c XOR k'


    Are stream ciphers perfectly secure?

    So what security guarantee do they give? Depends on G.

    So let's talk about PRGs in more detail!

---

PRG requires "unpredictability"

What does unpredictable even mean? Adversary cannot predict next bit.

Informally: A PRG G is predictable if there exists an integer i and
efficient algorithm A such that:

    G(k)_{1,...,i}   -- A -->  G(k)_{i+1, ...,n}

If G is predictable, then our stream cipher is not secure.

Proof (by contradiction).

Suppose PRG G is predictable and stream cipher _is_ secure.

Suppose a setting where the attacker knows a small part of the message
(e.g., many Internet/Web protocols have a "common" headers).

Then the attacker can use the knowledge of that small part of the message
and the ciphertext c to derive the output of G for that small part of the
message.

Since G is predictable, attacker can determine the next bits, and recover
the rest of the message. As a result, the stream cipher is not secure.

With mathematical notation:

Adversary knows m\_{1, ..., i} and c.
Adversary computes:

    G(k)_{1, ..., i} = m_{1, ..., i} XOR c_{1, ..., i}

Since G is predictable, adversary computes:

    G(k)_{i+1,...,n} = A(G(k)_{1, ..., i})

    m_{i+1, ..., n) = c_{i+1, ..., n} XOR G(k)_{i+1, ..., n}

    m = m_{1, ..., i} || m_{i+1, ..., n}

So adversary can recover the entire plaintext!

** Even if adversary can only recover next bit, this is still a problem **

How do we formalize unpredictability?

Let's start with a formal definition of predictability (easier to state).

Definition: A PRG G: K -> {0, 1}^n is predictable if:

there exists "efficient" algorithm A and 1 <= i <= n-1 such that
given k <-R K:

    Pr[A(G(k)_{1, ..., i}) = G(k)_{i+1}] >= 1/2 + epsilon

    for some "non-negligible" epsilon

(we discuss what "non-negligible" is later,
but epsilon >= 1/2^30 would be considered non-negligible)

Definition: A PRG G is unpredictable if:

For all i, no "efficient" algorithm can predict bit i+1
for non-negligible epsilon. In other words:

For all i and A, given k <-R K:

Pr[A(G(k)_{1,...,i}) = G(k)] <= 1/2 + negligible

Question:

Suppose G: K -> {0, 1}^n such that for all k: XOR(G(k)) = 1
In other words, it has the property that the XOR of all of its bits is 1.

Is this PRG predictable? Yup.

Suppose we have G(k)\_{1,...,n-1}, then we can predict G(k)\_n.

What about libc's random number generator?

libc random():
r[i] = r[i-3] + r[i-31] % 2^32
output r[i] >> 1

Is it predictable? Yup.

After a few dozen samples from above PRG, one can predict
the next bit with high probability. So... not a good cryptographic PRG.

---

Negligible and non-negligible

In practice (not rigorous and problematic): epsilon is a scalar

    Example of non-neglibile epsilon: 1/2^30  (likely to happen
                                               when encrypting 1 GB of data)

    Example of negligible epislon: 1/2^80     (won't happen over life of key)

In theory (rigorous definition):

    epsilon is a *function* of a security parameter (e.g., length of key)

      epsilon: Z^+ -> R^+  (from positive integers to positive reals)


    A function f(x) is non-negligible if there exists constant d such
    that f(x) >= 1/x^d for many x

      In other words: f(x) >= 1/poly(x)  (for many x)


    A function f is negligible if for all d: f(x) <= 1/x^d

      In other words: f(x) <= 1/poly(x)   (for large x)


    Examples:

      f(x) = 1/2^x : negligible (when x is large, it is smaller than 1/x^d for all d)

      f(x) = 1/x^1000 : non-negligible (it is bigger than 1/x^1001)


             | 1/2^x for odd x
      f(x) = |                         : non-negligible!
             | 1/x^1000 for even x

---

Problem: the definition of "unpredictable" is not easy to work with.

Want: A better definition for PRG that is easier to reason about.

Suppose we have a PRG G: K -> {0, 1}^n

Alternate definition: Indistinguishability from random

Def: G is indistinguishable from random if

          [ k <-R K,  output G(k)]

          is indistinguishable from

          [ r <-R {0,1}^n, output r]

In other words, given G(k) or r, an attacker cannot guess which one
it is with probability better than 1/2 + negl

[
We will discuss how to formalize this, and what can
be achieved next class
]
