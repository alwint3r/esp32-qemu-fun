# Building QEMU for ESP32

Right now the support for ESP32-family chip has not landed on upstream repository of QEMU. You can use the following repository to build QEMU with ESP32 support.

```bash
git clone https://github.com/espressif/qemu.git
```

## Configure

```bash
./configure --target-list=xtensa-softmmu \
    --enable-gcrypt \
    --enable-slirp \
    --enable-debug --enable-sanitizers \
    --enable-sdl \
    --disable-strip --disable-user \
    --disable-capstone --disable-vnc \
    --disable-gtk
```

## Compile

```bash
cd build
ninja all
```

You will find the `qemu-system-xtensa` binary in the `build` directory after a successful build.

### macOS Users Issues with gnutls

Some macOS users (including me) experienced the following errors while compiling:

```bash
FAILED: libcrypto.fa.p/crypto_tlscredsx509.c.o 
cc -Ilibcrypto.fa.p -I. -I.. -Iqapi -Itrace -Iui -Iui/shader -I/opt/homebrew/opt/libgcrypt/include -I/opt/homebrew/opt/libgpg-error/include -I/opt/homebrew/Cellar/glib/2.78.4/include/glib-2.0 -I/opt/homebrew/Cellar/glib/2.78.4/lib/glib-2.0/include -I/opt/homebrew/opt/gettext/include -I/opt/homebrew/Cellar/pcre2/10.42/include -I/opt/homebrew/Cellar/glib/2.78.4/include -fcolor-diagnostics -Wall -Winvalid-pch -std=gnu11 -O0 -g -fsanitize=undefined -fsanitize=address -fstack-protector-strong -Wundef -Wwrite-strings -Wmissing-prototypes -Wstrict-prototypes -Wredundant-decls -Wold-style-definition -Wtype-limits -Wformat-security -Wformat-y2k -Winit-self -Wignored-qualifiers -Wempty-body -Wnested-externs -Wendif-labels -Wexpansion-to-defined -Wmissing-format-attribute -Wno-initializer-overrides -Wno-missing-include-dirs -Wno-shift-negative-value -Wno-string-plus-int -Wno-typedef-redefinition -Wno-tautological-type-limit-compare -Wno-psabi -Wno-gnu-variable-sized-type-not-at-end -iquote . -iquote /Users/alwin/explores/qemu-esp32/qemu -iquote /Users/alwin/explores/qemu-esp32/qemu/include -iquote /Users/alwin/explores/qemu-esp32/qemu/host/include/aarch64 -iquote /Users/alwin/explores/qemu-esp32/qemu/host/include/generic -iquote /Users/alwin/explores/qemu-esp32/qemu/tcg/aarch64 -D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -fno-strict-aliasing -fno-common -fwrapv -fno-pie -MD -MQ libcrypto.fa.p/crypto_tlscredsx509.c.o -MF libcrypto.fa.p/crypto_tlscredsx509.c.o.d -o libcrypto.fa.p/crypto_tlscredsx509.c.o -c ../crypto/tlscredsx509.c
In file included from ../crypto/tlscredsx509.c:23:
../crypto/tlscredspriv.h:27:10: fatal error: 'gnutls/gnutls.h' file not found
#include <gnutls/gnutls.h>
         ^~~~~~~~~~~~~~~~~
1 error generated.
[897/2328] Compiling C object libcrypto.fa.p/crypto_hash-gcrypt.c.o
```

The reason is the compiler couldn't find the gnutls installation.
I installed the library using Homebrew, so it should be available in homebrew installation directory.
To fix that, we need to add the following flags to the `configure` command:

```bash
--extra-cflags="-I/opt/homebrew/opt/gnutls/include" \
--extra-ldflags="-L/opt/homebrew/opt/gnutls/lib"
--extra-cxxflags="-I/opt/homebrew/opt/gnutls/include" \
```

So we need to reconfigure the build with the following command:

```bash
./configure --target-list=xtensa-softmmu \
    --enable-gcrypt \
    --enable-slirp \
    --enable-debug --enable-sanitizers \
    --enable-sdl \
    --disable-strip --disable-user \
    --disable-capstone --disable-vnc \
    --disable-gtk \
    --extra-cflags="-I/opt/homebrew/opt/gnutls/include" \
    --extra-ldflags="-L/opt/homebrew/opt/gnutls/lib" \
    --extra-cxxflags="-I/opt/homebrew/opt/gnutls/include"
```

