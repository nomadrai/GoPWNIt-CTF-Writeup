---
title: "Phantom Hands"
ctf: "GoPWNIt"
date: 2026-10-10
category: forensics
difficulty: easy
points: 50
flag_format: "GPI{...}"
author: "nomadrai"
---

# Phantom Hands

## Summary

The USB capture contains a keyboard-layout switch, an edited decryption command, and a mouse-drawn passphrase. Reconstructing Dvorak keystrokes and rendering left-button mouse movements recovers the inputs needed to decrypt the authenticated flag.

## Solution

### 1. Reconstruct the keyboard command

The attachment contains `dist/usb.pcap`, a little-endian PCAP with Linux USB mmap encapsulation (link type 220). Each USB packet has a 64-byte header. The 2,158 packets span approximately 60.73 seconds.

| Device | Reports | Report size | Interpretation |
| --- | ---: | ---: | --- |
| 3 | 496 | 8 bytes | Main keyboard |
| 7 | 88 | 8 bytes | Second keyboard, decoy flag |
| 5 | 1,574 | 4 bytes | Mouse |

For device 3, decode HID key-down transitions, initially using QWERTY. After `setxkbmap dvorak`, switch the physical-key mapping to Dvorak. Honor Backspace, Left Arrow, and End; otherwise both the script name and ciphertext are reconstructed incorrectly.

The resulting command is:

```text
python3 unseal.py 626822313c1f6196c622999c35fb4dde83aababf62c901ddfe04682138cbb223f8a2362ed9cd54117456fa9650137cdad9a2d6862f881c600457179f7d561d148b6c0ebd060fae3dbc
```

The earlier `echo GPI{...}` and the flag typed by device 7 are decoys.

### 2. Render the mouse and decrypt

Device 5's reports are `(buttons, signed dx, signed dy, wheel)`. Accumulate **all** relative movements, but draw segments only while `buttons & 1` is set. This produces:

![Mouse-drawn passphrase](mouse.png)

The passphrase reads **`TALLOW-RIVET-83`**.

Although `unseal.py` is not supplied, the 73-byte blob successfully decrypts with:

- Key: `SHA256(b"TALLOW-RIVET-83")`, the raw 32-byte digest.
- Cipher: AES-256-GCM.
- Nonce: first 12 bytes of the blob.
- Ciphertext: the next 45 bytes.
- Authentication tag: final 16 bytes.
- Associated data: empty.

The GCM authentication tag verifies, confirming the recovered key and ciphertext.

### 3. Reproduce

The full [solve.py](solve.py) reads the original ZIP, reconstructs both keyboard transcripts, saves `mouse.png`, and prints the authenticated flag. Dependencies: Python 3, Pillow, and cryptography; verified with Python 3.11.2, Pillow 12.3.0, and cryptography 50.0.2.

From the project root, with those dependencies installed:

```bash
python3 "Phantom Hands/solve.py"
```

This compact standalone version also runs from the challenge folder. The passphrase is transcribed from the rendered image:

```python
import hashlib
import struct
import zipfile
from PIL import Image, ImageDraw
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

with zipfile.ZipFile('1791636351672-dist.zip') as z:
    raw = z.read('dist/usb.pcap')
assert raw[:4] == bytes.fromhex('d4c3b2a1')
assert struct.unpack_from('<I', raw, 20)[0] == 220
reports, pos = [], 24
while pos < len(raw):
    size = struct.unpack_from('<I', raw, pos + 8)[0]
    packet = raw[pos + 16:pos + 16 + size]
    pos += 16 + size
    if packet[8:11] == b'C\x01\x81':
        length = struct.unpack_from('<I', packet, 36)[0]
        reports.append((packet[11], packet[64:64 + length]))

keys = dict(enumerate('abcdefghijklmnopqrstuvwxyz', 4))
keys.update(dict(enumerate('1234567890', 30)))
keys.update({44: ' '})
keys.update(dict(enumerate("-=[]\\#;'`,./", 45)))
dv = dict(zip("qwertyuiop[]asdfghjkl;'zxcvbnm,./-=",
              "',.pyfgcrl/=aoeuidhtns-;qjkxbmwvz[]"))
shift = dict(zip("1234567890-=[]\\;'`,./", '!@#$%^&*()_+{}|:"~<>?'))
line, cursor, previous, dvorak = [], 0, set(), False
commands = []
for device, report in reports:
    if device != 3:
        continue
    current = set(report[2:]) - {0}
    for key in report[2:]:
        if not key or key in previous:
            continue
        if key == 40:
            command = ''.join(line)
            commands.append(command)
            dvorak |= command == 'setxkbmap dvorak'
            line, cursor = [], 0
        elif key == 42:
            if cursor:
                cursor -= 1
                del line[cursor]
        elif key == 80:
            cursor = max(0, cursor - 1)
        elif key == 77:
            cursor = len(line)
        else:
            char = keys[key]
            if dvorak:
                char = dv.get(char, char)
            if report[0] & 0x22:
                char = shift.get(char, char.upper())
            line.insert(cursor, char)
            cursor += 1
    previous = current

image = Image.new('RGB', (2400, 300), 'white')
draw = ImageDraw.Draw(image)
x = y = 0
for device, report in reports:
    if device != 5:
        continue
    buttons, dx, dy, wheel = struct.unpack('<Bbbb', report)
    nx, ny = x + dx, y + dy
    if buttons & 1:
        draw.line((x * 3, y * 3, nx * 3, ny * 3), fill='black', width=3)
    x, y = nx, ny
image.save('mouse.png')

command = next(c for c in commands if c.startswith('python3 unseal.py '))
blob = bytes.fromhex(command.split()[-1])
key = hashlib.sha256(b'TALLOW-RIVET-83').digest()
print(AESGCM(key).decrypt(blob[:12], blob[12:], None).decode())
```

Output:

```text
GPI{dv0r4k_h4nds_4nd_4_m0us3_th4t_dr4ws_60c9}
```

## Flag

```text
GPI{dv0r4k_h4nds_4nd_4_m0us3_th4t_dr4ws_60c9}
```
