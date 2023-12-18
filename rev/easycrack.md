# easycrack (rev, 200 points)

> I need to make a key for this crackme for my homework. I just want to play video games, can you make a valid key for me? You can submit it here as the flag.

## Files:

- easycrack

```
❯ file easycrack
easycrack: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=9e2ef9c0697b3b9c323113e09e6b60af2c97b0ec, not stripped
```

## Solution:

Back to Ghidra!

```c
undefined8 main(void)

{
  size_t len;
  long in_FS_OFFSET;
  int tot;
  int i;
  char key [24];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  printf("Please enter a key: ");
  fgets(key,0xd,stdin);
  tot = 0;
  i = 0;
  while( true ) {
    len = strlen(key);
    if (len <= (ulong)(long)i) break;
    tot = tot + key[i];
    i = i + 1;
  }
  if (tot == 0x539) {
    len = strlen(key);
    if (len == 0xc) {
      printf("Nice key :) %s",key);
      goto LAB_00400730;
    }
  }
  printf("Bad Key :( %s",key);
LAB_00400730:
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

It's a pretty simple binary: we sum up the ASCII values of the characters in the key, and check that the total is `0x539` / `1337` (nice). Then, we check that our key is `0xc` / `12` characters in length. The easiest way is to split the values up as evenly as possible: 1337 / 12 = 111 with a remainder of 5. Thus, we will put 7 characters with value 111 (`o`) and 5 characters with value 112 (`p`).

`flag{oooooooppppp}`
