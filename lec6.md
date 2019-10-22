# Lecture 6 (Thursday September 12, 2019)

## On the board

Last lecture: Privilege separation and isolation
Announcements:

- Lab 1 due on Sept 19

Agenda for Today

1. Recap how UNIX isolates:

   - Processes
   - Process memory
   - Files / Directories

2. How does UNIX isolate:

   - File descriptors
   - Local IPC (UNIX Sockets)
   - Networking

3. Finish up UNIX discussion:

   - How are UIDs and GIDs set? setuid
   - Another isolation mechanism: chroot

4. Passwords
   - The problem: user authentication
   - Passwords
   - Alternatives

---

2. [ See lec5.txt, starting at File descriptors (FDs) ]

3) - setuid [ See lec5.txt, starting at "How is a process's UID set ]

   - chroot [ See lec5.txt, starting at "Unix isolation trick: chroot" ]

4. Passwords

The real problem is _how to authenticate users_. Passwords are one solution

Basis of many security policies

Some interesting technical issues

Easy to do wrong on technical grounds

Also remains challenging on non-technical grounds, because
security isn't just a technical problem

Authentication: who is the user?
Challenging to know for sure
User registers some secret --- but _who_ registers it?
At the scale of a university, we can probably check the
identity of the user when registering

Typically settle for weaker guarantee
Establish that the user who logs has the secret used when registering
If so, then assume it is the same user
But, we have no guarantee that we know the true identity of the user
For many usages that is fine
E.g., Amazon doesn't really care who you really are as long as you pay

Problem: how to authenticate users?

```
Setting: [ user ] <-> [ computer ] <-> [ verifier server ]
```

**Passwords**

Need some secret between user and verifier call this set of bits a "password"

User types in username and password; server checks whether
password is correct for that username.

Passwords is a valuable secret so want to avoid repetitive use and exposure
Just for user authentication

- Once authenticated, use crypto keys between server/clients,
  client certificates, cookies, etc.

- Even for user authentication, good idea to compose passwords with
  other techniques:

  - Password manager, single-sign on, two-factor, etc.
  - Biometric (e.g., apple's fingerprint button)

---

How to _store_ passwords?

Server must be able to verify passwords.

Strawman: store plaintext passwords.

Problem: if adversary compromises server, gets full list of passwords.
Really bad since server compromises happen all of the time!

Hashing: store a table of `(username, H(password))`

- Can still check a password: hash the supplied string,
  compare with table, but now if adversary gets the table,
  doesn't get the passwords (because hash function is assumed
  to be hard to invert)
- Problem 1: password space is quite small.

  Top 5000 password values account for 20% of users.

  Skewed distribution towards common passwords chosen by many users.
  Yahoo password study: rule-of-thumb passwords are 10-20 bits of entropy.

  - This roughly means that if password is equivalent to 10 random bits
    attacker needs try 2^10 combinations to find password

- Problem 2: hash functions optimized for performance---this _helps_
  the adversary!

  E.g., a laptop can do ~2M SHA1 operations per second.
  Even with reasonable password (20 bits entropy), crack one account/second.

Response: expensive key-derivation (e.g., PBKDF2 or BCrypt):
replace the hash with a much more expensive hash function

- Key-derivation functions have adjustable cost: make it arbitrarily slow.
  E.g., can make hash cost be 1 second -- O(1M) times slower than SHA1.

  Internally, often performs repeated hashing using a slow hash.

  Problem: adversary can build "rainbow tables".

  - Table of password-to-hash mappings.

  - Expensive to compute, but helps efficiently invert hashes afterwards.
    Only need to build this rainbow table for dictionary of common passwords.

    Roughly: 1-second expensive hash -> 1M seconds ~ 10 days to hash the
    one million most common pws.

    After that, can very quickly crack common passwords in any password db.

- Better response: **salting**

  Input some additional randomness into the password hash: H(salt, pw).
  Where does the salt value come from? Stored on server in plaintext.

  Why is this better if adversary compromises the salt too?
  Cannot build rainbow tables.

  Choose a long random salt.

  Choose a fresh salt each time user changes password.

---

How to _transmit_ passwords?

Bad idea: sending password to the server in cleartext.

Slightly better: send password over encrypted connection.

Why is this not great?

- If user mistakenly enters password of server A into server B,
  server B can learn it.

Strawman alternative: send hash of password, instead of the password.

Not so great: hash becomes a "password equivalent", can still be resent
and server B would still be able to learn it.

Better alternative: challenge-response scheme.

1. User and server both know password.

2. Server sends challenge R.

3. User responds with H(R || password).

4. Server checks if response is H(R || password).

If server knew password, server convinced user knows password.

If server did not know password, server does not learn password.

---

How to prevent server from brute-force guessing password based on H() value?

- Expensive hash + salting (as discussed earlier)
- Allow client to choose some randomness too: guard against rainbow tables.

How to defend against guessing?

- Guessing attacks are a problem because of small key space.
- Rate-limiting authentication attempts is important.
- Implement time-out periods after too many incorrect guesses.
- What to do after many failed authentication attempts?

What matters in user's password choice?

- Many sites impose certain requirements on passwords (e.g., length, chars).

- In reality, what matters is entropy.

- Format requirements rarely translate into higher entropy

- Defeats only the simplest dictionary attacks.

- Also has an unfortunate side-effect of complicating password generation.
  E.g., no single password-gen algorithm satisfies every possible web site.
  Conflicting length, symbol rules.

---

Password recovery.

- Important part of the overall security story.
- Sarah Palin's email account hack: Yahoo's recovery question was
  her date of birth and high school. Attacker looked that up in 15 seconds
  on Wikipedia and reset her password.
- Also recall Matt Honen's gmail account (Wired Journalist) from Lecture 2.
- Think of this as yet another authentication mechanism.
- Composing authentication mechanisms is tricky: are both or either required?
- Recovery mechanisms are typically "either".
- Sometimes composing "both" is a good idea: token/paper + password/PIN, etc.

---

Alternatives

Biometrics: leverage the unique aspects of a person's
physical appearance or behavior.

- How big is the keyspace?

  - Fingerprints: ~13.3 bits.
  - Iris scan: ~19.9 bits.
  - Voice recognition: ~11.7 bits.

  So, bits of entropy are roughly the same as passwords.

Scorecard (Yes is good, No is bad):

```
                      Passwords        Biometrics
Easy-to-learn:        Yes              Yes
Infrequent errors:    Quasi-yes        No
Scalable for users:   No               Yes
Easy recovery:        Yes              No
Nothing to carry:     Yes              Yes
3.5 vs 3

                      Passwords        Biometrics
Server-compatible:    Yes              No
Browser-compatible:   Yes              No
Accessible:           Yes              Quasi-yes (entering is error-prone)
3 vs 0.5
```

Not obvious that biometrics are "better" than passwords!

---

CAP (Chip Authentication Program):

- The CAP reader was designed by Mastercard to protect online banking
  transactions.

- Usage:

1. Put your credit card into the CAP reader (which looks like a hand-held
   calculator).

2. Enter PIN.

3. Reader talks to the card's embedded processor, outputs an 8-digit code
   which the user supplies to the web site.

```
                      CAP reader
Easy-to-learn:        Yes
Infrequent errors:    Quasi-yes
Scalable for users:   No (users require card+PIN per verifier)
Easy recovery:        No
Nothing to carry:     No
1.5

                      CAP reader
Server-compatible:    No
Browser-compatible:   Yes
Accessible:           No (blind people can't read 8-digit code)
1
```

In practice, deployability and usability are often more important
than security.

- Migration costs (coding + debugging effort, user training) make
  developers nervous!

- The less usable a scheme is, the more that users will complain (and try to
  pick easier authentication tokens that are more vulnerable to attackers).

- Some situations may assign different weight to different evaluation metrics.

  On a military base, the security benefits of a hardware-based token might
  outweigh the problems with usability and deployability.

---

Multi-factor authentication (MFA): defense using depth.

Requires users to authenticate themselves using two or more authentication
mechanisms.

The mechanisms should involve different modalities!

- Something you know (e.g., a password)
- Something you possess (e.g., a cellphone, a hardware token)
- Something you are (e.g., biometrics)

Idea is that an attacker must steal/subvert multiple authentication
mechanisms to impersonate a user (e.g., attacker might guess a password,
but lack access to a user's phone).

Example: Google's two-factor authentication requires a password plus a
cellphone which can receive authorization codes via text message.

MFA is a good idea, but empirical studies show that if users are given a
second authentication factor in addition to passwords, users pick much weaker
passwords!

---

Acknowledgement: Some of these notes were written by Michael Walfish
