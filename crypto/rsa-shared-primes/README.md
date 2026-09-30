# RSA Weak Key Generation

## Approach
Given `encrypt.py`, which generates a 1024-bit RSA key using a `get_primes()` function
from an unprovided `setup.py`. The service (`nc chatelaine.cylabacademy.net 44481`)
returns a fresh `N`, `e`, and ciphertext on every connection.

Hints pointed at weak/reused randomness in prime generation:
1. "How much do we trust randomness?"
2. "Notice anything interesting about N?"
3. "Try comparing N across multiple requests"

Initial plan was to connect multiple times, collect several `N` values, and look for a
shared prime factor via `gcd(N1, N2)` — a classic weak-RNG RSA attack.

## Solution
Before even needing multiple samples, the first `N` collected was already even
(`N mod 2 == 0`), meaning one of its two "random" primes was literally `2`. That
collapses RSA down to a trivial break:

```python
from Crypto.Util.number import inverse, long_to_bytes, isPrime

N = 21861290959278236718352269369884653011227733255958669650931792868666253249974821894032148235594438609407575128174284798839263663845851366752966843731865474
c = 12454826919557248589578805889678719023922654773351104944386237513008270528937939224651510994096191399571624255259960748533282370013863364352430837525051435
e = 65537

q = N // 2
assert isPrime(q)
d = inverse(e, q - 1)          # phi(N) = (2-1)(q-1) = q-1
m = pow(c, d, q)               # decrypt mod q since plaintext < q
print(long_to_bytes(m))
```

For the general case (when `N` isn't conveniently even), the fallback is to collect
several `(N, ciphertext)` pairs across separate connections and check every pair for a
shared factor:

```python
from math import gcd
from itertools import combinations
from Crypto.Util.number import inverse, long_to_bytes

e = 65537
samples = [(N1, c1), (N2, c2), (N3, c3)]  # collected from separate nc sessions

for (N1, c1), (N2, c2) in combinations(samples, 2):
    p = gcd(N1, N2)
    if p in (1, N1) or N1 == N2:
        continue
    for N, c in ((N1, c1), (N2, c2)):
        q = N // p
        d = inverse(e, (p - 1) * (q - 1))
        print(long_to_bytes(pow(c, d, N)))
```

## Flag
`academy{tw0_1$_pr!m3cfb893a9}`

## Takeaway
Always sanity-check `N` (parity, small-prime trial division, gcd across samples) before reaching for a heavier factoring attack.
