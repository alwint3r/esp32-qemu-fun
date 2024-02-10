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