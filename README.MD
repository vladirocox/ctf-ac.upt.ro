# $ P3sk0b@r - Patrick, Vlad, Andrei
- 5th place

## Writeups

### Crypto

- Repeated RSA

- SSS

- Neverending Randomness

- XORBITANT

- Baby_crib

### Forensics

- Unknown-traffic1

- Unknown-traffic2

- Hidden in the cartige

- Baofeng

- 3-rd child

- Anticheat

### Reverse engineering

- Pythonese

- Ironveil

### Misc

- octojail

- onions1

- onions2

- disco_dance

- disco_rave

- discord

### web

- money

- dot-private-key

- random-gallery

- theme-generator

### Pwn

- Sigdance

- Baby-BOF

- Fini

- Minecrafty

### Osint

- Holiday Trip

- Prison

- Ancient_house

## --------------------------------------------------

### Discord
- The flag_channel does not actually contain the flag. Analyzing the food recipes will just make you hungry.
- Opening the sticker panel, we are instantly greeted with the CTF@AC Logo, with the qr code visually hidden.
- (qr photo)
- Vencord (custom discord client) let me instantly get the bigger picture, which I put a grayscale filter over, so that the picture would scan easier.
- (scanned)
- The link redirects you to example.com, but, inside the link, there is a hidden parameter stating the flag.
- (flag and photo)

### disco_dance
- The discord series was my favourite type of challanges.
- They were so well thought and engineered, and actually made me smile.
- Regarding disco_dance, the server sends an encrypted text, which can only be decoded using the contents of the last 5 messages in the "spam" channel.
- ChatGPT and I kinda underengineered it, as the script lacks the ability to automatically fetch the contents using the discord api. 
- This would have helped me get a first blood easily.
- Thankfully, my coleague, which I thank a lot, filled the chat with "a" s, which made it impressively easy for me and the script to decode the flag.
- Here is the script I used to decrypt the flag:
- (script script)
- (flag)

### disco_rave
- Another congrats to the organisers and challange writers, as this one was my favourite.
- It took me a while to find the bug in my code, which caused error 401 Unauthorized when trying to fetch data using the discord api.
- The challange works relatively the same. This time, it fetches the last messages ("content" and "timestamp") from the both spam channels. 
- This time, using an automatically message fetcher was necessary, as the whole chat was going crazy, and getting the exact timestamp manually would take too long. Even longer than actually making an automatic message fetcher.
- Long story short, using a similar script, this time automatically fetching every piece of information needed, I instantly got the flag.
- (script, flag)

### ancient_house
- This challange may seem difficult at first sight, but it actually uncovers a simple solve.
- Downloading the picture given and asking chatgpt to repair it according to the text, we get a clean picture, which google lens instantly recognises and gives us the flag at first sight.
- (chatgpt convo, google lens)
- CTF{Dumbrava}

### 3rd_child
- By playing the audio file, I instantly recognise the familiar song name, but also an audio pattern which surely states CTF.
- The rest of the flag/ sha256, is being overlapped by the song.
- Using audacity, I could see 3 tracks. I enabled the "show ......." option, and finally, the last track displayed the flag.
- (poza flag)  

### onions1
- The easiest challange. Easier than "feedback".
- The long "hashy" link and the ".onion" extension made me open tor browser in a split second.
- Opening the website, the flag was shown.
- (ipad photo)
- ctf{fc5b4974d9c262a40810686f06dc5525b046a810ad3f8cd20d62f4d2f6379dbd}

### Escaping Barcelona
- My qwant and google search queries were really bad, not finding any posibile solution to this challange.
- Grok, though, was ready to help me.
- Using a new library, that I have never heard of, the distance was easily calculated.
- (script)
- (flag)
