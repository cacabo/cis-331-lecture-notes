# Lecture 1 (Tuesday August 27, 2019)

## On the board

CIS331: Introduction to networks and security

Instructor: Sebastian Angel

Email: sebastian.angel@cis.upenn.edu

Office hours: Thursday 4 -- 6 PM

Course website: https://cis.upenn.edu/~cis331

TAs:

- Petra Robertson (petrar @ seas), Monday 1:30 -- 3:30 PM

- Lawrence Dunn, (dunnla @ seas), Tuesday 12:15 -- 2:15 PM

- Natasha Gedeon (nged @ seas), Wednesday 12:30 -- 2:30 PM

- Amit Lohe (alohe @ seas), Wednesday 4 -- 6 PM

- Michael Zhou (mizho @ seas), Friday 12:30 -- 2:30 PM

Content of today's lecture:

1. Course information

- Prerequisites
- Goals
- Skills
- Structure
- Ethics

2. Administrative stuff

- Course website
- Schedule
- TAs and office hours
- Piazza
- Lecture notes
- Labs, homeworks, exams
- Grading
- Late day policy
- Academic dishonesty
- FAQ

3. What is computer security?

---

# Course information

- Prerequisites

  - CIS 240 -- First lab requires knowledge of C, assembly, and how to debug C programs
  - CIS 160 -- Basic knowledge of discrete math is assumed
  - How to use a terminal/console

- Goals
  -- Learn how to think like an attacker
  -- Learn how to reason about threats and risks
  -- Learn defensive programming
  -- Learn how to balance security cost and benefits
  -- Learn how to be an ethical hacker

- Skills you will gain
  -- Learn to carry out different attacks (overflow, SQL injection, WEP cracking)
  -- Learn to use automated tools to help you find vulnerable programs
  -- Learn to carefully review code and spot mistakes
  -- Learn about different cryptographic primitives

- Structure and topics

  - First third of the class is on Application/OS security
    -- How can attackers exploit bugs in programs.
    -- What defenses are available.
    -- How can multiple users share the same OS.
    -- Should we be using passwords?

  - Second third of the class is an introduction to cryptography
    -- What was crypto like centuries ago?
    -- What is crypto like now?
    -- How is data encrypted? What are hash functions? What are signatures?
    -- What is the "s" in HTTPS?

  - Last third of the class is on network and web security
    -- How do networks prevent introduders
    -- How do websites keep your information private?
    -- How do they keep track of your session so you don't have to log in every time?
    -- Attacks: SQL injections, cross-site request forgery (CSRF), cross-site scripting
    -- Spam, Phishing, Botnets

* Ethics
  - We will be discussing and implementing real-world attacks
  - Using some of these these techniques in the real world may be unethical,
    a violation of university policies, or a violation of state or federal law.
    -- Ethics requires you to refrain from doing harm.
    -- Always respect privacy and property rights.
  - [ Slide with U.S. Code 1030 ]
    -- up to 5 years in prison

# Mechanics and administrative stuff

- All information is in the _public_ course website

- TAs and their office hours

- Schedule is available on the course website. It shows when an assignment is available
  and when an assignment is due. All assignments should be submitted via Canvas.

- Lecture notes are published in Canvas after every lecture. They may not cover everything
  discussed in class, particularly questions and side discussions.

- Piazza: We setup a piazza site for this course. That is our preferred method
  of communication.

- Labs: There are 4 labs. These labs are to be completed in pairs. Start looking for a partner
  Today. All the labs have some easy tasks, some hard tasks, and some very hard (but fun!)
  extra credit tasks.

- Homeworks: There are 4 homeworks. These are to be completed individually.

- Exams: There is a midterm exam and a comprehensive final exam.

- Grading:
  -- Homeworks 20%
  -- Labs 35%
  -- Midterm exam 20%
  -- Final exam 25%
  -- Is there a curve? If needed, but it can only benefit you (no curving down).

- Late day policy:
  -- 5 late days to be used throughout the semester.
  -- A day counts as 24 hours. No fractional late days.
  -- Can submit any lab or homework late without notifying instructor
  -- Can use _at most_ 2 late days for any assignment
  -- If a late day is used for a lab, the late day applies to _both_ team members.
  -- Once late days are used up, no credit for late assignments (the only
  exception are extreme circumstances, to be discussed on a case by case basis)

- Academic dishonesty:
  -- Zero-tolerance policy. Do not receive solutions from others or share your solutions
  with others. It is OK to work with your lab partner, but only on the labs. Homeworks
  should be done individually.

- Special accommodations:
  -- If you need special accommodations for the exams, please come talk to me.

- FAQ:
  -- Can I do the projects on my own? Strongly discouraged because the labs are hard and
  benefit from brainstorming with others. But _can_ I? Yes.

  -- Do I need special hardware? One of the labs requires a compatible wireless card. Most
  Laptops / OSes should be compatible, but we have a few USB wireless cards available that
  you may borrow during office hours.

  -- Is there a required book? No. There are multiple recommended books, all available for
  free online. See website for info.

# What is computer security?

- Security studies how a system behaves in the presence of an _adversary_,
  which is an entity that is _actively_ trying to cause the system to
  misbehave.

- Nature vs adversarial settings are very different.
  -- Consider a natural disaster vs a planned attack by a foreign country.

  [ Example in slides ]

  -- In nature, something bad _might_ happen, but we can reason about
  expectation or average case behavior.
  -- In an adversarial setting, we need to reason about worst-case behavior,
  or at least understand the tradeoffs that we are making.
  -- Murphy's machine vs Satan's machine.

# Security mindset

- Thinking like an attacker
  -- Understand techniques for circumventing a defense
  -- Look for ways security can break, not reasons _why_ it won't.

- Thinking like a defender
  -- Know what you're defending and against whom
  -- Weight benefits VS costs: No system is ever completely secure
  -- Exercise "rational paranoia"

# Thinking like an attacker

- Look for weakest links. A defender needs to be right every time.
  As an attacker, you only need to be right once.

- Identify assumptions that security depends on. Are they false? Often they
  are.

- Do not think like the designer of the system. Do not constrain yourself.
  Think outside the box.

- [ Slide with 2 examples:
  -- Example 1: Parking lot fence. Honest users will stop. Malicious will
  go around.
  -- Example 2: Keypad. Defender thinks: as long as my stuff is protected
  by a 4 digit combination and I never leak it, my stuff is fake. Defender
  fails to consider physical wear over time.
  ]

- Thought experiment: How would you break into Levine? Any ideas?

# Thinking like a defender

(1) Security policy

- What _assets_ are we defending?

  1. SSNs?
  2. passwords?
  3. all your files?

- What properties are we trying to enforce?

  1. Authenticity: the data or message under question originates from
     the purported entity. Argument of identity and authority. For example,
     if you receive an email from kumar@seas.upenn.edu, you may ask: "is this
     email authentic?" Was it really sent by Dean Kumar? As another example,
     you may ask: is a bitcoin transaction issued by the owner of the
     bitcoin in question?

  2. Integrity: the data or message has not been modified or tampered
     by someone else. For example, has the email that Dean Kumar sent been
     altered? Is the balance in your cryptocurrency wallet what it should be?

  3. Confidentiality: the data or message should be accessible only
     to authorized parties. For example, Dean Kumar's email should only
     be readable by SEAS students. The balance of your bitcoin transaction
     should be kept hidden from those not involved in that transaction.

  [Side comment: Privacy is a related notion to confidentiality
  (and often used interchangeably) and applies to the individual
  wishing to keep his/her information private. It reasons about what can
  be learn about the person (from the data) rather than the data itself.
  Confidentiality is about hiding information that has been shared between
  parties by consent. Example of financial transaction.]

  4. Availability: the service, data, or message is accessible. For example,
     Dean Kumar's email actually arrives in students' mailboxes. The
     bitcoin transaction actually goes through.

(2) Threat models

- Who is the adversary?
- What are the capabilities?
- What kind of attacks do we want to prevent (think like the attacker!)?
- What attacks should we ignore (cost benefit analysis)?

Example.

[ Show table from Micken's paper ]

(3) Assessing risk

- What would security breaches cost us?
  -- Direct cost: money, property
  -- Indirect cost: reputation

- How likely are these costs?
  -- Probability of attacks? Is this a multi-billion dollar business
  or just your personal homepage?
  -- Probability of success?

(4) Countermeasures

- Technical (e.g., build in defenses)
- Non-technical: audits, policy, incentives
