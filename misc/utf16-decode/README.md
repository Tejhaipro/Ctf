# 16 Bits Instead of 8

## Approach
Given a short block of CJK-looking characters:

```
慣慤敭祻ㄶ形楴獟楮獴㌴摟潦弸強㤰扡㌷敽
```

No hints were given for this one. Random Chinese/Japanese-looking characters coming from
what's obviously meant to be an ASCII flag is a strong tell that the text is ASCII bytes
that got paired up and misread as UTF-16 (2 bytes per character instead of 1).

## Solution
Re-encoded the given string as UTF-16 (both byte orders) and decoded the resulting bytes
as ASCII:

```python
s = '慣慤敭祻ㄶ形楴獟楮獴㌴摟潦弸強㤰扡㌷敽'
print(s.encode('utf-16-le'))   # byte-swapped, wrong
print(s.encode('utf-16-be'))   # correct
```

- `utf-16-le` → `cadame{y61b_ti_snits43_dfo8_7_09ab73}e` (letter-pairs swapped, wrong order)
- `utf-16-be` → `academy{16_bits_inst34d_of_8_790ba37e}` (correct)

## Flag
`academy{16_bits_inst34d_of_8_790ba37e}`

## Takeaway
Unexpected CJK characters in a "text" challenge is almost always an ASCII string misread as UTF-16 — try re-encoding both byte orders.
