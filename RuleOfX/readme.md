# rule of x
## Author: Stancium
### Description: The rule of X reigns supreme. Here's an equation to solve for X: "Never mind about your belongings, because the best of all Egypt will be yours." + "falò delle vanità" == Princeton + Harvard.


- This was a bit of a challenge, because the info was a bit misleading so we didn't know what to do.


- We connected to the machine to see what is it about
- ![Menu](https://i.imgur.com/ytyMjnz.png)
- Ok, so we have a hashed type of a challenge and we can access the algorithm to try to break it.
- ![Story](https://i.imgur.com/X8Sxt47.png)
- ![Cipher](https://i.imgur.com/l7oPYvH.png)
- First we need to understand how it works. Let's try to type ``AAAA`` and ``AAAAB`` to see if has some common letters in the hash
- ![bytes](https://i.imgur.com/PgVK7kj.png)
```
83cf568c = AAAA                  42c01d91 = Padding
83cf568c = AAAA                  1e82cd7d = Padding
```
- Bingo! It does! The first 8 bytes are the same. So now we need to create a script that connects to the machine and cipher a word of 4 letters and try to find each hash into the ``story + flag`` hash. (what we used [wordlist](https://github.com/dwyl/english-words/blob/master/words.txt))

```
from pwn import *
import time
import json

context.log_level = 'critical'

HOST = "HOST"
PORT = 1234
FILE = "words.txt"
LOOKUP_FILE = "lookup.json"

StoryAndFlag = ""
BlocksStoryAndFlag = {}
Lookup = {}
def SaveJson():
    with open(LOOKUP_FILE, 'w', encoding='utf-8') as f:
        json.dump(Lookup, f, ensure_ascii=False, indent=4)

def SplitInBlocks():
    global BlocksStoryAndFlag
    Blocks = int(len(StoryAndFlag) / 8)
    CurrentBlock = 0
    while CurrentBlock < Blocks:
        tmp = ""
        for i in range(CurrentBlock * 8, (CurrentBlock + 1) * 8):
            tmp += StoryAndFlag[i]
        BlocksStoryAndFlag[tmp] = "[?]"
        CurrentBlock += 1

def LoadJson():
    global Lookup
    with open(LOOKUP_FILE, 'r', encoding='utf-8') as f:
        Lookup = json.loads(f.read())

def LoadWords():
    with open(FILE, 'r') as file:
        for line in file:
            try:
                if Lookup[line.strip()]:
                  continue  
            except Exception:
                Lookup[line.strip()] = EncryptText(line.strip())
                print(f"Encrypting: {line.strip()} Result: {Lookup[line.strip()]}")
            
            

def ReadStoryAndFlag():
    global StoryAndFlag
    p = remote(HOST, PORT)
    p.recvuntil(b"> ")
    p.sendline(b"1")
    time.sleep(1)
    p.recvline()
    StoryAndFlag = p.recvline().decode().rstrip("\n")[:-8]
    p.close()
    SplitInBlocks()

def EncryptText(text):
    p = remote(HOST, PORT)
    p.recvuntil(b"> ")
    p.sendline(b"2")
    p.recvuntil(b"> ")
    p.sendline(text.encode())
    p.recvuntil(b":\n")
    BlockCipher = p.recvline().decode().rstrip("\n")
    p.close()
    return BlockCipher[:-8]

def Decrypt():
    for Hash in Lookup:
        try:
            if BlocksStoryAndFlag[Lookup[Hash]]:
                BlocksStoryAndFlag[Lookup[Hash]] = Hash
        except Exception:
            continue

def ShowStoryAndFlag():
    for Hash in BlocksStoryAndFlag:
        print(f"{BlocksStoryAndFlag[Hash]}", end='')

def main():
    ReadStoryAndFlag()
    LoadJson()
    LoadWords()
    Decrypt()
    ShowStoryAndFlag()
    SaveJson()
    return 0

if __name__ == "__main__":
    main()

```

- So we manage to get a ``python`` script working to cipher and search for each block into the ``story + flag``. After finding a match is going to save the match into a ``json`` called ``lookup.json``(to save the progress).

- So far, so good, we manage to get the ``story`` which was [Pandora's Box](https://www.goodreads.com/quotes/138315-hope-which-whispered-from-pandora-s-box-after-all-the-other)

- After, we added the story in our wordlist for further solves.

```
Decryption result (unknowns shown as [?]):
Hope[?] whi[?]hispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivion... It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the yardstick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our year[?][?][?][?][?][?][?][?][?][?][?][?][?][?][?][?][?][?][?]... whic[?]ispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivi[?]. It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the y[?]tick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our years[?]


Notes: 
- [?] = unknown word
```


- Ok, we didn't panic yet even tho we can see the whole story, but is missing only the flag. We tried to add words like ``CTF``, ``CTF{`` to check if the flag has the format or not. Nothing worked...
- We were so confused why wasn't working until we remembered is a block cipher which is on 4 blocks. So, we devided each of the words found in the solve to see what is the start of the flag and the end are missing. Ex: ``s.CT``, ``ope,``
- Jackpot!
```
Decryption result (unknowns shown as [?]):
Hope[?] whi[?]hispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivion... It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the yardstick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our years.CT[?][?][?][?][?][?][?][?][?][?][?][?][?][?][?][?][?]ope,... whic[?]ispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivi[?]. It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the y[?]tick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our years[?]
```

- Now we are getting somewhere. 
- Let's try to find the beginning by adding ``F{aa``, ``F{bb``, ``etc`` into our wordlist and we can do the same to the end of the flag (but add an extra ``H`` at the end after the ``}`` ``Ex: aa}H``) aswell.

```
Decryption result (unknowns shown as [?]):
Hope[?] whi[?]hispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivion... It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the yardstick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our years.CTF{99[?][?][?][?][?][?][?][?][?][?][?][?][?][?][?]52}Hope,... whic[?]ispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivi[?]. It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the y[?]tick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our years[?]
``` 

- Nice! Now is going to be the tricky part. We think is an easier way, but we just create a wordlist full of 4 letters words like ``aaaa`` till ``9999`` to guess the hash, is a bit of an overkill, but we didn't had any other options at that time.

- Annnnd we got it! 

```
Decryption result (unknowns shown as [?]):
Hope[?] whi[?]hispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivion... It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the yardstick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our years.CTF{99644e931be2c1c8abfa8a74f36544677b77b9ef5f894f760b8b6f5c128cfc52}Hope,... whic[?]ispered from Pandora's box after all the other plagues and sorrows had escaped, is the best and last of all things. Without it, there is only time. And time pushes at our backs like a centrifuge, forcing outward and away, until it nudges us into oblivi[?]. It's a law of motion, a fact of physics..., no different from the stages of white dwarves and red giants. Like all things in the universe, we are destined from birth to diverge. Time is simply the y[?]tick of our separation. If we are particles in a sea of distance, exploded from an original whole, then there is a science to our solitude. We are lonely in proportion to our years[?]
```


- Flag:
```auss
CTF{99644e931be2c1c8abfa8a74f36544677b77b9ef5f894f760b8b6f5c128cfc52}
```


- Note: We could've decipher the flag and leave the rest of the story blank, but where would've been the fun, right?
