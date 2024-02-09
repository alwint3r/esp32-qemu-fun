# Hello World Project

This project is a modified version of the hello_world example from the ESP-IDF framework. It is used to demonstrate the use of the ESP-IDF framework with the ESP32 microcontroller, or precisely a QEMU that emulates the ESP32 microcontroller. But, sure you can easily run this project on a real ESP32 microcontroller.


## Build and Run the Project

Use the idf.py tool to build the project.

```bash
idf.py build
```

Now, we need to merge the bootloader, partition table, and the application binary into a single binary file. We can do this by running the following command.

```bash
cd build
esptoo.py --chip esp32 merge_bin --fill-flash-size 4MB -o flash_image.bin @flash_args
```

Open another terminal and run the following command to start the QEMU emulator.

```bash
# Assuming you're still in the hello_world directory
../../qemu/build/qemu-system-xtensa -nographic -s -S -machine esp32 -drive file=build/flash_image.bin,if=mtd,format=raw
```

On the first terminal, run the following command to start the GDB debugger.

```bash
xtensa-esp32-elf-gdb hello_world.elf \
    -ex "target remote localhost:1234" \
    -ex "monitor system-reset" \
    -ex "tb app_main" \
    -ex "c"
```

The QEMU emulator will boot the firmware and stop at the `app_main` function. You can run the app_main function by typing `c` in the GDB debugger.
