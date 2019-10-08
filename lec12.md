# Lecture 12 (Thursday October 3, 2019)

## On the board

Last lecture: Block ciphers and hash functions
Announcements:

- Project 2 due October 17

Agenda for Today

1. Recap of last class

   - Block ciphers
   - Compression functions
   - Merkle damgard construction

2. Attacks on hash functions

   - Length extension attack
   - Generic attack (birthday attack)

3. Message authentication codes (MAC)

   - Introduction
   - MAC definition
   - HMAC
   - Authenticated encryption
   - Authenticated encryption construction

4. Asymmetric encryption introduction
   - Hard problem: discrete log
   - Diffie-Hellman key exchange

---

Recap

1.1. Block ciphers like AES operate in rounds.
(a) They expand a small key into rounds keys (11 round keys for AES) - Sometimes called the "key schedule".

(b) They process each block of plaintext at a time, and compute
a round function with different keys several times.
The ith execution of the round function uses as input the output of
i-1th execution of the round function and the ith round key.

       AES-128 works as follows (high level):

       Round 0: initial round where the round key is XORed with the input
       message. There is no round function computed here.

       Rounds 1--9: main round function (complex set of operations).

       Round 10: final round (similar to main round but without
       some operations)


      IMPORTANT: processing each block of the plaintext independently is
      *not* semantically secure (SS). This mode of encryption is called
      Electronic Codebook (ECB) mode. Why is this not SS?


      So what do we do? Several other modes that address this.

      Most common example: Cipher Block Chaining (CBC) mode.

      Idea: XOR the *ciphertext* of the previous block with the current
      plaintext before encrypting the plaintext.


           Plaintext 1                  Plaintext 2
               |                             |
      IV --->  XOR            |-----------> XOR
               |              |              |
      Key --> Block Cipher    |    Key--> Block cipher
               |_______________              |
               |                             |
           Ciphertext  1                Ciphertext 2

1.2. Compression functions

(a) Typically constructed from block cipher E.

(b) Davies-Meyer:
Given message m and initialization vector or previous hash H

      h(H, m) = E(m, H) xor H    // here "m" is used as the key in E


      What happens if we don't add the xor at the end?

      Recall that a block cipher is a PRP. And a PRP is one-to-one so it
      has an inverse. Furthermore, this inverse is easy to compute if you
      know the key. Which in this case the adversary does! It is m (the
      input to our compression function). So you can find a collision. How?

      Choose a random (H, m, m').
      Construct an H' such that: h(H, m) = h(H', m').
      How to construct H'? Hint: you have the key (m') so you can
      use the block cipher's decryption function.

1.3. Merkle-Damgard

(a) Given a compression function h, can build a hash function H
by iterating over h.

    Let m = m1 || m2 || m3     // assume each block is n bits
    let pb = padding + length

    H(m) = h(h(h(h(IV, m1), m2), m3), pb)

---

2. Attacks

   2.1. Generic attack on _any_ CRHF (Birthday paradox)

   Let H: M -> {0,1}^n be a hash function (|M| >> 2^n)

   Goal: have algorithm that finds collision in O(sqrt{2^n}) hashes


    Algorithm:

      1. Choose sqrt(2^n) random messages in M: m_1, ..., m_sqrt{2^n}

      2. For i = 1, ..., sqrt{2^n},  compute  t_i = H(m_i)

      3. Look for a collision (t_i = t_j) by comparing all pairs of hashes.
         If none found, go back to step 1.


      How well does this simple algorithm work? Surprisingly well...
      With very high probability we only need to run this algorithm a
      couple of times to get a collision.


      How is that possible? Birthday paradox.

      Wikipedia has a nice informal description (adapted below):

      Consider a scenario in which a teacher with a class of
      30 students (n = 30) asks for everybody's birthday (for
      simplicity, ignore leap years) to determine whether any
      two students have the same birthday (corresponding to a hash
      collision). Intuitively, this chance may seem small.

      If the teacher picked a specific day (say, October 3rd), then
      the chance that at least one student was born on that specific day
      is 1 − (364/365)^30, about 7.9%.

      However, counter-intuitively, the probability that at least
      one student has the same birthday as any other student on any day is
      around 70% (for n = 30), from the formula
      1 - (365! / ((365 - n)! 365^n))

      More formally: Let r1, ..., rn \in {1, ...,N} be independent
      and identically distributed (i.i.d) integers.

      Thm: when n = 1.2 * sqrt{N}, then
      Pr[there exists ri = rj, such that i != j] >= 1/2

      Proof: Pr[there exists ri = rj s.t. i!=j]
            = 1 - Pr[for all i != j, ri != rj]
            = 1 - Product from i to n-1 of (1 - i/N)
            >= 1 - e^{n^2/2N}
            = 1 -e^-0.72
            = 0.53
            > 1/2

2.2. Length extension attack on Merkle-Damgard-based constructions

    Recall construction:

    Let m = m1 || m2 || m3     // assume each block is n bits
    let pb = padding + length

    H(m) = h(h(h(h(IV, m1), m2), m3), pb)


    Note that if an attacker has H(m), it can construct H(m || other stuff)
    as follows.


    H(m || other stuff) = h(h(H(m), m'), new padding and length)

    Here, "other stuff" = pb || m'.

    While this might not seem like a big deal, you'll see in Project 2
    how this can be exploited in settings where one tries to use
    a hash function as a MAC (explained next!).

---

3. MAC

   3.1. Recall that collision-resistant hashing was useful in situations
   in which you need to upload a file to the cloud and keep a digest.
   Then you can later download the file and compare the digest.

What about in situations where Alice wants to send a message to Bob,
and Alice wants to ensure the message is not modified in the meantime?

secret k message m secret k
Alice ---------------------------> Bob

We saw before that Alice could use a stream cipher or a block cipher to
encrypt m, but that only guarantees confidentiality.

The adversary could still modify the message in transit. Suppose
Alice does not care about confidentiality, she only cares about
integrity. Can we use a CRHF here?

Strawman 1:

    Alice computes tag = H(m)

    Alice sends (m, tag) to Bob

    Broken! Why?

Strawman 2:
  
 Alice computes tag = H(k || m)
Alice sends (m, tag) to Bob

    Broken! Why?

3.2. We need a Message Authentication Code!

secret k message m, tag secret k
Alice ---------------------------> Bob

Generate tag: Verify tag:
tag <- S(k, m) V(k, m, tag) ?= "yes"

A MAC is a pair of signing and verification algorithms (S, V)
defined over (K, M, T)

S(k, m) outputs t in T
V(k, m, t) outputs "yes" or "no"

Requirement: For all k \in K, for all m \in M:

    V(k, m, S(k, m)) = "yes"

Secure MACs:

Attacker's power: chosen message attack
  
 For m_1, m_2, ..., m_q attacker is given t_i <- S(k, m_i)

    // This models a situation in which the attacker has seen many
    // messages in the past and knows their corresponding tags

Attacker's goal: existential forgery
  
 Produce any _new_ valid message/tag pair (m, t) without knowing k

    So:
       (m, t) not in {(m_1, t_1), ..., (m_q, t_q)}

       AND

       V(k, m, t) = "yes"

Formal security game for a MAC:

Challenger Adversary

k <- K
m1, m2, m3, ..., mq
<-----------------
ti = S(k, mi)  
 t1, t2, t3, ..., tq
------------------>
(m, t)
<-----------------
output b

b = 1 if V(k,m,t) = "yes" and (m, t) not in {(m1, t1), ..., (mq, tq)}

b = 0 otherwise

Definition: A MAC F is secure if for all efficient A

    Adv_MAC[A, F] = Pr[Chal outputs 1] = negligible

---

3.3. HMAC

Building a secure MAC out of a hash function

    S(k, m) = H(k XOR opad, H(k XOR ipad || m))

ipad and opad are constants that never change.

In pictures (assumes Merkle-Damgard CRHF):

    [k xor ipad]  [   m0   ]   [   m1    ]  [     m2  || PB   ]

IV -> h --> h ---> h ----> h ------
|
|
|
|
[ k xor opad ] ---> \/
h ---> h ----> tag
IV --->

---

3.4. Authenticated encryption

The encryption schemes we've seen so far only provide confidentiality
=> They provide semantic security against chosen plaintext attack
=> Only tolerate an attacker that is eavesdropping

MAC guarantees integrity
=> It guarantees existential unforgeability under a chosen message attack

Can we combine the two to ensure both confidentiality and integrity?

Yes! Authenticated encryption.

An Authenticated encryption system (E, D) is a cipher where

    E: K x M -> C
    D: K x C -> M union {\bot}   // symbol that means ciphertext is rejected

Security: The system must provide.

    - (Old) Semantic security under a CPA
    - (New) Ciphertext integrity: attacker cannot create new ciphertexts
    that decrypt properly.

We've already seen the semantic security game. Let's take a look at
Ciphertext integrity game.

Let (E, D) be a cipher over (K, M, C)

    Challenger                                        Adversary
      k <- K
                         m1, m2, m3, ..., mq
                    <------------------------------

      ci = E(k, mi)
                          c1, ..., cq
                     ------------------------------>

                                   c
                     <-----------------------------

      Output b


    b = 1  if D(k, c) != \bot  AND  c not in {c1, ..., cq}
    b = 0 otherwise

A cipher (E, D) guarantees ciphertext integrity if for all efficient A:

    Adv_CI[A, E] = Pr[Chal outputs 1] = negligible

---

3.5 Combining encryption and MACs to provide authenticated encryption:

Assume we have 2 keys. Encryption key k_E. MAC Key k_M.

A few options.

Option 1 (SSL): MAC then encrypt
  
 S(k_M, m) E(K_E, m||tag)
m ---------> m || tag ----------------> c

Option 2 (IPSec): Encrypt then MAC

          E(k_E, m)          S(k_M, c)
      m  ---------->    c  --------------->   c || tag

Option 3 (SSH): Encrypt and MAC

          E(k_E, m)
      m   -------->   c
                           -------->  c || tag
         S(k_M, m)
      m  ---------> tag

Question: Which one of these is actually secure???

Option 3 is bad because MACs are not designed to provide confidentiality!
There could be a MAC that leaks some of the bits of the plaintext,
violating semantic security!

Does this mean SSH is broken? Not quite... the specific MAC and Encryption
they use do not have this issue, but there exist "Encrypt and MAC" schemes
that do.

Option 1 has been shown to have issues for some pathological constructions
of MAC and Encryption that are independently secure, but not when used
together.

[Aside: if the encryption scheme is
malleable (https://en.wikipedia.org/wiki/Malleability_(cryptography)
it may be possible to modify the message to look valid and have a valid MAC.
This is mostly a theoretical point since the MAC secret should
provide protection]

Option 2 is always correct, regardless of which MAC or Encryption
scheme is used (as long as they are independently secure).
Encrypt-then-MAC is the recommended approach nowadays.

---

4. Key exchange

We have shown how users can encrypt data and MAC data using a shared
secret key.

Q: How do the users get the shared secret key in the first place?

A: Maybe they meet at a coffee shop? But they would need to do this
for every single one of their friends, and also online services
(e.g., Amazon) that they intend to use! Surely there is a better way.

Can use a Trusted Third Party (TTP)

Alice -----> TTP <---------- Bob

Every user only needs to have a shared key with the TTP.

K_A = key of Alice shared with TTP
k_B = key of Bob shared with TTP

How can Alice and Bob generate a shared key then?

Alice asks TTP for a shared key with Bob.

TTP generates a key at random k\_{A,B}, and then sends to Alice:

c1 = E(K*A, "Key to be used between A,B" || k*{A,B})
c2 = E(K*B, "Key to be used between A,B" || k*{A,B})

Alice can now send to Bob the messages:

c2 and c3 = E(K\_{A,B}, "Hi Bob! Let's have lunch")

Bob will first decrypt c2 (which it can). It will then check to make sure
it is coming from Alice. Then, it will use the provided key in c2 (K\_{A,B})
to decrypt c3, and get the message "Hi Bob! ...".

_This is secure only against eavesdroppers_

An active attacker can replay the conversation. For example, I can
store c2 and c3. And then send them to Bob next week. Bob will think
Alice wants to get lunch again, and he will be very angry
after he spends 1 hour waiting for Alice.

Key question: Can we generated shared keys without an online
trusted third party (TTP)?

Answer: yes! More next time.
