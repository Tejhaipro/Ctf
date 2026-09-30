# Unknown Reality

## Approach
Given `ukn_reality.jpg`, a photo with the flavor text "How about some hide and seek?"
Hints:
1. "How can you view the information about the picture?" — metadata.
2. "If something isn't in the expected form, maybe it deserves attention?" — a metadata
   field holding data of the wrong type/shape.

## Solution
Ran `exiftool` / `strings` against the JPEG and found an embedded Adobe XMP block. Inside
it, the `cc:attributionURL` field — which is supposed to hold a normal web URL — instead
contained a base64-looking blob:

```
<cc:attributionURL rdf:resource='YWNhZGVteXtNRTc0RDQ3QV9ISUREM05fYTgxZDk0MzF9Cg=='/>
```

The `==` padding and character set were the "not in the expected form" clue. Decoded it
directly:

```bash
echo 'YWNhZGVteXtNRTc0RDQ3QV9ISUREM05fYTgxZDk0MzF9Cg==' | base64 -d
```

## Flag
`academy{ME74D47A_HIDD3N_a81d9431}`

## Takeaway
Don't just grep metadata for the flag format — read every field and check whether its *value* actually matches its *expected type* (URL, date, GPS, etc.).
