# babyhide (forensics, 100 points)

> This little baby is figuring out how to computer! It looks like the baby hid some of my files though. I have no idea what to do, can you get my files back?

## Files:

- babyhide.jpeg
```
❯ file babyhide.jpeg
babyhide.jpeg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 1280x853, components 3
```

## Solution:

Opening up the image doesn't reveal too much:

![babyhide](../assets/babyhide/babyhide.jpeg)

Let's try our usual suite of tools, starting with `binwalk`:

```
❯ binwalk babyhide.jpeg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
382           0x17E           Copyright string: "Copyright (c) 1998 Hewlett-Packard Company"
117430        0x1CAB6         PDF document, version: "1.3"
```

Ooh, a PDF. Let's extract it.

```
❯ binwalk --dd='.*' babyhide.jpeg
❯ cd _babyhide.jpeg.extracted
❯ ls
0  1CAB6  17E
❯ file 1CAB6
1CAB6: PDF document, version 1.3, 1 pages
❯ mv 1CAB6 1CAB6.pdf
```

Finally, we get our PDF:
[the result!](../assets/babyhide/1CAB6.pdf)

Inside, we find the flag `flag{baby_come_back}`.
