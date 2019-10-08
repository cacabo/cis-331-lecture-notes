# Lecture 8 (Thursday September 19, 2019)

## On the board

Last lecture: Historical ciphers
Announcements:

- Lab 1 due on Thursday

Agenda for Today

1. Recap on previous ciphers:

   - Shift cipher / Caesar cipher
   - Affine cipher
   - Substitution cipher

2. Finish up historical ciphers:

   - Vigenere cipher

3. Review of probability and XOR

   - Finite set
   - Probability distribution
   - Random variables
   - Independence

4. Clarification on entropy

5. Modern cryptography
   - XOR
   - Perfect secrecy
   - One-time pad

---

1. See [lec7.txt]

2. See [lec7.txt, starting at Vigenere]

3. Review of discrete probability

Finite sets

    U = {0, 1}^n   // Set of n-bit strings (has 2^n elements)

    For example if n = 2 then

    U = {00, 01, 10, 11}

Probability distribution P over a set U is a function P: U -> [0,1]
such that Sum\_{x \in U} P(x) = 1
  
 If U = {0, 1}^2, P could be defined (for example) as:

      P(00) = 1/2
      P(01) = 0
      P(10) = 1/4
      P(11) = 1/4

    Indeed, it sums to 1, which is good. It also means that if we tried to
    sample from P, we would get the string "00" half the time, and the
    string "01" never.

Common probability distributions:

    - Uniform distribution: for all x \in U, P(x) = 1/|U|

    - Point distribution at point x':

        P(x') = 1, and for all x \in U, x != x', P(x) = 0

        In other words, point x' has all of the probability mass (1),
        and all other points have 0 probability of being sampled.

Outcomes, sample space, and events:

    Whenever you do an experiment like flipping a coin, you get an outcome.
    For example, "heads" is an outcome.


    The sample space is the set of all possible outcomes of an experiment.
    If the experiment is rolling a die, the sample space
    is: U = {1, 2, 3, 4, 5, 6}, since these are the possible outcomes.


    An event is a set of outcomes. For example, the event of rolling an
    even number is the set: {2, 4, 6}.

    An event is a *subset* of the sample space

    A \subset U

    Pr[A] means the probability of event A.

    Pr[A] = sum_x \in A P(x)  \in [0, 1].

    Pr[U] = 1, and Pr[A] <= Pr[U]


    Example:

      Experiment: sample a 4-bit string

      Sample space: U = {0, 1}^4  // |U| = 16 elements

      Event A: Sampling a string with least significant bit 1

      A = { all x \in U such that least significant bit is 1 } \subset U

      Assuming uniform distribution on U, then Pr[A] = |A| / |U| = 8/16 = 1/2

      Why?

      A = { 0001, 0011, 0101, 0111, 1001, 1011, 1101, 1111 }

      The probability of sampling a particular element is 1/|U|.
      The probability of sampling any one of the elements in A is |A| * 1/|U|
      We have that |A| = 8, and |U| = 16, so Pr[A] = 1/2

Union bound:

    Given 2 events, A1 and A2:

    Pr[A1 union A2] <= Pr[A1] + Pr[A2]

    If the events are disjoint (share no elements) the probability of
    the union is the sum of the probabilities.

    If they share some elements, then the probability of the union is
    less than the sum of the probabilities.


    Example:

      U = {0, 1}^4
      A1 = {all x \in U such that least significant bit is 1 }
      A2 = {all x \in U such that most significant bit is 1 }


      Pr[A1] = 1/2
      Pr[A2] = 1/2

      Pr[A1 union A2] = ?


      A1 = { 0001, 0011, 0101, 0111, 1001, 1011, 1101, 1111 }
      A2 = { 1000, 1100, 1010, 1110, 1001, 1101, 1011, 1111 }

      |A1| = 8
      |A2| = 8
      |A1 union A2| = 12   // "1001", "1101", "1011", "1111" are duplicates

      Pr[A union A2] = 12/16 = 3/4 < Pr[A1] + Pr[A2]

Random Variables

    A random variable X is a *function* from the set of possible
    outcomes of an experiment, U, to some set V.

      X: U -> V

    Example:

      Experiment: sample a n-bit string
      Sample space U: {0,1}^n

      Suppose I'm interested in talking/reasoning about the least
      significant bit of the sampled string.

      I can define a Random variable X as:

        X(y) = least significant bit(y)  // where y \in U

        So we have that:

             U         V
        X: {0,1}^n -> {0,1}


      For the uniform distribution on U we have that:

      Pr[X = 0] = 1/2
      Pr[X = 1] = 1/2


      You can read Pr[X = 0] as: "what is the probability that when we sample
      an element from U uniformly, the least significant bit is 0?".

Uniform Random Variable

    We often write X <-R U, to denote a uniform random variable over U

    In some sense this basically means that we are interested in the actual
    sampled string (instead of reasoning about, say, the least significant bit),
    so the random variable is simply the identity function:

      X(y) = y

    So we have that for all y \in U, Pr[X = y] = 1/|U|


    Question:

    Let X be a uniform random variables on U = {0, 1}^2

    Let random variable Y = sum of bits(X)   // Y("11") = 2, Y("10") = 1

    Pr[Y = 2] = ?

    Pr[Y = 2] = Pr[ X = "11"] = 1/|U| = 1/4

Randomized algorithm

    Deterministic algorithm:
      y = f(x) // always produces same y for the same x


    Randomized algorithm:

      y = f(x; r) where r <-R {0, 1}^n


      In this case, f depends on r, which is sample uniformly
      at random every time.


    We can think of y as a random variable!

        y <-R f(x)

Independence

    Events A and B are independent if Pr[A and B] = Pr[A] * Pr[B]

    Random variables X and Y taking values in V are independent if
    for all a,b \in V: Pr[X = a and Y = b] = Pr[X=a] * Pr[Y=b]


    Example:

      U = {0, 1}^2 = {00, 01, 10, 11} and r <-R U

      X = least sig. bit(r)
      Y = most sig. bit (r)


      Pr[X = 0 and Y = 0] = Pr[r = 00] = 1/4 = Pr[X = 0] * Pr[Y = 0]

4. Clarification on Entropy:

Last class we briefly mentioned Shannon entropy and said it was defined as:

S = Sum\_{x \in U} ( Pr[x is chosen] \* lg(1/Pr[x is chosen] )

How to interpret this? Let's think about a coin toss. There are two possible
outcomes: heads or tails. So the set of all possible outcomes is
U = {heads, tails}.

Suppose I have a random variable X for the result.

Suppose we are talking about a _fair_ coin. So this is the uniform
distribution over {heads, tails}

So Pr[X = heads] = Pr[X = tails] = 1/2

Shannon entropy of X:

H(X) = Pr[X = heads] * lg(1/Pr[X = heads]) + Pr[X= tails]*lg(1/Pr[X = tail])
= 1/2 _ lg(2) + 1/2 _ lg(2)
= 1/2 + 1/2
= 1

So we have that this uniform random variable X has 1 bit of entropy!

Now, imagine that the coin is biased. In other words
Pr[ X = heads] != Pr[X = tails].
Let's suppose that Pr[X = heads] = 1/2 + e (for some e in the range (0, 1/2) ).
This means that Pr[X = tails] = 1/2 - e.

Let's now compute the Shannon entropy of X when it is not a uniform
random variable:

H(X) = Pr[X=heads] * lg(1/Pr[X=heads]) + Pr[X=tails]*lg(1/Pr[X=tails])
= (1/2 + e) _ lg(1/(1/2 + e)) + (1/2 - e) _ lg(1/(1/2 - e))
= (1/2 + e) _ -lg(1/2 + e) + (1/2 - e) _ -lg(1/2 - e)
< 1

For example suppose e = 1/4

H(X) = (1/2 + 1/4) _ lg(1/(1/2 + 1/4)) + (1/2 - 1/4) _ lg(1/(1/2 - 1/4))
= 3/4 _ lg(4/3) + 1/4 _ lg(4)
= 0.8113

So, because the output is _less_ random, X has less entropy.

The same idea applies to passwords, secret keys, etc.
If a password is generated uniformly from the set of all passwords
(so all passwords are equally likely), then entropy would be
lg(total # of potential passwords). But if some passwords are more likely
to be chosen than others, then entropy goes down.

5. Modern cryptography


    A cipher is defined over

    K: key space
    M: plaintext space
    C: ciphertext space

    It consists of a pair of algorithms:

    E: K x M -> C
    D: K x C -> M

    Such that, for all m \in M, k \in K: D(k, E(k, m)) = m

    E is often a randomized algorithm.
    D is always deterministic.

- One-time pad

  Recall that XOR is bit-wise addition mod 2

  Example: 1001 XOR 0111 = 1110


    K = {0,1}^n
    M = C = {0,1}^n


    A key is a uniformly random bit string as long as the message

    c = Enc(k, m) = k XOR m
    m = Dec(k, c) = k XOR c


    If you are given m and c, can you recover the key?

    Yes: k = m XOR c

- Is the one-time pad secure then?

  Depends what you mean by secure.

  Let's assume a ciphertext-only attacker. An attacker who only knows c.


    How should we define security?

    (a) attacker cannot recover secret key

      Enc(k, m) = m     would meet this definition...


    (b) attacker cannot recover all of the plaintext

      Enc(k, m_0 || m_1 ) = m_0 || k XOR m_1     would meet this definition...


    (c) Shannon's Information Theoretic Security:

      Ciphertext should reveal no information about m!

      But how do we define this formally???

      A cipher (E, D) over (K, M, C) has *perfect secrecy* if:

      for all m0, m1 \in M  (|m0| = |m1|) and for all c \in C

        Pr[E(k, m0) = c] = Pr[E(k, m1) = c]  where k <-R K


      In other words: Given only a ciphertext an adversary cannot figure
      out anything about the plaintext because all plaintexts are equally
      likely to have generated c.



    Is OTP perfectly secure under a ciphertext-only attacker then? YES!


    Lemma: If X is a random variable on {0,1}^n, and Y is an independent
    uniform random variable on {0,1}^n, then Z = X XOR Y is a uniform random
    variable on {0,1}^n

    (Note that this claim holds even if X is not uniform)

    Proof for n=1:

      Suppose:
        Pr[X = 0] = p0         Pr[Y = 0] = 1/2
        Pr[X = 1] = p1         Pr[Y = 1] = 1/2


      We thus have:

      Pr[Z = 0 ] = Pr[(X = 0 AND Y = 0) OR (X = 1 AND Y = 1)]
                 = Pr[X = 0 AND Y = 0] + Pr[X = 1 AND Y = 1] // disjoint events
                 = Pr[X = 0] * Pr[Y = 0] + Pr[X=1]*Pr[Y=1] // independence
                 = p0 /2 + p1 / 2
                 = 1/2               // since p0 + p1 = 1




    Theorem: OTP is perfectly secure under a ciphertext-only attacker

    Proof:

      We have that: k <-R K

      for all m,c
        Pr[E(k, m) = c]  = "# keys \in K such that E(k, m) = c" / |K|


      Lemma: If for all m, c: "# keys \in K such that E(k, m) = c" is constant,
      then OTP is perfectly secure.

      Proof: Pr[E(k, m1) = c] = constant/|K| = Pr[E(k, m2) = c]



      But is it really the case that "#keys \in K such that E(k, m) = c" is
      constant??

      Let m \in M, and c \in C.

      How many OTP keys map m to c? 1


      So: # keys \in K such that E(k, m) = c = 1


      Which means that:

      for all m,c  Pr[E(k, m) = c]  = 1 / |K|

      So OTP is perfectly secure!

Major issue:
  
 Perfect secrecy implies |K| >= |M| ... Why? Think about it!
which implies that key length >= message length

    This makes OTP hard to use in practice... We need keys as big as the
    message. And we need a new key for every message...
