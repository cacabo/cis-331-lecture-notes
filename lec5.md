# Lecture 5 (Tuesday September 10, 2019)

## On the board

Last lecture: Return-oriented programming
Announcements:

- HW 1 due Today @ 10 PM
- Lab 1 due on Sept 19

Agenda for Today

1. UNIX basics (CIS 240 review)

   - How to use the terminal
   - How to compile a program
   - What is a Makefile
   - How to run a program
   - How to pass environment variables
   - How to figure out why things aren't working
   - Basic filesystem commands

2. Privilege separation and isolation

3. How does UNIX provide isolation?
   - Naming
   - Process UIDs
   - Filesystem permissions

---

1. DEMO

2. Privilege separation and isolation

The problem is bugs.

No-one knows how to prevent programmers from making mistakes.

Many bugs cause security problems.

We need ways to limit the damage from bugs that are still unknown.

Example: traditional Web server setup (Apache)

1.  Apache runs N identical processes, handling HTTP requests;
    all processes run as user 'www' (or 'httpd')

2.  Each Apache process has all application code: executes requests for many users;
    executes lots of different kinds of requests (log in, read
    e-mail, etc.) process executes application code (PHP, for example)

3.  Storage: SQL database stores application state (passwords, cookies, messages, etc.) typically one
    connection with full access to DB so the entire app has access to the entire DB

    In the event of bugs, this arrangement is very vulnerable:

    - if _any_ component is compromised, the adversary gets all
      of the data

    - Buffer overflow / code injection gives access to all data.

    - bugs in file handling may give access to sensitive files
      e.g., code might have
      `open("/profiles/" + user)`
      but now what if attacker sets "user" to
      `user=../etc/passwd` or `../mail/cis331`

Response: two big (related) ideas

1. **Privilege separation:** divide up the software and data to limit damage from bugs

   If a particular component gets compromised, then the damage of that
   component should be limited to that component alone.

   For example: split server into regions, each region in charge of
   a different part of the DB.

   [ See example ]

   designer must choose the separation scheme(s):

   - by type of data (friend lists vs passwords)

   - by user (my e-mail vs your e-mail)

   - by buggyness (image resizing vs everything else)

   - by exposure to direct attack (HTTP request parsing vs everything else)

   - by inherent privilege (hide superuser processes; hide the DB)

   Once you have done this separation, then you need a way to enforce it
   (to make sure one component cannot access another component). We do
   this via isolation.

2. **Isolation:** construct walls between units of privilege separation to prevent
   exploits in one unit from spreading to others

   Examples:

   - UNIX
   - client/server systems
   - virtual machines (each web site gets its own)
   - sandboxing
   - linux containers
   - other containers (Docker)

Challenges:

- Separation vs sharing
- Separation vs performance
- Hard to use OS to enforce isolation and control sharing.

We will study how UNIX provides isolation. But first... threat modeling!

- UNIX is a multi-user operating system (ancestor of MacOS, Linux, BSD).

- Adversary: other users!

- Policy: other users shouldn't access my files, or mess with my applications
  when they are running. Availability, confidentiality, integrity.

- But... UNIX doesn't reason in terms of users but rather processes.

3. UNIX's mechanisms for isolation and controlled sharing

   Principal: User (represented by a user ID, or UID)
   Subject: Process
   Object: Resources (files, network, memory, file descriptors, ...)

- UNIX actions are taken by processes.

  A process is a running program.

  Processes are the most basic UNIX tool for keeping code/data separate.

  A process's user ID (UID) controls many of its privileges.

  A UID is a small integer.

  Superuser (UID=0) bypasses most checks.

  A process also has a set of group IDs (GIDs) used in file permissions.

- Sharing often depends on naming.

  If a process can name something, it can often access it.

  More important: if it _can't_ name something, it usually _can't_ use it.
  We can isolate a process by limiting what names it can use.
  (sounds simple, but it's a deep idea)

  [ ASIDE (from folklore): "The Law of Names" states that knowing
  someone's, or something's, true name gives the person (who knows the true name)
  power over them. This effect is used in many folklore tales, such as in
  the German fairytale of Rumpelstiltskin: within Rumpelstiltskin,
  the girl can free herself from the power of a supernatural helper who
  demands her child by learning its name. ]

  So we want to know about the name-spaces UNIX provides:
  PIDs, UIDs, memory, files, file descriptors, network connections

- What types of objects does UNIX let processes manipulate?
  i.e., what do we need to control to enforce isolation, allow
  precise sharing?

  -- Processes.

  Processes with same UID can send signal, wait for exit & get status,
  debug (ptrace) each other.

  Otherwise not much direct interaction is allowed.

  Debugging, sending signals: must have same UID (almost).

  Various exceptions; this gets tricky in practice.

  Waiting / getting exit status: must be parent of that process.

  So: processes are reasonably well isolated for different UIDs.

  -- Process memory.

  One process cannot directly name or access memory in another process.

  Exceptions: debug mechanisms (ptrace), memory mapped files.

  So: process memory is reasonably well isolated for different UIDs.

  -- Files, directories.

  File operations: read, write, execute, change perms, ...

  Directory operations: lookup, create, remove, rename, change perms, ...

  An index node or _inode_ is a data structure that describes a filesystem
  object such as a file or a directory. inodes store attributes and also
  where in the disk the data is.

  Each inode has an owner user and group.

  Each inode has read, write, execute perms for owner, group, others.
  E.g. "lawrence staff rwxr-x---"

  You can think of it as a matrix:

  ```
        R W X
  owner 1 1 1
  group 1 0 1
  other 1 0 0
  ```

  Each row of the matrix is encoded as an octal number. Each permission
  contributes:

  ```
  R: 4
  W: 2
  X: 1
  ```

  For example, the above permission matrix can be expressed as: 754

  ```
  owner: 4 + 2 + 1 = 7
  group:     4 + 1 = 5
  other:         4 = 4
  ```

  Who can change a file's permissions?
  Only its owner (process UID).

  What does it mean if a directory has execute access but not read?

  Execute for directory means being able to lookup names (but not ls).

  What happens if a directory has read access but not execute?

  Can list file, but cannot access it so it can't even show its size!

  What checks does UNIX perform when you call open("/etc/passwd")?

  - Must be able to look up 'etc' in /, 'passwd' in /etc (x permission).

  - Must be able to open /etc/passwd (r or w permission).

  UNIX rwx scheme is simple but not very expressive;
  cannot e.g. have two owners, or permissions for specific users.

  But... one can hack around!

  How would you set up a file "hello.txt" that can only be accessed if
  uid = lawrence , and groupid = cis331?

  create: "foo/hello.txt".

  foo: rwx for cis331 group, but nobody else.
  hello.txt: rwx for lawrence, but nobody else.

So: can control which processes (UIDs) can access a specific file.
But hard to control the set of files a specific process can access.

Useful tools: chmod, chown, chgrp

chmod: changes permissions. octal masks.
chown: changes user owner (and optionally group)
chgrp: changes group owner

-- File descriptors (FDs).

A process has one FD per open file and open IPC/network connection.
File access control checks performed at file open.
Once process has an open file descriptor, can continue accessing.
Processes cannot see or interfere with each others' FDs.
Processes can pass file descriptors (via Unix domain sockets).
So: FDs are well isolated -- process-local names, not global.

What happens if a file's permission changes when another process
has already opened the file?

-- Local IPC -- "Unix domain sockets" -- socketpair().
A process can create a connection -- gets two FDs.
It can then give the connection end FDs to other processes,
either via fork()/exec() or by sending over existing connections.

So: Unix domain connections are well isolated.

-- Networking.

Very different than everything else. Perhaps because it didn't
exist when UNIX was originally designed.

Operations:
bind to a port
connect to some address
read/write a connection
send/receive raw packets

Rules:

    - any process can connect to any port as a client.

    - only root (UID 0) can bind to ports below 1024.
    Why? Mostly to prevent any random user from taking over "well-known"
    services like a web server (which runs on port 80) or ssh which
    (runs on port 22).

    - can only read/write data on connection that a process has an fd for.
    (not really true; bad people may snoop/inject on network)
    (So: servers have to be careful who they talk to.)

    - only root can send/receive raw packets (i.e., without socket).
    Why? well, to prevent users from "spoofing" packets. Otherwise you could
    in principle send a packet to someone pretending to come from port 80
    (messing with existing connections).

How is a process's UID set?
Superuser (UID 0) can call setuid(uid) and setgid(gid).
Non-superuser processes can't change their UID (to first approx)
UID/GID often initially set by login, from /etc/passwd.
UID inherited during fork(), exec().

Where does the user ID, group ID list come from?
On a typical UNIX system, login program runs as root (UID 0)
Checks supplied user password against /etc/shadow.
Finds user's UID based on /etc/passwd.
Finds user's groups based on /etc/group.
Calls setuid(), setgid(), setgroups() before running user's shell

In other words:

    When a user logs into a UNIX machine, the user interacts with the login
    process. When the user successfully logs in, the first process for the
    user is launched: "the shell"! During this launch, the login program
    sets the permissions by reading the user's UID/GID from the /etc/passwd file.

    Since UID is inherited during fork()/exec(), when you call a program from
    your shell, you are essentially doing both: fork and exec! So the running
    program that you just launched inherits the permissions of the shell.

How do you regain privileges after switching to a non-root user?

    - Could use file descriptor passing (but have to write specialized code)

    - Kernel mechanism: setuid/setgid binaries.

      When the binary is executed, set process UID or GID to binary owner.

      Specified with a special bit in the file's permissions

      For example, su / sudo binaries are typically setuid root.
      Even if your shell is not root, can run "su otheruser"
      su process will check passwd, run shell as otheruser if OK.

      Many such programs on Unix, since root privileges often needed.

      Why might setuid-binaries be a bad idea, security-wise?

        - Many ways for adversary (caller of binary) to manipulate process.
        - In Unix, exec'ed process inherits environment vars, file descriptors, ..
        - Libraries that a setuid program might use not sufficiently paranoid

      Historically, many vulnerabilities (e.g. pass $LD_PRELOAD, ..)

One more Unix isolation trick: chroot()

    Problem: it is too hard to ensure that there are no
      sensitive files that a program can read, or write;
      100,000+ files in a typical Unix install; applications
      are often careless about setting permissions.

    Solution: chroot(dirname)
      causes / to refer to dirname for this process and descendants,
      so they can't name files outside of dirname.

      e.g. chroot("/home/cis331/") causes subsequent absolute pathnames
      to start at /home/cis331/, not the real /.

      Thus the program can only name files/dirs under /home/cis331/.

      "/../" will not get you out of the chroot environment. UNIX changes
      how it evaluates ".." in the chrooted directory, by having ".." point
      to ".".

      chroot() is typically used to prevent a process from interacting
      at all with other processes via files, i.e. complete isolation.

      chroot environment can be escaped if attacker can call chroot.
      Suppose like before that we call chroot("/home/cis331").
      So the kernel maps "/" to "/home/cis331" for this process.

      Attack:
      ```
      fd = open("/") // we now have file descriptor pointing to /home/cis331/
      chroot("/public") // now the root ("/") -> "/home/cis331/public/"
      fchdir(fd) // we are now at "/home/cis331/"
      chdir("../..") // we are now at "/"
      // At this point attacker can access /etc/shadow.
      ```

Overall, Unix is awkward at precisely-controlled isolation+sharing:
Many global name spaces: files, UIDs, PIDs, ports.
Each may allow processes to see what others are up to.
Each is an invitation for bugs or careless set-up.

No idea of "default to no access".
Thus hard for designer to reason about what a process can do.

No fine-grained grants of privilege.
Can't say "process can read only these three files."
Privileges are coarse-grained, via UID, or implicit, e.g. wait() for children.

Chroot() and setuid() can only be used by superuser.
So non-superusers can't reduce/limit their own privilege.
Awkward since security suggests _not_ running as superuser.

why is it a security vulnerability if chroot is setuid root?
like what happens if user processes can "confine" themselves?

attack:
--attacker sets up jail's directory /tmp/dir
--within /tmp/dir, hard link to
passwd, su, login programs (not many restrictions placed on
hard linking):
/tmp/dir/sbin/passwd
/tmp/dir/sbin/login
etc.
--create fake /tmp/dir/etc/passwd
--chroot() into /tmp/dir
--the binaries are hard-coded to look at /etc/passwd. when they
run in the jail, they will be looking at the wrong version
--yet, they will have privilege (they are setuid)
--result: they will apply their privilege to the wrong
environment, and allow the attacker to, say, login as root...

---

Acknowledgement: Some of these notes were written by Michael Walfish
