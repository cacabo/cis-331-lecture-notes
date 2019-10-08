# Lecture 10 (Thursday September 26, 2019)

## On the board

Last lecture: PRGs and Stream ciphers
Announcements:

- HW 2 due next Tuesday
- Lab 2 released later Today

Agenda for Today

1. Recap PRG and negligible function

2. Computational security

   - Indistinguishability from random
   - Statistical test
   - Advantage
   - Real crypto definition of PRG
   - Computational indistinguishability

3. Semantic security

   - Stream cipher is semantically secure

4. Hash functions
   - Introduction

---

1. See [lec9.txt]

- Negligible function. Easier definition.

A negligible function f(n): Z -> R is one that not only
tends to zero as n -> infinity, but does so faster
than the inverse of any polynomial (1/poly(n)).

Examples:

    - 2^-n, 2^-\sqrt(n), n^-log n

Why do we draw the line at polynomial? We'll see later when
we discuss computational adversary. Sneak peak: because
we assume adversary can only use algorithms that run
in poly-time.

2. Computational security

- Indistinguishability from random

Problem: the definition of "unpredictable" is not easy to work with.

Want: A better definition for PRG that is easier to reason about.

Suppose we have a PRG G: K -> {0, 1}^n

Alternate definition: Indistinguishability from random

Def: G is indistinguishable from random if

          [ k <-R K,  output G(k)]

          is indistinguishable from

          [ r <-R {0,1}^n, output r]

Represented with a picture:

    [ Drawing of {0, 1}^n  with G() inside ]

But how do we define "indistinguishable"?

We use a statistical test.

---

- Statistical tests

Def: a statistical test on {0, 1}^n is an algorithm A
such that A(x) outputs 0 or 1.

It outputs 0 when it thinks the input is not random

It outputs 1 when it thinks the input is random

Examples of statistical tests:

    (1) A(x) = 1  iff  |# of zeros - # of 1s| <= 10 * sqrt(n)

      Since we expect #0s to be roughly the same as #1s if
      x is uniformly random


    (2) A(x) = 1  iff  |#00 - n/4| <= 10 * sqrt(n)

      Similar argument as above. We expect the number of consecutive 0s
      to occur with roughly probability 1/4.

    (3) A(x) = 1  iff  max-run-of-0  <= 10 * log(n)

      We expect longest number of consecutive 0s to be log(n) in
      expectation.

      Note this test would say string of all 1s "looks random"...
      these tests are not perfect... they can test whatever they want and
      can be arbitrarily wrong.


    In the old days, people would just have a ton of these statistical
    tests, and if all of them said the string looks random, then we would
    say the generator is OK. This is not a good idea.

So how do we know if a statistical test is "good"?

We use the notion of "advantage"

---

- Advantage

  The advantage captures how good a statistical test is
  at differentiating inputs given to it. In this case, how good it is at
  differentiating the output of a PRG vs a truly random string.


    Suppose we have PRG G: K -> {0, 1}^n and a stat. test A on {0,1}^n

    Define advantage:

      Adv_PRG[A, G] = | Pr[A(G(k)) = 1] - Pr[A(r) = 1] |

      where k<-R K    and   r <-R {0, 1}^n



    If advantage is close to 1:

      statistical test behaves differently when given pseudorandom inputs
      vs random inputs (so it is a good statistical test!)

      It can distinguish output of G from random.


    If advantage is close to 0:

      statistical test behaves the same for both. So it cannot distinguish
      output of G from random.



    Example 1:

        Suppose A(x) = 0 => Adv_PRG[A, G] = 0



    Example 2:

      Suppose we have PRG G: K->{0,1}^n with the following property:

        most-significant-bit(G(k)) = 1  for 2/3 of keys \in K


     Suppose we have a statistical test A(x):

        if msb(x) = 1 output 1 else output 0


      Then

        Adv_PRG[A, G] = | Pr[A(G(k)) = 1]  - Pr[A(r) = 1] | = 1/6

---

Secure PRG: Crypto definition

G: K -> {0,1}^n is a secure PRG if:

For all "efficient" statistical test A, Adv_PRG[A, G] <= negligible

Cool! So are there provably secure PRGs?

    We don't know!!! It's an open question!

    If you can prove it, that would imply P != NP...

    But we have heuristic candidates.

---

Computational indistinguishability

Let P1 and P2 be two distributions over {0,1}^n

Def: We say that P1 and P2 are computational indistinguishable

if for all "efficient" statistical tests A

    | Pr[A(x) = 1] - Pr[A(y) = 1] | <= negligible
       x<-P1            y<-P2

Using this definition, then we can say:

a PRG is secure if {k<-R K: G(k)} is computationally
indistinguishable from uniform({0,1}^n)

But what does "efficient" actually mean???

It means that we are making the assumption that adversary does
not have infinite computational power and therefore it cannot run
arbitrary statistical tests.

Instead, it can only run statistical tests that can finish in
a reasonable amount of time (i.e., that are "efficient").

Typically, this means that these statistical tests must have
polynomial running time: O(n), O(n^2), O(n^54335).

So we do not need to consider any statistical test with superpolynomial
running time: O(2^n), O(2^sqrt(n)), etc.

---

Semantic security

This is the definition that modern cryptosystems try to guarantee
(e.g., stream ciphers)

It is defined in terms of an experiment or a "security game".

The game has two entities: A challenger and an adversary

Setup:

- Challenger and Adversary are given the cipher's description \E:

  Encryption and Decryption algorithms (E, D) and spaces (K, M, C)

- Challenger is also given a secret key k<-R K

The game proceeds as follows:

- Adversary picks 2 messages m_0, m_1 \in M: |m_0| = |m_1| and sends them
  to the challenger (it can pick them however it wants)

- Challenger flips a coin uniformly at random: b = {0, 1}

- Challenger encrypts c = E(k, m_b) and sends it to Adversary

- Adversary looks at c and tries to guess the value of the bit b.
  Call the adversary's guess b'.

We define the advantage of the adversary as:

            Adv_ss[A, \E] = | Pr[W_0] - Pr[W_1] |


      W_0 = event that b = 0 and b' = 1.
      W_1 = event that b = 1 and b' = 1.

In other words, the above tries to capture whether the adversary
behaves differently when given the encryption of m_0 versus the
encryption of m_1.

A cipher \E = (E, D) is semantically secure if for all efficient A

    Adv_ss[A, \E] is negligible

In other words no efficient adversary can distinguish between
an encryption of m_0 and an encryption of m_1.

=> for all m_0, m_1 \in M (that the adversary can generate)
  
 { E(k, m_0) } is computationally indistinguishable from { E(k, m_1) }

This is essentially the same as perfect secrecy... except that it applies
only to efficient adversaries.

---

Stream cipher is semantically secure _if_ PRG is secure

Theorem: Given a G: K -> {0,1}^n is a secure PRG, then
Stream cipher \E derived from G is semantically secure

How do we prove this?

We will assume that the stream cipher is not secure but the PRG is secure.
Then we will show that we can break the PRG, which yields a contradiction.
Therefore, the stream cipher must be secure.

In particular we are going to show that:

    For all S.S. adversaries A, there exists a PRG adversary B such that

        Adv_SS[A, \E] <= 2 * Adv_PRG[B, G]

You can read the above as: if you give us a semantically secure adversary A,
what we'll do is we will build a PRG adversary B that satisfies the
above inequality.

This is useful, because it means that if the advantage of A is
non-negligible, then the advantage of B is also non-negligible.

Proof:

Assume that PRG G is secure and the stream cipher is _not secure_.
That means there exists some efficient adversary A that can win
the semantic security game with non-negligible advantage.

We will use that adversary A to build another adversary B that can
distinguish between a PRG and a random string with non-negligible advantage.

[ PRG Challenger ][ adv b ] [ Adv A ]  
  
 b_prg <- {0, 1}  
 k <-R K

     if b_prg = 0          val
      send val = G(k)
                         ------->
     else
      send val = r


                                               <-------  m1, m2


                              b <- {0,1}
                              c = Enc(m_b, val)
                                                    c
                                              ------------>

                                              <------------
                                                    b'

                              if b = b'
                                return b_prg' = 0
                              else
                                return b_prg' = 1




    So how do we analyze this? The key idea:

    Observe that when b_prg = 1, then c is an encryption using
    a one time pad. So Adv A has 0 advantage in that case (since
    one time pad guarantees perfect secrecy).

    However, when b_prg = 0, then c is an encryption of m_b under a
    stream cipher, so Adv A has non-negligible advantage of guessing
    right (by assumption). This event happens with probability 1/2.

    Adv B can in some sense exploit the non-negligible advantage provided
    by Adv A half of the time. As a result, Adv B must also
    have non-negligible advantage. This means that the PRG is not secure.
    Which is a contradiction.

    Therefore, the stream cipher must be secure if the PRG is secure.

---

Hash functions (introduction)

Hash functions map an arbitrarily large set into a fixed-size set

H: {0,1}^\* -> {0,1}^n

Pigeonhole argument. If \* > n, there will be collisions (i.e.,
multiple inputs map to the same "digest" or hash value)

There are a few properties we care about when talking about
cryptographic hash functions:

(1) Pre-image resistance:
  
 Given a digest y, it is hard for an adversary to find an x such
that H(x) = y. In other words, it is hard to find a pre-image of y.

(2) Second pre-image resistance

Given x1 it is hard for an adversary to figure out an x2 (x1 != x2)
such that H(x1) = H(x2). In other words, it is hard to find a
second (meaning another) pre-image.

(3) Collision resistance

It is hard for an adversary to figure out any two values x1 and x2,
x1 != x2 such that H(x1) = H(x2).

Collision resistance => second-preimage resistance

Proof:
Attacker can just choose arbitrary m1, compute second-preimage
and it has found a collision!
