---
title: "mp4"
ctf: "GoPWNIt (workspace label)"
date: 2026-10-10
category: forensics
difficulty: medium
points: 100
flag_format: "FLAG{...}"
author: "nomadrai"
---

# mp4

Challenge author: **bhaiya_ji**. Platform category: **Steganography**.

## Summary

The recording hides a chain of clues in metadata, a high-frequency spectrogram,
and right-channel Morse code. The final payload is two-frequency binary audio
encoding a Base64 flag.

## Solution

### Step 1: Read the metadata and spectrogram

The attachment `1791623409651-player_package.zip` contains
`last_transmission.mp4`. FFprobe identifies one 44,100 Hz stereo AAC stream,
approximately 19.66 seconds long, with no video stream.

```bash
unzip -n 1791623409651-player_package.zip
exiftool -a -u -g1 last_transmission.mp4
ffprobe -v error -show_format -show_streams last_transmission.mp4
```

The encoder, comment, and description fields contain:

```text
7363616e20746865207370656374726f6772616d
```

Hex decoding gives **`scan the spectrogram`**. A spectrogram using a 4096-sample
window and 128-sample hop reveals **`SECOND_CHANNEL`** around 14–16 kHz.

![Right-channel spectrogram](spectrogram_right.png)

### Step 2: Decode the second-channel Morse clue

The right channel contains on/off pulses at **2400 Hz**. Mixing this tone down
to zero frequency and low-pass filtering at 45 Hz reveals the pulse envelope.
Dots last approximately 0.15 seconds; dashes last approximately 0.45 seconds.

The first complete sequence, between about 1.5 and 13.05 seconds, is:

```text
.---- ----. ..--- -----
1     9     2     0
```

This identifies the **1920 Hz** carrier. Inspecting that region reveals two
alternating frequencies, 1920 Hz and 2000 Hz, with a **50 ms symbol period**
(20 bits per second).

### Step 3: Recover the binary packet and decode Base64

The packet spans **1.0–15.8 seconds** and contains **296 bits / 37 bytes**.
Use **1920 Hz = 0**, **2000 Hz = 1**, and pack bits most-significant first.
The first byte is an alternating-bit preamble, `0xAA`; the remaining 36 bytes
are Base64 text. Both stereo channels independently decode to the same packet.

The complete solver is [solve.py](solve.py). It accepts either the original ZIP
or the extracted MP4, checks the metadata, compares both channel decodes, and
prints the flag. It requires FFmpeg/FFprobe and Python with NumPy.

```bash
python3 solve.py 1791623409651-player_package.zip
```

For a minimal reproduction, run this complete script from the `mp4` folder:

```python
import base64
from pathlib import Path
import subprocess
import tempfile
import zipfile
import numpy as np

with tempfile.TemporaryDirectory() as temp:
    media = Path(temp) / 'last_transmission.mp4'
    with zipfile.ZipFile('1791623409651-player_package.zip') as archive:
        media.write_bytes(archive.read('last_transmission.mp4'))
    raw = subprocess.check_output([
        'ffmpeg', '-v', 'error', '-i', str(media), '-map', '0:a:0',
        '-f', 'f32le', '-ar', '44100', '-ac', '2', 'pipe:1'
    ])

audio = np.frombuffer(raw, dtype='<f4').reshape(-1, 2)
t = np.arange(2205) / 44100
detectors = np.exp(-2j*np.pi*np.array([1920, 2000])[:, None]*t)
packets = []
for channel in range(2):
    blocks = audio[44100:44100+296*2205, channel].reshape(296, 2205)
    power = np.abs((blocks*np.hanning(2205)) @ detectors.T)**2
    bits = (power[:, 1] > power[:, 0]).astype(np.uint8)
    packets.append(np.packbits(bits, bitorder='big').tobytes())
assert packets[0] == packets[1]
assert packets[0][0] == 0xAA
print(base64.b64decode(packets[0][1:], validate=True).decode())
```

Verified output from `solve.py`:

```text
Metadata hint: scan the spectrogram
Channel 1 packet: b'\xaaRkxBR3t0aGVfbGFzdF90cmFuc21pc3Npb259'
Channel 2 packet: b'\xaaRkxBR3t0aGVfbGFzdF90cmFuc21pc3Npb259'
FLAG{the_last_transmission}
```

Tools used: ExifTool 12.57, FFmpeg/FFprobe 5.1.9, Python 3.11, NumPy, SciPy,
and Matplotlib. The analysis scripts and spectrum/envelope images are saved
beside this writeup.

## Flag

```text
FLAG{the_last_transmission}
```
