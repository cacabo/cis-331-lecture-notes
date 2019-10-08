# Lecture 3 (Tuesday September 3, 2019)

## On the board

Las lecture: Buffer overflows
Announcements:

- Homework 1 released today! Due on September 10
- Lab 1 due on September 19

Agenda for Today

- Trivia: where is this place?
- Recap of Thursday's lecture
  -- Buffer overflows
  -- Integer overflows
  -- Format string attacks
- How can we defend against buffer overflows?
  -- Tools and good programming hygiene
  -- Stack canaries
  -- Bounds checking
  -- NX bit
  -- ASLR

---

# Recap of Thursday's lecture

Suppose we have the program:

f() {

//some code

int x = g(foo, bar);
return x;
}

g(foo, bar) {
char buf[100];

// do stuff
gets(buf);

// do other stuff
}

Remember that function f sets up the stack for g by:
(1) placing the arguments foo and bar in the stack
(2) placing the return address in the stack

When g starts, its stack looks like

[ ... ][ bar ]
[ foo ][ return addr ]
[ saved ebp ] <- %ebp
[ ... ] buf[99] ]
[ ... ] buf[0] ]
[ ... ] <- %esp

Buffer overflows work by copying too much data into the buffer,
which then overrides the return address (and/or other local variables)
and has it point to some other function (usually shellcode).

When function g calls the "leave" instruction, %esp goes back to %ebp,
then it pops current value in the stack and sets it to %ebp (so %ebp
becomes "saved ebp", and %esp now points to the return addr).

When function g calls the "ret" instruction, it pops the current value in
the stack and places it in %eip (the instruction pointer). This is what
causes execution to go either back to the caller (function "f", in the case of
normal execution) or somewhere else (in the case of an attack).

## Integer overflows

Recall also that you can have integer overflows.
If you have two 32-bit integers and you add them together, and the result
is more than 32-bits, only the least significant 32-bits are kept.
This is equivalent to performing the operation modulo 2^32. See Piazza
post for more detail.

How to prevent?

You need to add checks (or use checks provided by language/compiler).
For example, the following check detects overflow during multiplication:

int sum = x \* y;
if (x != 0 && sum / x != y) {
// OVERFLOW!
}

## Format string vulnerabilities

The function snprintf has a _variable_ number of arguments. The third argument
is supposed to be the format string. However, if one is not careful and passes
the input instead, that can be exploited (the input can contain
characters such as "%x"). It is very important to double check whether a
function has a variable number of arguments, and what does arguments mean.

# How can we defend against buffer overflow attacks?

1. Write bug-free code. If there are no bugs, the attacker cannot exploit
   the bugs. This means do not use unsafe operators, always specify
   the size of the data to copy to the buffer (use functions like `fgets'' instead of`gets'' which has the additional length parameter), perform
   overflow checks whenever you add and multiple integers, etc.

2) Use memory safe language: Java, Python, C#, Haskell, Rust, Go.

- This is great for new code, but we still need to learn about, and defend
  against, buffer overflows in all the legacy code that already exists.

- Accessing hardware still requires low-level code which if not done
  correctly can open the door to buffer overflows!

- Python runtime itself could have a buffer overflow. Rust unsafe code
  for handling disk IO could have a buffer overflow, etc.

3. Use tools to find and fix bugs. You will experiment with one such
   tool (the AFL security fuzzing tool) in Homework 1.

4) Stack canaries. A reference to caged canary birds that miners would
   carry down into the mines with them. If dangerous gases
   such as CO2 built up, the canaries would die before the miners. This served
   as a warning to the miners.

   Similarly, we can add "stack canaries" to our stack. They are essentially
   an entry with a particular value that is placed below the return address:

[ args ][ ret addr ]
[ saved ebp ] <- %ebp
[ ... ][ canary ]
[ ... ][ buf ]
[ ... ] <- %esp

Before the ret instruction is executed, the compiler adds additional
instructions that compare the current value of the canary to its original value.
If the values do not match, that serves as a warning sign that something went
wrong ("the canary is dead due too much gas buildup"), and we should run to
the hills and stop execution.

But what happens if the attacker knows the value of the canary?
Couldn't the attacker simply "replace it" as part of the overflow?
Yup...

A few defenses:

(i) canary could be copy-unfriendly terminator characters.
For example: "\0\CR\LF-1". If the buffer overflow exploits a
string copy, the \0 will be hard to write. Similar idea for
carriage return (CR), line feed (LF), and the value -1.
  
 This only works in cases where these characters stop the copying.

(ii) the canary could be randomly generated on every run. When the program
starts, it randomly generates a canary and stores it in a global
variable (it is hard for the attacker to get access to that canary,
since in x86 global variables are stored in the data segment---separate
from the stack and the heap).

      Wait... where is this "data" segment you speak of?


        High addresses

        Stack (grows down)
        ...
        Heap (grows up)
        Uninitialized data (.bss segment)
        Initialized data (.data segment)
        Code (.text segment)

        Low addresses


       Not a perfect defense. Adversary could still read the canary if it
       knows its address in memory or if it can read from the stack (since
       it can read the value of the canary in the stack!).

(iii) random canary XORed with control data. Instead of storing the random
canary in the stack below "ret addr", the program can store:
"ret addr XOR saved ebp XOR canary XOR [any other control data]" into
the stack. It then works the same as a canary, except that it is a
bit more difficult for the attacker to figure out.

Note that canaries fail to detect overwritten pointers (e.g., function pointers)
  
 int foo() {
void (\*p)(int) = &some_function;
char buf[100];
p(123);
}

An overflow can change the value of p, and hijack execution that way.

Could you add canaries to every single pointer? Yes, but horribly expensive.
Check out: "Code pointer integrity" by Kuznetsov et al. for a better
approach.

5. Bounds checking. Make sure that a pointer can only touch the memory
   that has been allocated to it.


    (i) Electric fences:

    Every allocated memory chunk (e.g., buffer) is preceded by a guard page.
    This page has special permissions such that any access triggers a fault.

    [guard page]
    buf1

    [guard page]
    buf2

    Useful for debugging, but horribly inefficient in terms of space,
    because the pages have a minimum size (usually 4KB) required by the
    hardware (since we are relying on a hardware mechanism).


    (ii) Fat pointers:

    Changing pointers to include bounds information in the pointer itself.

    Existing pointer:  [ 32-bit address ]
    Fat pointer:   [32-bit start addr][32-bit end addr][32-bit ptr addr]


    Example:

      int *ptr = malloc(12); // has 3 elements
      ptr[2] = 1;


      The ptr has information on beginning, end, and current, and can
      check the bounds.


    Drawbacks:

      - fat pointers are big! They make data structures and everything bigger.

      - Legacy code doesn't understand fat pointers so you can't just call an
        existing library and pass a fat pointer to it.

      - Updates are not atomic because we have 3 words to describe a
        pointer instead of one. This makes multithreaded code a bit of a
        nightmare. Suppose 2 threads try to update a fat pointer at the same time.
        With normal pointers one of the two threads will win, and the pointer
        will point to one of the two potential addresses (the one set by
        thread 1, or the one set by thread 2). Since updating fat pointers
        is not atomic, you can run into situations where the final result is
        something inconsistent (e.g., the bounds set by one thread, the
        address set by the other thread).

    (iii) More advanced bound checking: baggy bounds checking by Akritidis et al.
      - Out of scope of this course but you can check it out!

6. Non-executable stack

Recall that many attacks relied on the attacker supplying its own
code ("shellcode"), placing it into the stack, then forwarding control to it.

In modern hardware, each memory page has 3 permission bits:
R: read
W: write
X: execute

By setting the NX bit (i.e., setting X bit to 0) on all the memory pages
that make up the stack, if the CPU ever attempts to execute an instruction
in a non-executable page, it will trigger a page fault.

What about the rest of memory? A lot of systems (OSes) usually set the
following policy: Memory is "W^X". This means memory can either be
"writable" or "executable" but not both.

Drawbacks:

- This prevents benign code rewriting at runtime (common
  in interpreted languages like Python for just-in-time compilation). There
  are ways to get around.

- You can still overflow the stack and change the return address to
  _existing_ code (which is presumably in an executable page).

7. Address Space Layout Randomization (ASLR)

The attacks that we've discussed so far rely on the attacker being able to
point the return address to the address of our shellcode (or some other function).
The attacker can do this easily because the addresses are hardcoded (so
the attacker can just disassemble the program and find them).

ASLR proposes moving where things are in memory. So the stack, heap,
etc. no longer starts at the same place on every execution. Instead,
its start address is randomized.

Even if an attacker uses gdb and figures out where the stack is
on _their_ machine on a particular run, that doesn't help them figure out
where the stack starts on a target's machine.

How to defeat ASLR:
  
 (i) Replicate shellcode: Write the shellcode many times all over
memory! That way you may get lucky and point the return address to one of
the many copies of the shellcode. Drawback: if you don't point the return
address to the _beginning_ of the shellcode the attack might not work
properly.

    (ii) NOP slide: You can pad the beginning of shellcode with a lot of
    NOP instructions ("\x90" in x86). The CPU simply ignores these NOP
    instructions and moves on to the next instruction. If your shellcode
    has 100 NOP instructions before the real instructions, as long as your
    return address hits one of the 100 NOP instructions, your shellcode
    will be called correctly. Execution will essentially "slide" through
    these NOPs until it hits the first real instruction.
