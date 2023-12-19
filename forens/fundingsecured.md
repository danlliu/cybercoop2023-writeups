# Funding Secured (forensics, 200 points)

> Someone in our company leaked some very sensitive information. We absolutely cannot let this stand.

Thankfully our monitoring software intercepted the screenshot that was leaked. An old engineer of ours did write some kind of watermarking for screenshots but we have no idea how it works. Can you figure it out?

## Files:

- captured.png
```
❯ file captured.png
captured.png: PNG image data, 600 x 127, 8-bit/color RGB, non-interlaced
```

## Solution:

![diamond hands](../assets/fundingsecured/captured.png)

Again, we'll try our standard gauntlet of steg tools:

```
❯ binwalk captured.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 600 x 127, 8-bit/color RGB, non-interlaced
54            0x36            Zlib compressed data, default compression
```

Nothing really here.

Let's try another tool: zsteg.

```
❯ zsteg captured.png
imagedata           .. text: "xxxrrr..."
b1,r,msb,xy         .. text: "(*LLUejO"
b1,rgb,lsb,xy       .. text: "creator.txtUT\r"
b1,bgr,msb,xy       .. file: OpenPGP Public Key Version 2
b2,r,msb,xy         .. text: "UUUUUU}UUu"
b2,g,msb,xy         .. text: "}WWu_]W]W"
b2,b,msb,xy         .. text: "UUUUUUWU"
b2,rgb,msb,xy       .. text: "]UUUWUUUWUU"
b2,bgr,msb,xy       .. text: "U]]UUUWUUuUUu"
b4,r,msb,xy         .. text: ["w" repeated 10 times]
b4,g,msb,xy         .. text: ["w" repeated 10 times]
b4,b,msb,xy         .. text: ["w" repeated 12 times]
b4,rgb,msb,xy       .. text: ["w" repeated 33 times]
b4,bgr,msb,xy       .. text: ["w" repeated 34 times]
```

Ooh... `creator.txt`. Let's extract it:

```
❯ zsteg -E 'b1,rgb,lsb,xy' captured.png > extracted.dat
❯ file extracted.dat
extracted.dat: data
❯ binwalk extracted.dat

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
3             0x3             Zip archive data, at least v2.0 to extract, uncompressed size: 103, name: creator.txt
163           0xA3            Zip archive data, at least v2.0 to extract, uncompressed size: 4882, name: exif.txt
1904          0x770           Zip archive data, at least v2.0 to extract, uncompressed size: 49, name: flag.txt
2298          0x8FA           End of Zip archive, footer length: 22
❯ binwalk -e extracted.dat

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
3             0x3             Zip archive data, at least v2.0 to extract, uncompressed size: 103, name: creator.txt
163           0xA3            Zip archive data, at least v2.0 to extract, uncompressed size: 4882, name: exif.txt
1904          0x770           Zip archive data, at least v2.0 to extract, uncompressed size: 49, name: flag.txt
2298          0x8FA           End of Zip archive, footer length: 22

❯ ls
_extracted.dat.extracted  captured.png  extracted.dat
❯ cd _extracted.dat.extracted
❯ ls
3.zip  creator.txt  exif.txt  flag.txt
❯ cat flag.txt
File: flag.txt
flag{what_came_first_the_stego_or_the_watermark}
```
