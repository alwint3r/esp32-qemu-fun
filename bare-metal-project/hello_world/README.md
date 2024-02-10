# Hello Bare Metal World

Using QEMU to run bare metal ESP32 projects is great because it allows us to run our code on a virtual machine and avoid the risk of bricking our real ESP32 boards (even though they are cheap and widely available almost everywhere, I love it).

This project will override the first stage bootloader of the ESP32 and run our code directly on the hardware.

## Building and Running

Use the xtensa-esp32-elf toolchain to build the project.

```bash
xtensa-esp32-elf-gcc -o hello_world.elf main.c -specs=sim.elf.specs -specs=sys.qemu.specs
```

Then run the project using QEMU.

```bash
../../qemu/build/qemu-system-xtensa -nographic \
    -monitor null \
    -cpu esp32 \
    -M esp32 \
    -kernel ./hello_world.elf \
    --semihosting
```

## Building with Clang

Espressif has created a fork of LLVM (that will soon be merged into the upstream repository) that supports the Xtensa targets.
We can compile the source code using clang, but somehow the linking process is not working as expected.

```
clang -target xtensa -mcpu=esp32 -specs=sim.elf.specs -specs=sys.qemu.specs -o hello_world.elf main.c
```

This command will yield the following linker error:

```
clang-16: warning: argument unused during compilation: '-specs=sim.elf.specs' [-Wunused-command-line-argument]
clang-16: warning: argument unused during compilation: '-specs=sys.qemu.specs' [-Wunused-command-line-argument]
/Users/alwin/Downloads/esp-clang/bin/../lib/gcc/xtensa-esp32-elf/11.2.0/../../../../bin/xtensa-esp32-elf-ld: warning: cannot find entry symbol _start; defaulting to 0000000000400094
/Users/alwin/Downloads/esp-clang/bin/../lib/gcc/xtensa-esp32-elf/11.2.0/../../../../bin/xtensa-esp32-elf-ld: /var/folders/cf/0yl4nkf536d0fg_pz2qwt4p40000gn/T/main-46143a.o:(.literal+0x4): undefined reference to `printf'
clang-16: error: ld command failed with exit code 1 (use -v to see invocation)
```

In short, the linker can't find the symbol to `_start` and `printf` which may be located in the linker script and the newlib library respectively.

Let's stick with the `xtensa-esp32-elf-gcc` for now.