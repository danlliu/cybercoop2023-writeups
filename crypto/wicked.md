# wicked (crypto, 200 points)

> The dev team implemented their own crypto that's supposed to be perfect. Like super perfect. But there's no way they did it right... They gave me some ciphertext to try to decrypt. Can you help me?

## Files:

- ciphertext.txt
```
❯ file ciphertext.txt
ciphertext.txt: ASCII text
```

## Solution:

Let's take a look at the contents of `ciphertext.txt`:

```
280316470f1f0d45250c3c0a17017f3205040b0d420e39411a0c3a473b0c1d104137083b540a1a0147071a3a540d1016735718162b571d0b0a1144
110d12471a0448151d123a13141033450d03480442153a0d0b173c081c0c424400310d7f17071a190348073a11480c053a05180437121b06454544
35034d471a0448161a007f1213117f0c02501c0d07413b0e01167f080a490601137f0a3e071c03104b480737114801122f07041d3a134917044544
0a030e0c5b161a0a070b3b41130b3b451f111f45260e2d0e1a0c26470010070a067f082c180d0a054b4803360000491b3a0541152d1e0c0d0f1644
07000d471a15071006453704004b7f310415114515042d044e057f0b0307094405361a2b15060c104707123958480b062b57151b3a5749434b4544
3105020c1e1348321b113c0952123e164c11060210187f150144390e020d4e10093a047f1d064f1d021a543c1b1d07072d0e5a532c184910030044
040004105b02180a1c453e41010c3313090248120a082c1502017f1304081a44092a073854091d1a1206107f1c0d1b5331120218715749434b4544
```

These look like hex strings... let's print them out:

```python
with open('ciphertext.txt') as f:
    texts = [bytes.fromhex(l) for l in f]

for t in texts:
    print(t)
```

```
❯ python3 solve.py
b'(\x03\x16G\x0f\x1f\rE%\x0c<\n\x17\x01\x7f2\x05\x04\x0b\rB\x0e9A\x1a\x0c:G;\x0c\x1d\x10A7\x08;T\n\x1a\x01G\x07\x1a:T\r\x10\x16sW\x18\x16+W\x1d\x0b\n\x11D'
b'\x11\r\x12G\x1a\x04H\x15\x1d\x12:\x13\x14\x103E\r\x03H\x04B\x15:\r\x0b\x17<\x08\x1c\x0cBD\x001\r\x7f\x17\x07\x1a\x19\x03H\x07:\x11H\x0c\x05:\x05\x18\x047\x12\x1b\x06EED'
b'5\x03MG\x1a\x04H\x16\x1a\x00\x7f\x12\x13\x11\x7f\x0c\x02P\x1c\r\x07A;\x0e\x01\x16\x7f\x08\nI\x06\x01\x13\x7f\n>\x07\x1c\x03\x10KH\x077\x11H\x01\x12/\x07\x04\x1d:\x13I\x17\x04ED'
b'\n\x03\x0e\x0c[\x16\x1a\n\x07\x0b;A\x13\x0b;E\x1f\x11\x1fE&\x0e-\x0e\x1a\x0c&G\x00\x10\x07\n\x06\x7f\x08,\x18\r\n\x05KH\x036\x00\x00I\x1b:\x05A\x15-\x1e\x0c\r\x0f\x16D'
b'\x07\x00\rG\x1a\x15\x07\x10\x06E7\x04\x00K\x7f1\x04\x15\x11E\x15\x04-\x04N\x05\x7f\x0b\x03\x07\tD\x056\x1a+\x15\x06\x0c\x10G\x07\x129XH\x0b\x06+W\x15\x1b:WICKED'
b'1\x05\x02\x0c\x1e\x13H2\x1b\x11<\tR\x12>\x16L\x11\x06\x02\x10\x18\x7f\x15\x01D9\x0e\x02\rN\x10\t:\x04\x7f\x1d\x06O\x1d\x02\x1aT<\x1b\x1d\x07\x07-\x0eZS,\x18I\x10\x03\x00D'
b'\x04\x00\x04\x10[\x02\x18\n\x1cE>A\x01\x0c3\x13\t\x02H\x12\n\x08,\x15\x02\x01\x7f\x13\x04\x08\x1aD\t*\x078T\t\x1d\x1a\x12\x06\x10\x7f\x1c\r\x1bS1\x12\x02\x18qWICKED'
```

One thing we see here is the string `WICKED` at the end of a few messages (or substrings of it). This, combined with the description, suggests that we're using a one-time pad, but we're reusing the key. Based on the length of messages, we could guess that one is likely the flag. Let's try XORing each message with `flag{`, and look for runs of `\x00` bytes at the beginning:

```python
with open('ciphertext.txt') as f:
    texts = [bytes.fromhex(l) for l in f]

from Crypto.Util.strxor import strxor
xorkey = b'flag'
xorkey += b'\x00' * (len(texts[0]) - len(xorkey))

for t in texts:
    print(strxor(xorkey, t))
```

```
❯ python3 solve.py
b'Now \x0f\x1f\rE%\x0c<\n\x17\x01\x7f2\x05\x04\x0b\rB\x0e9A\x1a\x0c:G;\x0c\x1d\x10A7\x08;T\n\x1a\x01G\x07\x1a:T\r\x10\x16sW\x18\x16+W\x1d\x0b\n\x11D'
b'was \x1a\x04H\x15\x1d\x12:\x13\x14\x103E\r\x03H\x04B\x15:\r\x0b\x17<\x08\x1c\x0cBD\x001\r\x7f\x17\x07\x1a\x19\x03H\x07:\x11H\x0c\x05:\x05\x18\x047\x12\x1b\x06EED'
b'So, \x1a\x04H\x16\x1a\x00\x7f\x12\x13\x11\x7f\x0c\x02P\x1c\r\x07A;\x0e\x01\x16\x7f\x08\nI\x06\x01\x13\x7f\n>\x07\x1c\x03\x10KH\x077\x11H\x01\x12/\x07\x04\x1d:\x13I\x17\x04ED'
b'look[\x16\x1a\n\x07\x0b;A\x13\x0b;E\x1f\x11\x1fE&\x0e-\x0e\x1a\x0c&G\x00\x10\x07\n\x06\x7f\x08,\x18\r\n\x05KH\x036\x00\x00I\x1b:\x05A\x15-\x1e\x0c\r\x0f\x16D'
b'all \x1a\x15\x07\x10\x06E7\x04\x00K\x7f1\x04\x15\x11E\x15\x04-\x04N\x05\x7f\x0b\x03\x07\tD\x056\x1a+\x15\x06\x0c\x10G\x07\x129XH\x0b\x06+W\x15\x1b:WICKED'
b'Wick\x1e\x13H2\x1b\x11<\tR\x12>\x16L\x11\x06\x02\x10\x18\x7f\x15\x01D9\x0e\x02\rN\x10\t:\x04\x7f\x1d\x06O\x1d\x02\x1aT<\x1b\x1d\x07\x07-\x0eZS,\x18I\x10\x03\x00D'
b'blew[\x02\x18\n\x1cE>A\x01\x0c3\x13\t\x02H\x12\n\x08,\x15\x02\x01\x7f\x13\x04\x08\x1aD\t*\x078T\t\x1d\x1a\x12\x06\x10\x7f\x1c\r\x1bS1\x12\x02\x18qWICKED'
```

Ah, the _key_ is the flag, rather than the flag being encrypted in one of the messages. Slowly, we can figure out parts of the flag from the incomplete words:

```python
with open('ciphertext.txt') as f:
    texts = [bytes.fromhex(l) for l in f]

from Crypto.Util.strxor import strxor
xorkey = b'flag'
xorkey += strxor(b'\x1e\x13', b'ed')
xorkey += strxor(b'\r', b'e')
xorkey += strxor(b'\n\x07\x0b;', b'ound')
xorkey += strxor(b'\n\x17\x01', b'ked')
xorkey += b'\x00' * (len(texts[0]) - len(xorkey))

for t in texts:
    print(strxor(xorkey, t))
```

```
❯ python3 solve.py
b'Now the Wicked\x7f2\x05\x04\x0b\rB\x0e9A\x1a\x0c:G;\x0c\x1d\x10A7\x08;T\n\x1a\x01G\x07\x1a:T\r\x10\x16sW\x18\x16+W\x1d\x0b\n\x11D'
b'was as powerfu3E\r\x03H\x04B\x15:\r\x0b\x17<\x08\x1c\x0cBD\x001\r\x7f\x17\x07\x1a\x19\x03H\x07:\x11H\x0c\x05:\x05\x18\x047\x12\x1b\x06EED'
b'So, as she sat\x7f\x0c\x02P\x1c\r\x07A;\x0e\x01\x16\x7f\x08\nI\x06\x01\x13\x7f\n>\x07\x1c\x03\x10KH\x077\x11H\x01\x12/\x07\x04\x1d:\x13I\x17\x04ED'
b'look around an;E\x1f\x11\x1fE&\x0e-\x0e\x1a\x0c&G\x00\x10\x07\n\x06\x7f\x08,\x18\r\n\x05KH\x036\x00\x00I\x1b:\x05A\x15-\x1e\x0c\r\x0f\x16D'
b'all about her.\x7f1\x04\x15\x11E\x15\x04-\x04N\x05\x7f\x0b\x03\x07\tD\x056\x1a+\x15\x06\x0c\x10G\x07\x129XH\x0b\x06+W\x15\x1b:WICKED'
b'Wicked Witch w>\x16L\x11\x06\x02\x10\x18\x7f\x15\x01D9\x0e\x02\rN\x10\t:\x04\x7f\x1d\x06O\x1d\x02\x1aT<\x1b\x1d\x07\x07-\x0eZS,\x18I\x10\x03\x00D'
b'blew upon a si3\x13\t\x02H\x12\n\x08,\x15\x02\x01\x7f\x13\x04\x08\x1aD\t*\x078T\t\x1d\x1a\x12\x06\x10\x7f\x1c\r\x1bS1\x12\x02\x18qWICKED'
```

From here, we can find the original plaintext by searching for the initial fragments:

> Now the Wicked Witch of the West had but one eye, yet that was as powerful as a telescope, and could see everywhere. So, as she sat in the door of her castle, she happened to look around and saw Dorothy lying asleep, with her friends all about her. They were a long distance off, but the Wicked Witch was angry to find them in her country; so she blew upon a silver whistle that hung around her neck.

We'll grab the first line, and use it as our new key:

```python

with open('ciphertext.txt') as f:
    texts = [bytes.fromhex(l) for l in f]

from Crypto.Util.strxor import strxor
xorkey = strxor(texts[0], b'Now the Wicked Witch of the West had but one eye, yet that ')

print(xorkey)
for t in texts:
    print(strxor(xorkey, t))
```

```
❯ python3 solve.py
b'flag{where_are_elpheba_and_glinda_i_thought_this_was_wicked'
b'Now the Wicked Witch of the West had but one eye, yet that '
b'was as powerful as a telescope, and could see everywhere.  '
b'So, as she sat in the door of her castle, she happened to  '
b'look around and saw Dorothy lying asleep, with her friends '
b'all about her. They were a long distance off, but the      '
b'Wicked Witch was angry to find them in her country; so she '
b'blew upon a silver whistle that hung around her neck.      '
```
