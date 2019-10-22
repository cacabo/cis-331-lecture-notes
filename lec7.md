# Lecture 7 (Tuesday September 17, 2019)

## On the board

Last lecture: Passwords
Announcements:

- Lab 1 due on Thursday

Agenda for Today

1. Recap of passwords:

   - User authentication
   - How to store passwords
   - How to transmit passwords

2. Finish up passwords:

   - Password recovery
   - Alternatives

3. Intro to cryptography

   - Goals and applications
   - Historical ciphers
     - Shift cipher
     - Affine cipher
     - Substitution Cipher
     - Vigenere Cipher
     - Enigma

---

2. [ See lec6.txt, starting at Password recovery ]

3. Intro to cryptography

What is cryptography?

The study of how codes and ciphers can be used to protect the
confidentiality of secret data

Used in antiquity to protect the confidentiality of personal
communication and in the military to protect secrets such as when or
who to attack.

[Story]

Used Today to protect the confidentiality and integrity of everything
we do online, from credit card transactions to the movement of characters
in a video game.

But modern cryptography can do a lot more! Some examples:

- Digital signatures
  Goal: must prevent copy-paste of signatures

- Anonymous communication
  Goal: must prevent ISPs/Proxies from learning who is talking to whom.

- Digital cash / Anonymous digital cash
  Goal: must prevent double spending

- Homomorphic encryption
  Goal: Compute Enc(f(x, y)) from Enc(x) and Enc(y) without learning
  x or y. For example if f(x, y) = x + y, then we want to be able to
  compute Enc(x + y) from Enc(x) and Enc(y).

- Private auctions
  Goal: Compute the result of an auction without revealing the bids

- Secure multiparty-computation
  Goal: multiple parties have their own secret inputs.
  For example, Alice has input x1, Bob has input x2, Charlie has
  input x3. Multiparty computation allows them to compute some
  function y = f(x1, x2, x3), without revealing their secret input.

- Verifiable outsourcing
  Goal: send an input x and a function f to an untrusted server
  (e.g., the "cloud"), and get back a result y and a proof \pi.
  Client can check the proof to convince itself that y = f(x).
  Crucial: checking the proof is _cheaper_ than computing f(x) itself!

Cryptography is a rigorous science!

- Requires a formal definition of the property/guarantee
- Requires a mathematically precise threat model
- Requires a construction

- Requires a proof showing that if one can violate the guarantee provided
  by the construction under the threat model, then one can solve
  some problem believed to be very hard to solve.

  Example: if you can learn which message was encrypted, you can
  factor a 4096-bit composite number into its prime factors.

---

# Historical ciphers

Setup: Two entities, Alice and Bob

Assumption: They share a secret key (k) and an encryption (E)
and decryption (D) algorithm

We call an unencrypted message the "plaintext" (denoted by "m") and the
encrypted messages the "ciphertext" (denoted by "c")

         Alice: k                         Bob: k

m --> [ E ] --> c = E(k, m) c --> [ D ] -> m = D(k, c)
k --> k -->

a) Shift cipher

- Suppose your alphabet has N elements.

- Come up with a random number between 0 and N-1. This becomes your key!
  So: k \in_R [0, N-1]. Encryption works by shifting each character
  of the plaintext by k positions to the right (mod N). In other
  words: E(k, m) = m + k (mod N).

  Decryption works by shifting each character of the ciphertext by k
  positions to the left (mod N). So D(k, c) = c - k (mod N).

  For example:

  Suppose the alphabet consists only of characters a through z.

  - We are working mod 26

  Suppose the key is: k = 2

  The plaintext is m = "i like the zoo"

  c = E(k, m) = "k nkmg vjg bqq"

  m = D(k, m) = "i like the zoo"


    Shift cipher with a shift of 3 is known as a "Caesar cipher", since Caesar
    is said to have extensively used a shift of 3 to encrypt his communications.

    Shift cipher with a shift of 13 is known as "ROT13", and has the nice
    property that E and D are the same algorithm.

    What's wrong with this cipher? Very few keys. How many are there in this
    example? 26. We can try them all. If you guess wrong key it will decrypt to
    garbage with high probability.

b) Affine cipher (generalization of shift cipher):

    In the shift cipher, encryption was defined as: E(k, m) = m + k (mod N)
    where the key k is an integer in [0, N-1].

    In an affine cipher, the key is a pair of integers k = (a, b).

    Encryption and decryption are as follows:

    E(k = (a, b), m) = a * m + b (mod N)
    D(k = (a, b), c) = a^-1*(c - b) (mod N)

    where a*a^-1 = 1 (mod N)

    For this to work, the integer "a" must be *coprime* to N. Otherwise "a"
    has no inverse mod N.

    a and N are coprime if the only integer factor that divides both
    of them is 1.

    Example 1:

    2 has no inverse mod 4 because 2 and 4 are not coprime.

    2 * 0 = 0  (mod 4)
    2 * 1 = 2  (mod 4)
    2 * 2 = 0  (mod 4)
    2 * 3 = 2  (mod 4)

    Example 2:

    3 has an inverse mod 4 because 3 and 4 *are* coprime.

    3 * 0 = 0     (mod 4)
    3 * 1 = 3     (mod 4)
    3 * 2 = 2     (mod 4)
    3 * 3 = 9 = 1 (mod 4)  // This is the definition of inverse!

    so: 3^-1 (mod 4) = 3, since 3 * 3 = 1 (mod 4).


    What's wrong with this cipher? # of keys is still very small.

    Assuming the alphabet is a through z:

    # of possible values for b = 26 (same as shift cipher).
    # of possible values for a = 12 (there are 12 integers in [0, 25]
      coprime with 26)

    Total number of keys: 26 * 12 = 312. Very few keys. Can try all of them
    again.

    Note that when a = 1, the affine cipher is the same as a shift cipher.

c) Substitution cipher:

- Come up with a random mapping between letters in your alphabet.
  This becomes your key k.

  For example:

  Assume an alphabet consisting of letters a through z.

  k = [
  a -> d
  b -> x
  c -> w
  d -> y
  ...
  y -> b
  z -> e
  ]


    c = E(k, "bad day") = "xdy ydb"

    m = D(k, "xdy ydb") = "bad day"


    Pretty simple! So, what's wrong with it?

    How many keys are possible? Total number of ways to permute 26 letters,
    or 26! ~ 2^88

    2^88 keys is pretty hard to brute force (trying to guess the key).
    So that's not the problem.


    ISSUE: Every letter in the plaintext maps to the same exact letter
    in the ciphertext. We can use *frequency analysis* to guess the shift.


    Question: What is the most common letter in English text? "E"


    (1) Figure out the frequency of letters

        "e": appears 12.7% of the time
        "t": appears 9.1% of the time
        "a": appears 8.1% of the time

        We can then guess that the most common letter in the ciphertext
        is corresponds to "e" in the plaintext.

        The second most common letter in the ciphertext maps to "t" in
        the plaintext.

        The third most common letter in the ciphertext maps to "a" in
        the plaintext.

        The rest are tricky. Also this doesn't work very well for short
        messages.


    (2) Use frequency of pairs of letters (diagrams)

        "he", "an", "in", "th"

        Most common pairs of letters in ciphertext is likely to be one of the
        above diagrams in the plaintext.

    (3) Use frequency of trigrams, quadgrams, etc.


    Attack gets easier for longer ciphertexts.

    Consequently, substitution cipher is weak against the worst
    possible attack, a *ciphertext-only attack*: an attack in which
    all the adversary needs is the ciphertext.

d) Vigenere cipher [16th century]

The key is a secret word consisting of letters from an alphabet
with N symbols.

To encrypt, we replicate the key to match the size of the plaintext.
The ith character of the ciphertext is the ith character of the plaintext
added to the ith character of the replicated key mod N.

To decrypt, we replicate the key to match the size of the ciphertext.
The ith character of the plaintext is the ith character of
the ciphertext minus the ith character of the replicated key mod N.

Example:

Suppose our alphabet consists of letters A through Z. Where A = 0,
B = 1, C = 2, ..., Z = 25.

    k = "UPENN"
    m = "CRYPTOISREALLYAWESOME" (crypto is really awesome)

Setup: replicate key until it has 21 characters (size of m)

    k' = "UPENNUPENNUPENNUPENNU"

Encryption: set c[i] = m[i] + k'[i] mod 26

    k'= "UPENNUPENNUPENNUPENNU"
               +
    m = "CRYPTOISREALLYAWESOME"
    ----------------------------
    c = "WGCCGIXWERUAPLNQTWBZY"

Decryption: set m[i] = c[i] - k'[i] mod 26

    c = "WGCCGIXWERUAPLNQTWBZY"
               -
    k'= "UPENNUPENNUPENNUPENNU"
    ----------------------------
    m = "CRYPTOISREALLYAWESOME"

What's wrong with this cipher? Can also be broken by a ciphertext-only
attack!

An effective attack: - Assume you know the length of the key. Say it is 4.

    - Break ciphertext into chunks of 4-characters.

    - Look at first letter in each of the chunks.

      * We know they are all encrypted using the same letter.

      * So they are essentially encrypted by a shift cipher!

      * Can perform frequency analysis (we know most common letter is
        likely to be encryption of e, diagrams, trigrams, etc.)

- What if you don't know the key length? Try for key length 1,
  then 2, then 3, etc. until you get a decryption that makes sense.

e) Enigma / Rotor machines

[ Show pictures ]
