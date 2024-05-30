<img width="326" alt="image" src="https://github.com/mreddybalaji/ARM-BARE-METAL-PROGRAMMING/assets/130784457/1e4dd2d8-ef17-4ad4-88b1-926bfc971942">


## 1. COFIGURATION SETUP:
### Requirement:
`Windows OS`, 1TB HDD, 16GB RAM.Allocate 8GB RAM, 100GB HDD for `Oracle` Virtual Machine. Install `Ubuntu` using `VM`

### Installing QEMU:
QEMU:

Install QEMU's ARM version using the qemu-system-arm package on Debian/Ubuntu systems for ARM emulation. This avoids hardware requirements, software flashing, and provides better tools for inspecting emulated hardware.

```
 sudo apt-get install qemu-system-arm
```
### GCC cross-compiler toolchain:

By running `gcc -dumpmachine`, I get `x86_64-linux-gnu`, and yours will likely be the same or similar. This information helps identify the target triplet for your platform when selecting the appropriate cross-compiler toolchain.

Install gcc-arm-none-eabi, `version 6` or newer, for U-Boot on Ubuntu.

```
sudo apt-get install gcc-arm-none-eabi
```
Check the version with `arm-none-eabi-gcc --version`. If your distribution has an old version, download from ARM and add the toolchain's folder to your PATH after extraction.


### Build system essentials:
Install build-essential for Make and other tools, and cmake for CMake on Debian-based systems.
```
 sudo apt-get install build-essential cmake

```
Install `bison` and `flex` if not already present; these tools are required for building U-Boot on certain Linux variants.
```
 sudo apt-get install bison flex

```

sort -R ~/facts-and-trivia | head -n1

The flex program is an implementation of lex, a standard lexical analyzer first developed in the
mid-1970s by Mike Lesk and Eric Schmidt, who served as the chairman of Google for some years.


## 2. THE FIRTST BOOT:

Starting an Emulated `ARM` Machine in `QEMU` by the Cross-Compiler Toolchain 

### First code:
```
qemu-system-arm -M vexpress-a9 -m 32M -no-reboot -nographic -monitor telnet:127.0.0.1:1234,server,nowait

```
`-M vexpress-a9`: Specifies the machine type as vexpress-a9, which is a specific ARM development board model.

`-m 32M`: Sets the RAM size to 32MB for the emulated machine.

`-no-reboot`: Prevents the emulated machine from automatically rebooting after shutdown.

`-nographic`: Disables graphical output, running the emulation in text console mode only.

`-monitor telnet`:127.0.0.1:1234,server,nowait: Opens a telnet monitor interface on localhost port 1234, allowing remote control and monitoring of the emulated machine.

This command is quite comprehensive and sets up a minimal ARM system for emulation without graphical output, with a specific machine type and RAM size, and enables remote monitoring via telnet.

By running, although it will not do anything except crashing with a message like  `qemu-system-arm: Trying to execute code outside RAM or ROM at 0x04000000`.

### The first hang:

```
ldr r2, str1
```
```
b .
```
```
str1: .word 0xDEADBEEF

```
ldr r2, str1: This line loads a 32-bit word from the memory location labeled str1 into register r2. The ldr instruction stands for "load register."

b .: This line is a branch instruction that branches to the current location, effectively creating an infinite loop. The dot (.) refers to the current location in the code.

str1: .word 0xDEADBEEF: This line defines a label str1 and assigns a 32-bit hexadecimal value (0xDEADBEEF) to the memory location labeled str1. The .word directive is used to allocate space for a 32-bit word.

In summary, the code loads the value 0xDEADBEEF from the memory location str1 into register r2, then enters an infinite loop. This is a simple example often used in bare-metal programming to demonstrate loading data from memory and creating an infinite loop.

Because this is not executable code? it’s just an instruction to the assembler to allocate the 4-byte word here.

`sort -R ~/facts-and-trivia | head -n1`.

To assemble the code, use the GNU Assembler (as) to create startup.o:

```
 arm-none-eabi-as -o startup.o startup.s
```

Without any C files yet, we can link the object file to obtain an executable. Here's the command:

```
 arm-none-eabi-ld -o first-hang.elf startup.o
```

To convert the ELF file to a raw binary dump, use the following command:

```
 arm-none-eabi-objcopy -O binary first-hang.elf first-hang.bin

```

We've successfully transformed the `startup.s` assembly code into an executable format for an ARM CPU. Starting with assembling the code using `arm-none-eabi-as` to create `startup.o`, we then linked it with `arm-none-eabi-ld` to produce `startup.elf`, even though it gave a warning about a missing `_start` symbol which we ignored as we only needed the ELF file temporarily. To convert it into a raw binary dump, we used `arm-none-eabi-objcopy` with the `-O binary` option, resulting in `first-hang.bin`, a 12-byte file containing the assembly instructions and data. While matching bytes to assembly instructions in a hexdump is atypical, it illustrates the process of generating executable code for ARM architecture, crucial for bare-metal programming.


### And. . . Blastoff!:

Run code on `ARM machine`:
```
qemu-system-arm -M vexpress-a9 -m 32M -no-reboot -nographic -monitor telnet:127.0.0.1:1234,server,nowait -kernel first-hang.bin
```

qemu-system-arm: This is the command to start the QEMU emulator for ARM systems.

-M vexpress-a9: This option specifies the machine type to emulate. vexpress-a9 is an ARM Versatile Express development board with a Cortex-A9 CPU.

-m 32M: This sets the memory size for the emulated machine to 32 megabytes.

-no-reboot: This tells QEMU not to reboot the machine after it shuts down. This can be useful for debugging purposes.

-nographic: This disables graphical output so that the output goes to the terminal instead. It is useful for running QEMU in environments where you don't need a graphical interface.

-monitor telnet:127.0.0.1:1234,server,nowait: This option sets up a QEMU monitor (a command-line interface for interacting with the emulator) that listens on port 1234 of the localhost (127.0.0.1). The server option makes QEMU wait for incoming connections, and nowait allows QEMU to continue running even if there is no connection.

-kernel first-hang.bin: This specifies the kernel file to load. first-hang.bin is the binary file of the kernel you want to run on the emulated ARM machine.

```
 QEMU 2.8.1 monitor - type 'help' for more information
```

At the (qemu) prompt, type info registers. This command shows the current state of the CPU registers. You should see an output listing all the registers and their values. Look for the register R2 near the beginning of the output, and you should spot our 0xDEADBEEF constant loaded into it.

```
R00=00000000 R01=000008e0 R02=deadbeef R03=00000000
```

We have our first register write and hang.

### Memory mappings:

The system don't know the start address where we want, it begins executing from address 0x0.

```
 str1: .word 0xDEADBEEF
```
```
ldr r2,str1
```
```
b .
```

























