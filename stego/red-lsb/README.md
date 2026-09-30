# RED, RED, RED, RED

## Approach
Given a flat, solid-red `red.png`. Hints:
1. "The picture seems pure, but is it though?"
2. "Red? Ged? Bed? Aed?" — pointing at the individual colour channels.
3. "Check whatever Facebook is called now." — Meta → metadata.

Started with the metadata angle first (`exiftool`, `strings`), then moved to pixel-level
inspection since the image "looked" like a single flat colour.

## Solution
`exiftool` revealed a PNG `tEXt` chunk named `Poem`. The first letter of each line spelled
out **CHECKLSB** — a direct pointer to least-significant-bit steganography.

Pixel values weren't a uniform `(255,0,0,255)` — they varied by ±1 per channel
(e.g. `(254,1,0,254)`), the classic LSB-encoding signature.

Extracted the low bit of every channel (R, G, B, A, in that order) per pixel and packed
the bits into bytes:

```python
from PIL import Image

im = Image.open("red.png").convert("RGBA")
bits = [p[c] & 1 for p in im.getdata() for c in range(4)]   # R,G,B,A per pixel
data = bytes(int("".join(map(str, bits[i:i+8])), 2) for i in range(0, len(bits) - 7, 8))

text = data.decode()
b64 = text[:text.index("==") + 2]   # message repeats/tiles across the image
```

The output was a base64 string (ends in `==`). Decoding it:

```bash
echo 'YWNhZGVteXtyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==' | base64 -d
```

## Flag
`academy{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}`

## Takeaway
When an image "looks" too flat/plain, diff the pixel values against the expected solid colour before assuming there's nothing there.
