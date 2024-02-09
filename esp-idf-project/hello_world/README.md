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


## Using esptool.py and espefuse.py

Generate an efuse drive file using the following command

```bash
dd if=/dev/zero bs=1 count=124 of=efuse_image.bin
```

Run the emulator with the efuse drive file

```bash
../../qemu/build/qemu-system-xtensa -nographic -machine esp32 \
    -drive file=build/flash_image.bin,if=mtd,format=raw \
    -global driver=esp32.gpio,property=strap_mode,value=0x0f \
    -drive file=efuse_image.bin,if=none,format=raw,id=efuse \
    -global driver=nvram.esp32.efuse,property=drive,value=efuse \
    -serial tcp::5555,server,nowait
```

Run esptool.py to communicate with the emulator using socket connection

```bash
esptoo.py --chip esp32 -p socet://localhost:5555 --after no_reset --before no_reset flash_id
```

The `no_reset` value for both `--after` and `--before` options is used to avoid errors because the QEMU emulator does not allow us to reset the emulator using the RST signal.


### Burning Custom MAC Address

We will use the espefuse.py tool to burn the custom MAC address.

```bash
espefuse.py --port socket://localhost:5555 --before no_reset --do-not-confirm burn_custom_mac aa:bb:cc:11:22:33
```

Here's the output

```
espefuse.py v4.7.0
WARNING: Pre-connection option "no_reset" was selected. Connection may fail if the chip is not in bootloader or flasher stub mode.
Connecting...
Device PID identification is only supported on COM and /dev/ serial ports.

Detecting chip type... Unsupported detection protocol, switching and trying again...
Connecting...
Device PID identification is only supported on COM and /dev/ serial ports.
..
Detecting chip type... ESP32

=== Run "burn_custom_mac" command ===
    - 'MAC_VERSION' (Version of the MAC field) 0x00 -> 0x1
    - 'CUSTOM_MAC' (Custom MAC address) 0x000000000000 -> 0x332211ccbbaa
    - 'CUSTOM_MAC_CRC' (CRC8 for custom MAC address) 0x00 -> 0x6a

Check all blocks for burn...
idx, BLOCK_NAME,          Conclusion
[03] BLOCK3               is empty, will burn the new value
. 
This is an irreversible operation!
BURN BLOCK3  - OK (write block == read block)
Reading updated efuses...
Custom MAC Address version 1: aa:bb:cc:11:22:33 (CRC 0x6a OK)
Successfu
```

Note that you can only do this once. If you're using QEMU, we can reset the MAC address by simply re-generating the efuse drive file using the `dd` command. But in a real ESP32 microcontroller, you can't reset the custom MAC address once it's burned.

If we retry the `burn_custom_mac` command, we will get the following message:

```
espefuse.py v4.7.0
WARNING: Pre-connection option "no_reset" was selected. Connection may fail if the chip is not in bootloader or flasher stub mode.
Connecting...
Device PID identification is only supported on COM and /dev/ serial ports.

Detecting chip type... Unsupported detection protocol, switching and trying again...
Connecting...
Device PID identification is only supported on COM and /dev/ serial ports.

Detecting chip type... ESP32

=== Run "burn_custom_mac" command ===
    - 'CUSTOM_MAC' (Custom MAC address) 0x332211ccbbaa -> 0x332211ccbbaa
        The same value for CUSTOM_MAC is already burned. Do not change the efuse.
    - 'CUSTOM_MAC_CRC' (CRC8 for custom MAC address) 0x6a -> 0x6a
        The same value for CUSTOM_MAC_CRC is already burned. Do not change the efuse.

Check all blocks for burn...
idx, BLOCK_NAME,          Conclusion
Nothing to burn, see messages above.
```

### Reading the Custom MAC address

The first option is to use the espefuse.py tool again with a slightly different command

```bash
espefuse.py --port socket://localhost:5555 --before no_reset get_custom_mac
```

Here's the output

```
espefuse.py v4.7.0
WARNING: Pre-connection option "no_reset" was selected. Connection may fail if the chip is not in bootloader or flasher stub mode.
Connecting...
Device PID identification is only supported on COM and /dev/ serial ports.

Detecting chip type... Unsupported detection protocol, switching and trying again...
Connecting...
Device PID identification is only supported on COM and /dev/ serial ports.
..
Detecting chip type... ESP32

=== Run "get_custom_mac" command ===
Custom MAC Address version 1: aa:bb:cc:11:22:33 (CRC 0x6a OK)
```

The second option is to use `xxd` to inspect the efuse drive file

```
xxd efuse_image.bin
```

Here's the output

```
00000000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000040: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000050: 0000 0000 0000 0000 0000 0000 6aaa bbcc  ............j...
00000060: 1122 3300 0000 0000 0000 0000 0000 0000  ."3.............
00000070: 0000 0001 0000 0000 0000 0000            ............
```

You can see the custom MAC address.

### Flashing the Firmware

We can use the same port and protocol as before, but we need to change the `--before` and `--after` option to `no_reset` because again, the QEMU emulator does not allow us to reset the emulator using the RST signal.

Unfortunately, we can't use the CLI for this since the `esptool.py` tool is called internally by the `idf.py` tool. But, we can configure the flasher using the `idf.py menuconfig` command.

Go to the `Serial flasher config` section and change the `Before flashing` option to `No reset` and the `After flashing` option to `Stay in bootloader`.

Now, we can flash the firmware using the `idf.py flash` command.

```bash
idf.py flash --port socket://localhost:5555
```

This manouver will cause the entire source code to be recompiled. But, hey we can "flash" the firmware to the QEMU emulator without any real hardware. Cool, right?


My only question is, does the flash_image.bin file changed? Let's find out.
We're going to slightly modify the `hello_world/main/hello_world_main.c` file and reflash the firmware to the QEMU emulator using the previous command.

Now, let's re-run the QEMU emulator and see if the firmware has been updated.

```bash
../../qemu/build/qemu-system-xtensa -nographic -s -S -machine esp32 -drive file=build/flash_image.bin,if=mtd,format=raw
```

Here's the output of the QEMU emulator

```
Hello worldeeee!
This is esp32 chip with 2 CPU core(s), WiFi/BTBLE, silicon revision v0.0, 4MB external flash
Minimum free heap size: 301236 bytes
This is esp32 chip with 2 CPU core(s), WiFi/BTBLE, silicon revision v0.0, 4MB external flash
Minimum free heap size: 301236 bytes
This is esp32 chip with 2 CPU core(s), WiFi/BTBLE, silicon revision v0.0, 4MB external flash
Minimum free heap size: 301236 bytes
This is esp32 chip with 2 CPU core(s), WiFi/BTBLE, silicon revision v0.0, 4MB external flash
```

Notice that the `Hello world!` message has been changed to `Hello worldeeee!`. So, the flash_image.bin file has been updated! Cool, right?
