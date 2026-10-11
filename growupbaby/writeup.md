---
title: "growupbaby"
ctf: "GoPWNIt"
date: 2026-10-10
category: reverse
difficulty: medium
points: 100
flag_format: "GPI{...}"
author: "nomadrai"
---

# growupbaby

Challenge author: **bhaiya_ji**.

## Summary

The attachment contains `memo.exe`, a packed 64-bit Windows crackme with virtualized loader/import code, anti-analysis worker threads, and mixed Boolean-arithmetic obfuscation. Its password ultimately decrypts with a bitwise NOT followed by XOR: **`patrobicha`**.

## Solution

### 1. Unpack and isolate the password routine

Extract `1791623842524-gpi.zip`. The executable's preferred image base is `0x140000000`, and its packed entry point is `0x140a2193c`. The original sections have no raw file contents, so static strings do not expose the password.

Run under Wine 8.0 and GDB 13.1 using the included `unpack.gdb` and `launch_gdb.py`. The debugger script:

- Bypasses Wine's unimplemented `RtlQueryProcessHeapInformation` function.
- Delivers the loader's intentional trap-flag exceptions to its handlers.
- Intercepts thread creation and suppresses the timing/integrity watchdogs while retaining the password worker at `0x1400107f0`.
- Supplies the candidate to the input string and lets the original length check and byte comparison run.
- Dumps the loaded image to `unpacked.bin` and records execution in `runtime.log`.

The password initialization contains these 64-bit ciphertext words:

| Instruction RVA | Ciphertext |
|---|---|
| `0x128ff` | `0x024d7afa0236cba0` |
| `0x12918` | `0x898f11fc896c95a3` |

The two XOR keys are `0x9edbe76a8fbd552f` and `0x7670ee0376930b34`, loaded at RVAs `0x10877` and `0x1088d`. Unicorn emulation of the obfuscated helper at RVA `0x15f30` produces the 64-bit complement of each ciphertext word. The subsequent obfuscated loop XORs with the corresponding key.

### 2. Recover the password

This complete, dependency-free decoder uses the constants recovered above:

```python
import struct

ciphertext = (0x024D7AFA0236CBA0, 0x898F11FC896C95A3)
keys = (0x9EDBE76A8FBD552F, 0x7670EE0376930B34)
mask = (1 << 64) - 1

plaintext = b"".join(
    struct.pack("<Q", ((~word) & mask) ^ key)
    for word, key in zip(ciphertext, keys)
)
password = plaintext.split(b"\0", 1)[0].decode("ascii")
print(f"Password: {password}")
print(f"Flag: GPI{{{password}}}")
```

The saved solver can also extract the constants directly from the raw memory image:

```bash
python3 solve.py --image unpacked.bin
```

Output:

```text
Password: patrobicha
Flag: GPI{patrobicha}
```

### 3. Verify the original comparison

The native password routine checks string lengths at `0x140011eca`, invokes its comparison at `0x140011ef0`, and branches to the success path when the result is zero. Supplying `patrobicha` reached `0x1400128c1`, immediately after the success-message decoder returned:

```text
Supplying password: patrobicha
Thread 7 "memo.exe" hit Breakpoint 2, 0x00000001400128c1
0x140133160: "wtf how did you crack this\n"
```

For this verification, the debugger populated the input `std::string` at the console-read boundary because console interaction under headless Wine was unreliable. The length check, comparison result, and success branch were executed normally. The binary validates the bare password; the submission uses the challenge's `GPI{...}` wrapper. Competition-server acceptance has not been tested.

## Flag

```text
GPI{patrobicha}
```
