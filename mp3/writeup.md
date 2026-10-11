---
title: "mp3"
ctf: "GoPWNIt CTF 2026"
date: 2026-10-10
category: forensics
platform_category: "Steganography"
difficulty: medium
points: 100
flag_format: "FLAG{...}"
author: "nomadrai"
---

# mp3

## Summary

The intercepted Galvan Prime transmission embeds a diagnostic payload and
encryption-key fragments in audio metadata and signal channels. The most
probable process combines metadata inspection, stereo separation, and
spectrogram analysis before decrypting the concealed payload.

## Solution — probable approach

The retained artifacts are `ben10_omnitrix_signal.mp3`, its original ZIP, and
`spec_left.png`, `spec_right.png`, and `spec_diff.png`. They establish that
channel and difference-spectrum inspection was part of the local work; no
final key, decoder, or output transcript is retained.

### 1. Recover metadata instructions

Inspect the MP3's ID3 text and comment frames with ExifTool or FFprobe. Treat
encoded comments as possible key fragments or instructions, preserving their
order and exact capitalization. Follow the metadata's indicated encoding
rather than assuming it is itself the complete flag.

### 2. Isolate the hidden stereo clues

Decode to lossless PCM while preserving both channels. Compare left, right,
and left-minus-right spectrograms; subtraction can suppress common audio and
make an opposite-phase or channel-specific signal visible. Use the resulting
text or timing clues to identify the payload carrier and key-fragment order.

The saved filenames support this channel-comparison route. They do not
establish a carrier frequency, symbol rate, or particular cipher, so none is
asserted here.

### 3. Reconstruct the key and open the diagnostic payload

Combine the fragments according to the recovered instructions, extract the
payload bytes in their specified order, and apply the indicated decryption
scheme. Check the scheme's authentication tag or checksum where provided,
then preserve the exact `FLAG{...}` plaintext.

## Tools and lesson

Probable tools: ExifTool, FFmpeg/FFprobe, Audacity or a Python spectrogram
workflow. Downmixing stereo prematurely can destroy the channel relationship
that carries the key information.

## Flag / key

**Submission note (participant-provided):** The flag/key was not saved at the
moment of submission due to hurry.
