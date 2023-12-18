# nullnull (crypto, 300 points)

> Welcome to my secure flag sharer where there are null nulls!

## Files:

- server.py
```
❯ file server.py
server.py: Python script text executable, ASCII text
```

- `nc 0.cloud.chals.io 15076`

## Solution:

Let's take a look at `server.py`:

```python
import random

def generate(n):
    key = b""
    for i in range(n):
        key += chr(random.randint(1, 127)).encode() #can't have the flag leaking, can we?
    return key


def server():  
    print("Welcome to my secure flag sharer where there are null nulls!")
    
    flag = open("flag.txt", "rb").read()
    while True:
        guess = input("Print encrypted flag? (Y/N): ")

        if str(guess).upper() == "Y":
            enc = b""
            key = generate(64)
            for i in range(len(flag)):
                enc += chr(flag[i] ^ key[i] % 128).encode()
            print(enc)
        else:
            exit()

if __name__ == "__main__":
    server()
```

It looks like we can ask it for a one-time pad encrypted version of the flag, where the key is generated randomly. If we run the server locally, we can see that we get a different key every time.

One of the key observations here is that `random.randint` is not called with `0, 127`. Instead, it's called with `1, 127`. Thus, we'll never generate `0` as the XOR key. With enough tries, we should be able to generate every character _except_ the correct character.

```python
from pwn import *

flaglen = 49
chars = []
for _ in range(flaglen):
    chars.append(set())


def get_keys():
    result = []
    try:
        con = remote('0.cloud.chals.io', 15076)
        for _ in range(256):
            con.recvuntil(b'(Y/N): ')
            con.sendline(b'Y')
            enc = eval(con.recvline())
            result.append(enc)
            print('.', end='')
    except e:
        pass
    print()
    for r in result:
        for i in range(flaglen):
            chars[i].add(r[i])
    for s in chars:
        print(len(s), end=' ')
    print()


while any(len(x) != 127 for x in chars):
    get_keys()

for s in chars:
    for i in range(128):
        if i not in s:
            print(chr(i), end='')
```

This asks the server for 256 different encrypted flags over and over until we get every position narrowed down. It'll take a little to run, but eventually:

```
❯ python3 solve.py
[+] Opening connection to 0.cloud.chals.io on port 15076: Done
................................................................................................................................................................................................................................................................
109 111 114 111 110 107 107 115 114 109 113 108 114 113 111 107 113 114 111 103 105 110 115 107 115 108 116 111 113 116 109 107 112 110 108 109 108 111 111 110 113 108 106 116 111 108 113 113 116

... more output ...

127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127 127
flag{so_many_possibilities_but_only_one_solution}
```
