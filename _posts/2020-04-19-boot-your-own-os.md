---
layout: post
title: Boot your own OS
---

This post is a tutorial to understand the internals of booting, and to boot our own piece of code.

The code we are going to boot is very simple.
```nasm
jmp $
```

jmp $ is the simplest OS anyone can write. This instruction instructs the CPU to jump back to the same instruction, effectively making it an infinite loop.

$ is a special token in nasm that refers to the address at the beginning of the line. So the above code is effectively same as:
```nasm
label: jmp label
```

Before we can make this bootable, we first need to understand how the booting process occurs.

I'm going to throw a couple of boot related jargons.

<dl>
<dt>Firmware</dt>
<dd>Firmware is the piece of code that initializes hardware.</dd>

<dt>Reset Vector</dt>
<dd>Reset Vector is the memory location which is used by CPU to execute instructions after power on/reset.</dd>

<dt>BIOS</dt>
<dd>Acronymn for Basic Input/Output System. This is a firmware standard specifically adopted for PCs and has been in use since the 90s.
As a firmware, BIOS initializes hardware like mouse, keyboard, monitor, RAM, etc, performs few checks (POST - power on self test),
and provides a set of services to the bootloader/OS.</dd>

<dt>MBR</dt>
<dd>Short for Master Boot Record. MBR refers to the first 512 bytes of a device and is an important entity for the BIOS. MBR serves two purpose - provide the bootloader code and provide the partition table of the device</dd>

<dt>Bootloader</dt>
<dd>
Also called as loader sometimes. Bootloader refers to the code invoked by BIOS. The bootloader code is responsible for loading
the main OS like Windows or Linux. In this post, we are going to code the infinite loop in bootloader part and boot it.
</dd>
</dl>

* * *

Now lets understand what exactly happens when we power on the system.

### Step 1
Processor starts in real mode, fetches instruction from memory located at reset vector and starts executing it.
Reset vector corresponds to physical memory address ```0xFFFF0```. The ROM chip containing BIOS code will be mapped to this address.

Note: This is specific for PC (x86_64 aka amd64 processor), for others devices like android (usually arm),
it will be different according to that processor.

### Step 2
The BIOS code is fetched and executed. BIOS will initialize the hardware, and will start looking for valid MBR in the order of boot device priority.
A valid MBR is marked by the two magic bytes ```AA``` ```55``` at its end.

### Step 3
Once a valid MBR is found, the BIOS copies the MBR (512 bytes) into address ```0x7C00h``` and then transfers execution to this address.

<br/>

Note: If you can't understand this part properly, you can try brushing up on the following:
> x86 memory map, ISA, Addressing modes

[This](https://www.tutorialspoint.com/microprocessor/microprocessor_8086_overview.htm) is a good place to start.

* * *

Now lets get cooking. Its time to build our code.
The points to note are
1. Processor is in real mode
2. We need to create a valid MBR

We can instruct nasm to generate code for real mode with the following directive
```nasm
bits 16
```

For a valid MBR, we need to package our ```jmp $``` to occupy 512 bytes and to contain magic bytes ```55AA``` at the end. To do that, we will pad null bytes (zeros) to fill up to 510 bytes. And in the end we will add 55 & AA.

To pad with null for upto 510 bytes, we will use
```nasm
times 510-($-$$) db 0
```
times is a assembler directive that repeats the line given number of times. <br>
db is short for define byte.

For the curious ones who want to know how this works, $ refers to current line's address, and $$ refers to the address of the start of the section.
So we can know how many bytes our code has consumed by adding the above statement at the end of our instructions.
And since we need to pad upto 510 bytes, we subtract the number of bytes of our code uses from 510, and fill it with bytes ```0```.

Note: We can fill it with any other byte other than ```0``` as we are not using it anyways. 0s are better because even if by mistake
this part is executed, it will result in a no-op instruction and won't corrupt anything.

To add 55, AA at the end we will use
```nasm
db 0x55
db 0xAA
```
We are adding 0x in the front to say that the given argument is a hex value (base 16). By default, decimal value (base 10) will be assumbed.

Instead of defining two bytes, we can combine these together to define a single word. 1 word = 2 bytes.
```nasm
dw 0xAA55
```
Note: The order is reversed because our processor is little endian

Stitching it all together, our final code is as follows
```nasm
bits 16
jmp $
times 510-($-$$) db 0
dw 0xAA55
```

* * *

We can assemble this into our MBR payload using
```
nasm -f bin -o blank-os.bin blank-os.asm
```

We would get blank-os.bin We can test it by using this as a hard disk in qemu.
```
qemu-system-x86_64 blank-os.bin
```

<br>
Check if your screen is stuck as below
```
Booting from Hard Disk...
_
```
If yes, congratulations, you have booted your own OS!

<br>
Note: BIOS is replaced with a newer standard called UEFI. Though UEFI can support BIOS using legacy option, the
way it loads the bootloader is entirely different. I will try to make the same infinite loop os tutorial for UEFI later.

<!---
Provide links for
- real mode
- MBR
- Endianness
- Assembly
- NASM
- Next posts
--->