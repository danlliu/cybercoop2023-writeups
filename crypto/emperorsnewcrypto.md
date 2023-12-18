# Emperor's New Crypto (crypto, 300 points)

> The emperor heard that the one time pad is really secure so he implemented it himself. Can you decrypt his secret message and get the flag?

## Files:

- emperors_new_crypto_encrypted.txt
```
❯ file emperors_new_crypto_encrypted.txt
emperors_new_crypto_encrypted.txt: ASCII text
```

## Solution:

Another one time pad! This time with more ciphertexts.

```
2b0d0f1e57540504311c4d0900131d01533e090a57330e040a0b560431491632180c0130064354160403592b060a02380b0659031b4f32533c00491c39460f0b14593c020a0337061f4f000004070115
034c12171e1a1c453e0901500d1b1c521e3000000e7f0e1c4f0004013a1b532b07491c3d000e1d0f4c18113a035e57370a01591f1a0326063e050b1a2b0f0e00430e3e1d450330430e0a540909044004
154c1602171848012d001e03001641523b3a4e011e3b471c001b56063e1b167f0e06017f1c0607411f03153b0700052c4f52181e104f2b4e3a481d1b3a07151c06593b070157310c184f15051000445d
0e050c5c5b0000007f0a031c1c521b1a1a31094957360952090e151173491b3a481d1b3001081c154c0d17261a0d1e310452161654183e557f1c06533b1408180659301b11573e0d084f07000a04011c

... a lot of hex strings later ...

131c0e095b0000007f000000000000005f7f080a057f0e064f1c1300320c177f1c0653371d025415040d0d7f1a0d122643051c02114f2d4f38001d487f04141a43113a4e111f30160b070048111c0115
0f011202171244457d2b0207453b4f1f062c1a45153a06004f1a06452b06532b000c533a1a0b5a434c2d173b4e111f3a43111111190d3a543309001d2c46160f0f123a0a45003617044f071c0c1f4d5d
011e04060f111a453b0c0a1e0c06165e533e1d451e394706070a0f453c08012d010c177f00071141181e1836004500370a11115010063b0631071d533a1e081d17577f1a0d122b0b091b1c0d111b445d
```

We'll use a similar approach to Wicked; I won't go through every step but it just comes down to guessing enough words and then looking up the correct phrase.

```python
from Crypto.Util.strxor import strxor

messages = [bytes.fromhex(x) for x in open('emperors_new_crypto_encrypted.txt')]

key = b'flag{'
key += strxor(b'\x1a\x0f', b'ng')
key += strxor(b'\x041\x1c', b'any')
key += strxor(b'\x18\x04\x0c\x14\x1a\x1e', b'utiful')
key += strxor(b'\x1c0\x03', b'oom')
key += strxor(b'\t\x12', b'le')
key += strxor(b'7\x0e\x1c\x08', b'hing')
key += strxor(b'\x06\x15\x001\x1d', b'icent')
key += strxor(b'\x17', b'd')
key += strxor(b'+\r\x1b\x1d', b'tern')
key += strxor(b'0\x06', b'or')
key += strxor(b'\x0e\x1d\x0f', b'ain')
key += strxor(b'\x18\x05\x1f*\x02', b'tiful')
key += strxor(b'\x0c\x18*\x10', b'ious')
key += strxor(b'\x1c\r', b'nt')
key += b'\x00' * (len(messages[0]) - len(key))

key = strxor(messages[0], b'Many, many years ago lived an emperor, who thought so much of new clothes that h')


for m in messages:
    print(strxor(m, key))
print(key)
```

```
b'flag{the_emperors_new_groove_is_his_totally_new_crypto_&_his_fancy_new_clothes!}'
```
