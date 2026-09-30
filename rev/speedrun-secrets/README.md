# Speedrun Reverse Engineering (20 Binaries, 10s Each)

## Approach
`nc chatelaine.cylabacademy.net 30354` sends a raw hex dump of a small x86-64 ELF binary,
then prompts `What's the secret?:` with a 10-second window to answer. 20 rounds total,
binary changes every round — has to be fully automated.

Disassembling the first binary by hand showed a trivial `scanf("%u")` + `cmp` against a
hardcoded stack value in `main`:

```asm
mov DWORD PTR [rbp-4], 0x378d26ad   ; hardcoded secret
mov DWORD PTR [rbp-8], 0            ; scanf target
call scanf
mov eax, [rbp-8]
cmp  [rbp-4], eax
jne  fail
```

## Solution
Wrote a Python script (stdlib only, no pwntools needed) that:
1. Reads the hex blob from the socket each round.
2. Converts it to raw ELF bytes.
3. Regex-searches for `mov DWORD PTR [rbp-X], imm32` (opcode `C7 45 <disp8> <imm32 LE>`)
   paired with the later `cmp`/`3B`/`39` comparison on the same stack offset.
4. Sends the decimal value back (`%u` requires decimal, not hex).
5. Repeats until the server stops sending binaries.

```python
import socket, re

HOST, PORT = "chatelaine.cylabacademy.net", 30354
s = socket.create_connection((HOST, PORT))
buf = b""

def read_until(marker):
    global buf
    while marker not in buf:
        chunk = s.recv(65536)
        if not chunk:
            break
        buf += chunk
    i = buf.find(marker) + len(marker)
    out, buf = buf[:i], buf[i:]
    return out.decode(errors="replace")

def find_secret(elf):
    movs = {}
    for m in re.finditer(rb"\xc7\x45(.)(....)", elf, re.S):
        v = int.from_bytes(m.group(2), "little")
        movs.setdefault(m.group(1), []).append(v)
    for m in re.finditer(rb"[\x39\x3b]\x45(.)", elf, re.S):
        vals = [v for v in movs.get(m.group(1), []) if v != 0]
        if vals:
            return vals[-1]
    return None

while True:
    data = read_until(b"secret?:")
    m = re.search(r"\b([0-9a-f]{200,})\b", data)
    if not m:
        print(data)   # flag / final message
        break
    elf = bytes.fromhex(m.group(1))
    secret = find_secret(elf)
    s.sendall(f"{secret}\n".encode())
```

**Status: in progress.** Pattern-matching worked for the first binary's `mov`/`cmp` shape;
still confirming it holds for all 20 rounds (some rounds may use a different comparison
instruction, a string compare, or a computed value instead of a flat immediate).

## Flag
_TBD_

## Takeaway
For timed multi-round rev challenges, don't disassemble by hand each round — find the byte pattern once, then pattern-match it in a script.
