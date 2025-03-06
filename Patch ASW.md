FUN_CBOOT__800226ce() - Reads from afe81f84, must be non-zero
FUN_CBOOT__800226dc() - Reads from afe81e45, must be 0xa2
FUN_CBOOT__800226e8() - Reads from afe81e40, must be 0xd01

Place your patch at 0x800C0000
Modify the value at 0x800C0200 to point to 0x800C0000 instead of 0x801AF180
Your patch should end by jumping to 0x801AF180 to continue normal ASW execution

ASW1 CRC-Checksum 0xDB97280B

crchack -x 00000000 -i 00000000 -w 32 -p 0x4c11db7 -b 0x304:0x308 ASW1_modified.bin 0xDB97280B > ASW1_patched.bin

Change ASW Entry
     1::800c0200 00 00 0c 80     ddw        800C0000h

Patch
                             //
                             // ASW1 
                             // ASW1::800c0000-ASW1::8017fbff
                             //
     1::800c0000 91 20 00 c8     movh.a     a12,#0x8002
     1::800c0004 91 00 00 4d     movh.a     a4,#0xd000
     1::800c0008 d9 44 40 10     lea        a4,[a4]0x440
     1::800c000c d9 cf 42 ae     lea        a15,[a12]-0x197e
     1::800c0010 2d 0f 00 00     calli      a15
     1::800c0014 d9 cf 66 2e     lea        a15,[a12]-0x1b5a
     1::800c0018 3b 00 20 41     mov        d4,#0x1200
     1::800c001c 2d 0f 00 00     calli      a15
     1::800c0020 d9 cf a4 5e     lea        a15,[a12]-0x169c
     1::800c0024 3b 00 20 41     mov        d4,#0x1200
     1::800c0028 2d 0f 00 00     calli      a15
     1::800c002c d9 cf 42 9e     lea        a15,[a12]-0x19be
     1::800c0030 0d 00 c0 04     isync
     1::800c0034 91 00 00 fc     movh.a     a15,#0xc000
     1::800c0038 d9 ff 80 c7     lea        a15,[a15]0x7b00
     1::800c003c 80 f0           mov.d      d0,a15
     1::800c003e 0d 00 80 04     dsync
     1::800c0042 cd 40 e2 0f     mtcr       #0xfe24,d0
     1::800c0046 0d 00 c0 04     isync
     1::800c004a 0d 00 c0 04     isync
     1::800c004e d9 cf 58 9e     lea        a15,[a12]-0x19a8
     1::800c0052 2d 0f 00 00     calli      a15
     1::800c0056 91 20 00 28     movh.a     a2,#0x8002
     1::800c005a 49 22 00 0a     lea        a2,[a2]0x0
     1::800c005e 91 00 00 fd     movh.a     a15,#0xd000
     1::800c0062 d9 ff 00 07     lea        a15,[a15]0x7000
     1::800c0066 91 00 00 50     movh.a     a5,#0x0
     1::800c006a d9 55 e0 f1     lea        a5,[a5]0x1fe0
                             LAB_ASW1__800c006e                              XREF[1]:     ASW1::800c0074(j)  
     1::800c006e 09 22 01 00     ld.b       d2,[a2+]0x1
     1::800c0072 24 f2           st.b       [a15+],d2
     1::800c0074 fc 5d           loop       a5,LAB_ASW1__800c006e
     1::800c0076 91 00 00 fd     movh.a     a15,#0xd000
     1::800c007a d9 ff 80 40     lea        a15,[a15]0x900
     1::800c007e 49 ff 20 1a     lea        a15,[a15]0x60
     1::800c0082 49 ff 0c 0a     lea        a15,[a15]0xc
     1::800c0086 49 ff 0e 0a     lea        a15,[a15]0xe
     1::800c008a 82 10           mov        d0,#0x1
     1::800c008c 74 f0           st.w       [a15],d0
     1::800c008e 91 00 00 fd     movh.a     a15,#0xd000
     1::800c0092 d9 ff 80 40     lea        a15,[a15]0x900
     1::800c0096 49 ff 20 1a     lea        a15,[a15]0x60
     1::800c009a 49 ff 0d 0a     lea        a15,[a15]0xd
     1::800c009e 49 ff 0c 0a     lea        a15,[a15]0xc
     1::800c00a2 3b 20 0a 00     mov        d0,#0xa2
     1::800c00a6 74 f0           st.w       [a15],d0
     1::800c00a8 91 00 00 fd     movh.a     a15,#0xd000
     1::800c00ac d9 ff 80 40     lea        a15,[a15]0x900
     1::800c00b0 49 ff 20 1a     lea        a15,[a15]0x60
     1::800c00b4 49 ff 0e 0a     lea        a15,[a15]0xe
     1::800c00b8 49 ff 08 0a     lea        a15,[a15]0x8
     1::800c00bc 3b 10 d0 00     mov        d0,#0xd01
     1::800c00c0 74 f0           st.w       [a15],d0
     1::800c00c2 3b 00 20 41     mov        d4,#0x1200
     1::800c00c6 91 00 00 fd     movh.a     a15,#0xd000
     1::800c00ca d9 ff 00 07     lea        a15,[a15]0x7000
     1::800c00ce 2d 0f 00 00     calli      a15
     1::800c00d2 91 a0 01 f8     movh.a     a15,#0x801a
     1::800c00d6 d9 ff 80 00     lea        a15,[a15]0x800
     1::800c00da d9 ff 40 00     lea        a15,[a15]0x400
     1::800c00de d9 ff 00 c0     lea        a15,[a15]0x300
     1::800c00e2 49 ff 00 2a     lea        a15,[a15]0x80
     1::800c00e6 49 ff 00 4a     lea        a15,[a15]0x100
     1::800c00ea dc 0f           ji         a15
