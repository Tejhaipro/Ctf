# CTF Writeups

Category-wise writeups for CylabAcademy challenges.

## Index

| Category  | Challenge                                              | Status |
|-----------|---------------------------------------------------------|--------|
| Crypto    | [RSA Weak Key Generation](crypto/rsa-shared-primes/)     | Solved |
| Stego     | [RED, RED, RED, RED](stego/red-lsb/)                     | Solved |
| Forensics | [Unknown Reality](forensics/ukn-reality-metadata/)        | Solved |
| Misc      | [16 Bits Instead of 8](misc/utf16-decode/)                | Solved |
| Rev       | [Speedrun Reverse Engineering](rev/speedrun-secrets/)      | In progress |
| Pwn       | [Read The Flag](pwn/xebec-privesc/)                        | In progress |

## Structure

```
writeups/
├── crypto/
│   └── rsa-shared-primes/
├── forensics/
│   └── ukn-reality-metadata/
├── misc/
│   └── utf16-decode/
├── pwn/
│   └── xebec-privesc/
├── rev/
│   └── speedrun-secrets/
└── stego/
    └── red-lsb/
```

Each folder contains a `README.md` in the format:

```
# Challenge Name

## Approach
## Solution
## Flag
## Takeaway
```
