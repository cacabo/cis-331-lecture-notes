# Lecture 2 - Buffer overflows (August 29, 2019)

## On the board

Last lecture: Security mindset, thread models

Announcements:

- Lab 1 released today! Find your project buddy
- Lecture notes are available 5 mins before class on canvas

Agenda for today

- Recap of Tuesday's lecture
- What are buffer overflow attacks?
- How do they work?
  - How is the stack of a process set up in x86?
  - How does an overflow alter this stack?
  - How do function calls and return work?
- Why do buffer overflows exist?
- Other types of overflow

---

# Recap of last lecture: Example of security policy

- Matt Honan's case. Journalist for Wired.com
- Attacker wants to get into Matt's gmail account.
- Gmail allows reset password by sending reset link to another address.

  - It shows the actual address (is this reasonable?)
  - Address happens to be me.com (Apple service)

- Attacker now wants access to Matt's @me.com account.
- Me.com allows reset with billing address and 4 digits of cc.

  - Is this reasonable?

- Attacker learns that Matt has amazon account.

  - Amazon allows adding credit cards to a user's account!
    (their thinking: "why would an attacker _add_ a credit card?")

- Exploit:

  - Attacker adds credit card to amazon account.
  - Attacker figures out billing address for Matt.
  - Attacker resets me.com account using 4 digits of cc + billing
  - Attacker clicks on link sent to me.com account.
  - Attacker gains access to gmail account.

Moral of the story: setting security policy is hard. One needs to be
aware of what other services have. It is good to be conservative.

# What is a buffer overflow attack?

A buffer overflow attack exploits a vulnerable program that copies data
controlled by the attacker (for example arguments as part of the program,
data from a file, or inputs received through stdin) into a _buffer_
(for example an array) without checking to make sure that all of the data
fits in the buffer.

Let's focus on a concrete example: a Web server.

```
Client ----- requests-----> [ Web server ][ database ]
       <---- responses-----
```

Let's apply what we learned last class and discuss the _assets_, _security policy_,
and _threat model_.

What are the assets (i.e., what are we trying to protect)? Some possibilities:

- The Web server itself
- The OS on top of which the server runs
- Files on the same machine as the web server
- ??

What is the security policy (i.e., what are we trying to guarantee)?

- Availability (an attacker shouldn't crash the server; otherwise we might
  lose business).
- Integrity (e.g., an attacker shouldn't cause other users visiting the Web
  server to receive bogus pages).
- Confidentiality (e.g., an attacker shouldn't learn passwords stored at the server)

What is the threat model (i.e., what can the adversary do)?

- Adversary can connect to web server.
- Adversary can send any requests.

In addition to the above, we also have a _mechanism_. Which is code that
ensures the assets are covered by the security policy against the
corresponding threat model.

What is the mechanism in the case of a Web server? Well... the Web server's code!

So what happens if the mechanism is not implemented correctly? Said differently,
if the code is buggy?

- Possible for security policy to no longer be maintained.

Suppose our Web server is written in C as follows:

[ see web_server.c ]

What happens when we pass in a very big value? program crashes... Why?
Let's explore!

[ gdb demo for the code ]

Some useful gdb commands. There are many more.

- `b [function name or address]`
- `info reg`
- `p [variable or &variable]` the former prints value, the latter prints the address
- `disass [function name or address]`
- `x [register or address]` e.g. `x \$eax`
- `set {int}%esp=0x12345` sets the value of register `%esp` to `0x12345`.

Programs have a stack. Stack in x86 grows _down_ from high address to low addresses.

High addresses

```
[ args ]
[ ret ]
[ ... ]
```

Low addresses

Relevant CPU registers:

- `%eip` instruction pointer. It stores the address of the next instruction to be executed.
- `%esp` stack pointer and points to the bottom of the stack
- `%ebp` frame pointer (also called base pointer) and tells us where the
  top of the stack is _for the current frame_. This is _not_ the top of the entire
  stack, only the top of current function.
- `%eax` register used for storing operands, addresses, etc. This is a
  general-purpose register. A real workhorse! Also used to store return values.

Some relevant X86 instructions:

- `push [register]`: pushes value in register onto the stack
  (decrements `%esp` and stores the value there).
- `mov [dst, src]`: moves value of `dst` into `src`
- `mov [dst, (addr)]`: moves value of `dst` into the address pointed by `addr`
- `sub [offset, addr]`: subtracts offset from address
- `lea [offset(src_addr), dst]`: stores address `src_addr + offset` into `dst`.
- `call [address]`: calls a function. The stack should already be setup (`%esp` should
  already contain arguments).
- `ret:` pops the value at the bottom of the process's stack (`%esp`) and puts
  it into `%eip`. In other words, it sets the next instruction to be executed
  and shrinks the stack upwards by incrementing `%esp`.

Let's analyze `web_server.c`. Let's draw the stack after the program
gets to the get_req function:

```
> gdb web_server
> b read_req
> r
> disas
```

Let's work through the instructions and write down the stack. It looks
like this:

```
[ ... ]
[ args ]
[ ret ][ saved ebp ] <- ebp
[ ... ]
[ i ]
[ buf ]
[ ... ]
[ ptr to buf ] <- esp
```

What happens _before_ a function is called (i.e., before the call instruction)?

Where is `buf[0]`? `buf[1]`? why doesn't `buf` start at the bottom (`%esp`?)
What is there instead?

What does the function `gets()` do? hint: run `man gets` in the command line

What happens when we pass a big input via `stdin`?

What does the leave instruction do?

- Restores the stack back to where it was:
  - Pushes stack pointer up to `%esp` points to `%ebp`
  - Pops saved `epb` into `%ebp` (this shrinks stack so `%esp` now points at `ret`).

What happens if the return address is rewritten?

- When the function returns, if the address is invalid it'll trigger a segfault
- What if the address is _valid_ (meaning it points to valid executable
  x86 instructions) but is not the original caller's address?
  - the CPU will just start executing those instructions like
    nothing happened!
  - Note that the process might still crash later for other reasons (e.g.,
    maybe the stack for the newly called function wasn't set up properly),
    or it may corrupt state. This is one of the reasons why if you get
    a segfault, that does not mean the last instruction that trigger that
    segfault is the root cause. The real issue could've happened much earlier.

Therein lies the power a buffer overflow. It allows an attacker
to exploit a vulnerable program and hijack control of it.

- Particularly bad if the program has root permissions. Why?

Our `web_server` program is very limited though... There aren't that many valid
places where we can redirect control (recall that we need to override the return
address to valid x86 instructions).

What can we do? We can provide the code ourselves!
What if instead of supplying a bunch of garbage when overflowing the buffer,
we supply actual x86 instructions of our choosing? Then, we could redirect
control to _those_ instructions.

It is particularly useful to set those instructions to be code that
open a command line shell. That way, the attacker gets access to a shell and
can then do whatever he or she wants. This is why this code that is inserted
is often called "shellcode".

Also, as we will discuss in a few classes, one can even point ret to
functions (or even a subset of the instructions within a function) supplied
by libraries (e.g., `libc`).

# Why do buffer overflows even exist?

- Systems software is written in C (or unsafe languages)

  - Operating systems, databases, web servers, editors.

- C allows programs to manipulate raw memory addresses. Recall that we
  can do things like set the value of ret!

- C does not perform checks on bounds when copying data into buffers.

  - This is the job of the programmer

- Do buffer overflows exist in other languages?
  What about Assembly? Java? C++? Python? Rust? Go?

# Are there other types of overflow?

- Integer overflows.

  ```
  unsigned int x; // in x86 this is a 32-bit integer
  ```

  What happens if you try to store something bigger than 32-bits???
  It overflows and wraps around!

  - Often it is useful to combine multiple types of overflow.

- Format string attacks (not quite an overflow, but they go well together).

# How do we defend against these attacks?

- Use a memory safe language.
- Write bug-free code (formal verification can help).
- We'll talk about defenses introduced by compilers/hardware/OS next class.
