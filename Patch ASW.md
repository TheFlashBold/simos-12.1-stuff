# Simos 12.1 Sample Mode Enabler
This patch allows bypassing RSA signature verification on Simos 12.1 ECUs by enabling "Sample Mode" in CBOOT, which disables the cryptographic validation.
This is based on information from https://github.com/bri3d/VW_Flash for Simos 18

## Addresses
0x80000000 are flash addresses, for example 0xC0000 is the location inside the bin.
Look here for details https://github.com/bri3d/VW_Flash/blob/master/lib/modules/simos12.py

## How It Works
The patch has three key components:

- Position: Places code at the beginning of ASW1 (0x800C0000)
- Entry Point: Modifies the ASW entry point to execute our patch first
- Functionality: Loads CBOOT into RAM, patches it to enable Sample Mode, then executes it

When the ECU boots, CBOOT will jump to our patch code instead of the normal ASW entry point. Our code:

- Loads CBOOT into RAM at address 0xD0007000
- Patches the Sample Mode check functions in RAM
- Jumps to the patched CBOOT in RAM
- If it returns, continues to the original ASW entry point

## Sample Mode Check Functions
The patch targets three critical functions in CBOOT that determine if an ECU is in "Sample Mode":

```
Function		Memory Check	Required Value
FUN_CBOOT__800226ce()	afe81f84	non-zero
FUN_CBOOT__800226dc()	afe81e45	0xa2
FUN_CBOOT__800226e8()	afe81e40	0xd01
```

When all three checks pass, CBOOT operates in Sample Mode, which disables signature verification.
Applying the Patch

### 1. Modify the ASW entry point
Change the value at 0x800C0200 from 0x801AF180 to 0x800C0000:
`800c0200: 80f1 1a80  →  800c 0000`

### 2. Add the patch code at 0x800C0000

```
// Set up environment
movh.a     a12, #0x8002       // CBOOT address: 0x80020000
movh.a     a4, #0xd000        // Task context base address
lea        a4, [a4]0x440      // Complete address: 0xd0000440
lea        a15, [a12]-0x197e  // Calculate function address
calli      a15                // Set task context

// Configure timers with Programming Mode parameter
lea        a15, [a12]-0x1b5a  // Timer setup function
mov        d4, #0x1200        // Programming Mode parameter (0x1200)
calli      a15                // Call timer setup

// Reset peripherals 
lea        a15, [a12]-0x169c  // Peripheral reset function
mov        d4, #0x1200        // Programming Mode parameter
calli      a15                // Call reset function

// Enable ENDINIT (CPU protection)
lea        a15, [a12]-0x19be  // ENABLE_ENDINIT function
calli      a15                // Call function

// Configure trap vectors
isync                         // Instruction sync
movh.a     a15, #0xc000       // Base address for trap vectors
lea        a15, [a15]0x7b00   // Trap vector address: 0xc0007b00
mov.d      d0, a15            // Put address in d0
dsync                         // Data sync
mtcr       #0xfe24, d0        // Set trap vector register
isync                         
isync

// Disable ENDINIT (for memory operations)
lea        a15, [a12]-0x19a8  // DISABLE_ENDINIT function
calli      a15                // Call function

// Copy CBOOT from flash to RAM
movh.a     a2, #0x8002        // Source: 0x80020000 (CBOOT)
lea        a2, [a2]0x0        // Complete source address
movh.a     a15, #0xd000       // Destination base
lea        a15, [a15]0x7000   // Complete destination: 0xd0007000
movh.a     a5, #0x0           // Prepare for size
lea        a5, [a5]0x1fe0     // Size: 0x1fe0 (CBOOT size)

// Memory copy loop
copy_loop:
ld.b       d2, [a2+]0x1       // Load byte from flash
st.b       [a15+], d2         // Store byte to RAM
loop       a5, copy_loop      // Repeat until done

// Patch first Sample Mode check (must return non-zero)
movh.a     a15, #0xd000       // RAM base
lea        a15, [a15]0x900    // Building address
lea        a15, [a15]0x60     // Building address
lea        a15, [a15]0xc      // Building address
lea        a15, [a15]0xe      // Final: 0xd00096ce (function in RAM)
mov        d0, #0x1           // Set to non-zero value
st.w       [a15], d0          // Patch function

// Patch second Sample Mode check (must return 0xa2)
movh.a     a15, #0xd000       // RAM base
lea        a15, [a15]0x900    // Building address
lea        a15, [a15]0x60     // Building address
lea        a15, [a15]0xd      // Building address
lea        a15, [a15]0xc      // Final: 0xd00096dc (function in RAM)
mov        d0, #0xa2          // Set to required value
st.w       [a15], d0          // Patch function

// Patch third Sample Mode check (must return 0xd01)
movh.a     a15, #0xd000       // RAM base
lea        a15, [a15]0x900    // Building address
lea        a15, [a15]0x60     // Building address
lea        a15, [a15]0xe      // Building address
lea        a15, [a15]0x8      // Final: 0xd00096e8 (function in RAM)
mov        d0, #0xd01         // Set to required value
st.w       [a15], d0          // Patch function

// Jump to patched CBOOT in RAM
mov        d4, #0x1200        // Programming Mode parameter
movh.a     a15, #0xd000       // RAM address base
lea        a15, [a15]0x7000   // CBOOT in RAM: 0xd0007000
calli      a15                // Jump to CBOOT in RAM

// If CBOOT returns, continue to normal ASW execution
movh.a     a15, #0x801a       // Original entry point high part
lea        a15, [a15]0x800    // Building address
lea        a15, [a15]0x400    // Building address
lea        a15, [a15]0x300    // Building address
lea        a15, [a15]0x80     // Building address
lea        a15, [a15]0x100    // Final: 0x801af180 (original entry)
ji         a15                // Jump to original ASW entry
```

### 3. Update the CRC checksum
The ASW1 block requires valid CRC checksum. Change the value at 0x800C0304:

´´´
0xC0304: 0b28 97db  →  b92a 3ae6
´´´

You can use this Python script to calculate the correct CRC:

```
import struct
import binascii

def crc32_fast(data, poly=0x4C11DB7, init=0, xorout=0):
    # Create CRC32 table
    table = []
    for i in range(256):
        c = i << 24
        for j in range(8):
            c = (c << 1) ^ poly if (c & 0x80000000) else c << 1
        table.append(c & 0xFFFFFFFF)
    
    # Calculate CRC
    crc = init
    for byte in data:
        crc = (crc << 8) ^ table[((crc >> 24) ^ byte) & 0xFF]
        crc &= 0xFFFFFFFF
    
    return crc ^ xorout

# Load modified binary and update checksum
with open("modified_binary.bin", "rb") as f:
    data = bytearray(f.read())

# ASW1 starts at 0xC0000, checksum at 0xC0304
checksum_location = 0xC0300 + 4
checksum_areas = [(0xC0000, 0xC02FF), (0xC0800, 0x17F9FF)]

checksum_data = bytearray()
for start, end in checksum_areas:
    checksum_data.extend(data[start:end+1])

checksum = crc32_fast(checksum_data)
data[checksum_location:checksum_location+4] = struct.pack("<I", checksum)

with open("patched_binary.bin", "wb") as f:
    f.write(data)
```

### Patch Raw Bytes
If you need to apply the patch using a hex editor, here are the raw bytes:

```
91 20 00 c8 91 00 00 4d d9 44 40 10 d9 cf 42 ae
2d 0f 00 00 d9 cf 66 2e 3b 00 20 41 2d 0f 00 00
d9 cf a4 5e 3b 00 20 41 2d 0f 00 00 d9 cf 42 9e
0d 00 c0 04 91 00 00 fc d9 ff 80 c7 80 f0 0d 00
80 04 cd 40 e2 0f 0d 00 c0 04 0d 00 c0 04 d9 cf
58 9e 2d 0f 00 00 91 20 00 28 49 22 00 0a 91 00
00 fd d9 ff 00 07 91 00 00 50 d9 55 e0 f1 09 22
01 00 24 f2 fc 5d 91 00 00 fd d9 ff 80 40 49 ff
20 1a 49 ff 0c 0a 49 ff 0e 0a 82 10 74 f0 91 00
00 fd d9 ff 80 40 49 ff 20 1a 49 ff 0d 0a 49 ff
0c 0a 3b 20 0a 00 74 f0 91 00 00 fd d9 ff 80 40
49 ff 20 1a 49 ff 0e 0a 49 ff 08 0a 3b 10 d0 00
74 f0 3b 00 20 41 91 00 00 fd d9 ff 00 07 2d 0f
00 00 91 a0 01 f8 d9 ff 80 00 d9 ff 40 00 d9 ff
00 c0 49 ff 00 2a 49 ff 00 4a dc 0f
```
