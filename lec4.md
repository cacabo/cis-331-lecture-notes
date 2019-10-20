# Lecture 4 (Thursday September 5, 2019)

## On the board

Last lecture: Defenses against buffer overflows
Announcements:

- HW 1 due on Sept 10
- Lab 1 due on Sept 19

Agenda for Today

- Recap of Tuesday's lecture

  - Some clarifications:
  - return value VS return address
  - where are the environment variables?
  - who is setting page permissions? who is enforcing them?
  - Stack canaries
  - DEP / NX bit
  - ASLR
  - What about libc?

- How to defeat these defenses

  - ASLR:
  - heap spraying
  - NOP slide
  - Format string vulnerabilities to learn addresses

- DEP:

  - Return-oriented programming

---

# Return value versus return address

Suppose we have the following functions. I'll add made up addresses to the
margin to ease reference.

```
0  | int f() {
4  |   int a = 1;
8  |   int x = g(a);
12 |   return x;
   | }
   |
16 | int g(int a) {
20 |   return a + 1;
   | }
```

The stack when g is called looks as follows:

```
[ argument: a = 1 ][ return address: 12 ]
[ saved ebp ] <- %ebp
[ ... ][ ... ] <- %esp
```

So where is the return value??? In other words, where is a+1?
It's not in the stack! It's in register %eax.

The x86 calling convention places return values in register %eax.
This is fine for return values that are 1 word (4-bytes) or less (short,
int, memory addresses). If something is 2 words the compiler might
place it in registers %eax and %edx.

What if one needs to return something bigger, like a struct? Then the caller
allocates space in its stack, and passes a pointer to it to the callee.
This is why you often hear people say: "return by reference vs return by
value". In the case of primitive types (e.g., int) there is no difference,
both are returned in %eax. In the case of larger structs, there is a
difference: returning by reference places the address (the "reference")
in %eax. Returning by value requires the caller to allocate space in the stack.

For example:

```
  typedef struct {
    int x;
    int y;
    int z;
  } my_struct;

0  int f() {
4    int a = 1;
8    my_struct s = g(a);
12   return s.x;
   }

16 my_struct g(int a) {
20   my_struct s = {0, 0, a+1};
24   return s + 1;
   }
```

The stack when g is called looks as follows:

```
[ ... ]
[ s.z ]
[ s.y ]
[ s.x ]
[ ... ]
[ a = 1 ]
[ pointer to s ]
[ return address: 12 ]
[ saved ebp ] <- %ebp
[ ... ][ ... ] <- %esp
```

# Where are the environment variables?

We need to look more closely a process' address space

```
High addresses

[ NULL ]
[ filename of program ]
[ program environment ] <- environment variables live here
[ program arguments ]
[ stack ] <- pointers to env variables are in the stack
[ ... ]
[ heap ]
[ .bss ]
[ .data ]
[ .text ]

Low addresses
```

In particular, recall that `main` is defined as:

```c
int main(int argc, char *argv[], char *envp[]);
```

So pointers to the environment variables are in the stack,
and the actual variables are above the stack.

# Who is setting page permissions when the process starts (e.g., the NX bit)?

The ELF file format (which is the format of executable files on Linux),
contains section headers of sections of memory, and these headers contain
the field "sh_flags" (section header flags). Some of the available
flags include: "SHF_WRITE", which states whether the memory should be
writable, and "SHF_EXECINSTR", which states whether the section contains
executable machine instructions.

The _loader_ is part of the kernel and its job is to load the executable file
and place it in memory so it can run. The loader is the one responsible
for looking at these flags and setting regions of memory as writable,
executable, etc. If a region of memory does not have the SHF_EXECINSTR flag
the loader sets the associated memory pages as non-executable by setting
their corresponding non-execute (NX) bit.

A process can ask the kernel to change those permissions at runtime using the
`mprotect` system call.

The Memory Management Unit (MMU) is part of the CPU. It is the one responsible for
enforcing the memory permissions. When the CPU attempts to fetch an instruction
from a page that has the NX bit set, the MMU will notify the CPU that this is a
page fault.

# ASLR

Recall that ASLR randomizes the start address of the stack, heap, program code,
and library functions start. This makes it hard for an attacker to know
where to redirect control.

ASLR requires the collaboration of the compiler, dynamic linker, and the kernel.

## Compiler

To benefit from ASLR, shared libraries should be compiled with
position independent code (PIC) flag (-fPIC in gcc) and executables must be
compiled with position independent execution (PIE) flag (-pie in gcc). In
such a case, the compiler ensures that the program does not rely on
absolute addresses; instead, addressing is relative to the program counter
(%eip in x86).

For external functions, for example printf() which is part of libc,
instead of having call instructions reference the address of
printf (such as: "call 0x23423"), the compiler places a call to the
PC-relative address of some stub code in the Procedure Linkage
Table or PLT. The call would look like: "call 0x12344 <printf@plt>".

Aside: x86 doesn't support PC-relative addressing for "mov" instructions
since there is no instruction to get the value of %eip in x86 (this is
not an issue in x86_64). It can be done with some indirection though
(https://eli.thegreenplace.net/2011/11/03/position-independent-code-pic-in-shared-libraries/).

## Kernel

The loader performs the randomization when the process starts.

## Dynamic linker

When the process calls the stub code in the PLT (printf@plt), the stub code
essentially invokes the dynamic linker and asks it to lookup
the address of the desired function (printf in libc). The dynamic linker
looks up the address, and adds it to the Global Offset Table (GOT).

The GOT is a table with entries that map between global symbols and addresses.
The GOT lives in the .got section (yet another section in the processes'
address space!) which is at a constant offset from the code section.
Similarly, PLT lives in the .plt section. Note that since ASLR randomizes the
start of the code section, the start of the GOT and the PLT are also
randomized (but are at constant offsets from the code section).

If the stub code (printf@plt) gets called again, instead of invoking the
dynamic linker again, the stub code just looks up the address in the GOT.

What about libc? is it also randomized? Yes. Just run ldd on a program that
uses libc (nearly any program) and see for yourself!

# How to defeat stack canary

- Trial and error, one byte at a time. Use crashes as a signal.

# How to defeat ASLR

- Last lecture we discussed NOP slides and heap spraying.
- One can use format string vulnerabilities to learn where things are.

# How to defeat DEP / NX bit: Return-oriented programming

The key idea is to write an exploit input that sets up the stack ABOVE
the return address in a particular way, and then points the return address to
some other function. This is analogous to how a caller sets up the stack for the
calle (setting up arguments, return values, _another return address_, etc.).
The result is that when the function is called via a "ret" (as opposed to a
"call" instruction), the callee doesn't know it wasn't called via a "call"
instruction. As long as its stack/registers are setup properly, it'll go ahead
and execute.

Example of how to call a function in libc.

Suppose we have a function called foo:

```c
int foo(void) {
  char buf[10];
  gets(buf);
  return 0;
}
```

When foo is called its stack looks like:

```
[ ... ]
[ ret addr ]
[ saved ebp ] <- %ebp
[ ... ]
[ buf ]
[ ... ] <- %esp
```

Suppose we want to call a libc function called bar with argument x.
We should construct an input that modifies foo's stack to look like:

```
[ ... ]
[ x ]
[ other ret addr ]
[ address of bar ]
[ saved ebp ] <- %ebp
[ ... ] <- %esp
```

After the leave instruction, the stack will look like:

```
[ ... ] <- %ebp (somewhere up the stack)
[ x ]
[ other ret addr ]
[ address of bar ] <- %esp
```

After ret instruction, the stack will look like:

```
[ ... ]
[ x ]
[ other ret addr ] <- %esp
```

And the instruction pointer will be at the address of bar.
At this point, the CPU will execute the preamble of bar which pushes %ebp, moves
%esp into %ebp, decrements %esp (to allocate space in the stack), etc.

So we get:

```
[ ... ]
[ x ]
[ other ret addr ]
[ saved ebp ] <- %ebp
[ ... ]
[ ] <- %esp
```

Which looks just like the stack of bar when it is called normally!

Note that if you are jumping somewhere in the middle of the function (as opposed
to the beginning), the function preamble is _not executed_. It is the
responsibility of your exploit input to set up the stack to look as if the
preamble had been executed.

One can chain multiple functions together (or parts of functions)
by initially setting the stack, arguments, and return values, carefully.
Note that in the above example I have "other ret addr", the instruction
pointer will point there once the current function (bar) returns.
