# Lecture 11 (Tuesday October 1, 2019)

## On the board

Last lecture: Semantic security and intro to hash functions
Announcements:

- HW 2 due Tonight

Agenda for Today

1. Recap of last class

   - Semantic security
   - Hash function definitions

2. Random oracles

3. Applications of hash functions

4. Block ciphers and pseudorandom permutations

5. Collision-resistant hash functions from block cipher
   - Merkle-Damgard construction
   - Compression functions
   - Case study: SHA256

---

1. See [lec10.txt]

2. Random oracles

A random oracle is a function H: {0, 1}^_ -> {0, 1}^n
that outputs a uniformly random value in {0,1}^n for each input in {0,1}^_.

You can think of it as a giant table. It is initially empty.
When you call H(0101), it first checks the table to see if there is an
entry for "0101".

If not, it chooses a value x from {0,1}^n uniformly at random, and
adds the mapping "0101"->x to the table.

If yes, it returns the corresponding value.

A random oracle is in some sense an idealized version of a hash function.

Constructing a random oracle requires essentially materializing the table,
which could have trillions of entries in it.

You already know a thing or two about creating giant tables from HW2.
It is not practical...

Why can't you shrink this table with some clever mechanism like a rainbow
table? Think about it.

Consequently, whenever we need a random oracle we "heuristically
approximate it" with a collision-resistant hash function.

---

3. Applications of (cryptographic) hash functions

- Upload a file to the cloud keep a hash. Download a file later and check.

- Digital signatures (we will discuss the schemes later).

  - Instead of hashing 100 MB file, I hash the file and sign the digest!

- Proving that an integer x is > some value. For example, your age > 21

  (a) Assume hash function H: {0,1}^\* -> {0,1}^256
  (b) Compute c = H^x(1 || seed), where seed \in {0,1}^256 and x is your age
  (c) Assume a trusted third party (e.g., Social security) signs c.
  (d) Bartender asks for your age. You show them:
    
   p = H^{x-21}(1||seed)

      Bartender can compute: H^21(p), and compare with c. If they are equal
      you are at least 21.

      Bartender doesn't learn your age, only that you are > 21.

      You cannot lie about your age (that would require identifying a
      preimage!).

  - Pedantic point: you actually need a random oracle instead of H,
    but ignore that for now.

---

4. Block ciphers and PRPs

- Block ciphers (quick description)

  We have seen stream ciphers before. They work by XORing output of
  PRG with message.

  Block ciphers operate in blocks.


         n bits                                n bits
    [ plaintext blocks ]  -> E , D  ->   [ciphertext blocks]

                            key (k bits)


    AES (most widely used cipher) is a block cipher with parameters:
    n = 128, and k = 128, 192, or 256 bits


    In a block cipher the key is expanded into a sequence of
    keys called round keys.


    key --- key expansion ---> [key 1, key 2, key n]


    The block cipher then computes a round function R(k_i, m) for each
    key i, iteratively, where the output of round function i becomes
    the input m of round function i+1.

    For example, suppose we have 4 rounds:

      c = Enc(k, m) = R(key 4, R(key 3, R(key 2, R(key 1, m))))


    For comparison, AES-128 uses 10 round functions.

    We won't go into the details of the round functions (the reading
    covers this in painful detail). Instead, we will abstract the
    block cipher using pseudorandom permutations.

---

- Pseudorandom functions and pseudorandom permutations

  A good abstraction for a block cipher is what is known as
  pseudorandom permutation. But let's talk about pseudorandom
  functions first.


    A pseudorandom function (PRF) defined over (K, A, B)

      F: K x A -> B

    such that there exists efficient algorithm to evaluate F(k, a)


    Secure PRF:

      Funs[A, B] = set of all fns from A to B (this is HUGE |B|^|A|).

      S_f = set of all fns from A to B defined by the PRF
          = { F(k, .) such that k\in K } \in Funs[A,B]  (size is |K|)


      A PRF is secure if:

        a random function in Funs[A, B] is computationally indistinguishable
        from a random function in S_f.


        In a security game it would look like:

        1. Challenger flips a coin b.
            If b = 0, challenger uses f \in Funs[A, B]
           else challenger uses F(k, .) \in S_f

        2. Adversary sends a bunch of inputs a_1, a_2, a_3, ..., to chal.

        3. Challenger computes either f(a_1), f(a_2), ...
           or F(k, a_1), F(k, a_2), ... depending on coin flip, and sends
           result to adversary.

        4. Adversary guesses challenger's coin.

---

    A pseudorandom permutation (PRP) defined over (K, A)

      E: K x A -> A

    such that:

      - there exists efficient deterministic algorithm to evaluate E(k, a)

      - the function E(k, .) is one-to-one (this means it's also invertible)

      - there exists efficient inversion algorithm D(k, y)


    Secure PRP. Similar to Secure PRF, but we use the following:

    Perms[A] = the set of all permutations from A to A (size |A|!)

    S_p = { P(k, .) such that k\in K } \in Perms[A]  (size is |K|)

    The game is the same. Adversary has negligible advantage in distinguishing
    between a PRP and a random permutation in Perms.

---

5. Building a collision resistance hash function (CRHF) from a block cipher

- We will do this in 2 steps:

  (a) Given a CRHF for small fixed-sized messages, we will build a
  CRHF for large variable-sized messages

  (b) Build a CRHF for small fixed-sized messages

(a) Merkle-Damgard construction

    Given a CRHF h for small fixed-sized messages (usually called a compression
    function), we build a CRHF H for large variable-sized message as follows.


    Step 1. Break message into small chunks that can be processed by h.

    Step 2. Add padding as needed, and append the size of the pre-padded message.

    Step 3. Set the initialization vector (IV). Usually hard-coded into
            the standard and the function (e.g., in SHA256)


    Step 4. H(m) = h(h(h(IV, m_0), m_1), m_2)

           For ease of reference, call:
            H_0 = IV
            H_1 = h(H_0, m_0)
            H_2 = h(H_1, m_1)
            ...
            H_i = h(H_{i-1}, m_{i-1})


    [ Draw figure ]

Theorem: If h is collision resistant then so is H

Proof (for you to do). Hint: Show that collision on H => collision on h.

(b) Building a compression function from a block cipher

    Recall block cipher (E, D), where E: K x {0,1}^n -> {0,1}^n

    We can use this to build a compression function as follows.


    Davies-Meyer construction:

      h(H_i, m_i) = E(m_i, H_i) XOR H_i

      Note that it is using the message (known to the attacker) as a key!
      That's okay, we are using this for 2 properties:

      (1) Compression. It is taking k + n bits of data (m_i and H_i) and
      compressing them into n-bits.

      (2) Collision-resistance.

      Theorem. If E is an ideal cipher (collection of |K| random permutations)
      then h is collision resistance.


    There are several secure variants of this construction:

    Miyaguchi-Praneel: h(H_i,  m_i) = E(m_i, H_i) XOR H_i XOR m_i

---

Building a _provable_ compression function

Choose a random 2000-bit prime p, and a random 1 <= u, v <= p

for m,H \in {0,p-1}, h(H, m) = u^H \* v^m (mod p)

Fact: finding collision for h(., .) is as hard as solving a discrete
logarithm modulo p (believed to be "hard").

Problem: really slow. Modular exponentiation is much slower than
block ciphers. So nobody uses this.

---

Case study: SHA 256

    - Merkle-Damgard function
    - Davies-Meyer compression function
    - SHACAL-2 Block cipher


     512-bit key    --->
                            SHACAL-2   -------> 256-bit block
     256-bit block  --->


    Remember that in Davies-Meyer, the "key" is really the message.
    This means that SHA256 processes blocks of the message of size 512-bit
    at a time.
