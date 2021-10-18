+++
title = "WSL2 and Embedded Rust"
date = 2021-10-16
draft = true

[taxonomies]
tags = ["Rust", "Embedded"]
+++

WSL2 and embedded Rust seem incompatible. [WSL2 does not](https://github.com/microsoft/WSL/issues/5158) have access to the host Windows machine's USB ports. In order to flash the binary onto the board, the program needs access to this port.

The trick is to have your flashing and monitoring programs installed as Windows binaries. Windows binaries are [always run as if](https://docs.microsoft.com/en-us/windows/wsl/filesystems#run-windows-tools-from-linux) in a Windows CMD prompt. Windows shells have access to the USB ports. Here are two examples:

## STM32F3DISCOVERY

The dev setup outlined in the [Discovery Book](https://docs.rust-embedded.org/discovery/) for the STM32F3DISCOVERY board uses gdb to flash and debug the program on the board. Here is a typical setup:

```
// Terminal 1: Start gdb server, listen for connections
openocd.exe -s ./scripts/ -f interface/stlink.cfg -f target/stm32f3x.cfg -c "bindto 0.0.0.0"

// Terminal 2: Build project, Then call "gdb-multiarch -q -x ../openocd.gdb" 
cargo run

// Terminal 3, same directory as gdb client: Capture logging
// Still works with openocd now being a Windows binary!
itmdump -F -f itm.txt
```

There is now a little overhead to get the gdb server and gdb client to connect:
- We need to tell the gdb server to accept connections from other machines. Hence, we added "bindto 0.0.0.0" to openocd.exe command.
- Also we need to tell the gdb client running in Ubuntu the ip address of the gdb server on the Windows machine. Hence we added the ip address in the target remote command in openocd.gdb, e.g. 'target remote 172.21.159.1:3333'. 

Problem is, WSL2 assigns the Windows instance a new IP address on computer startup. My solution is to run a command like this in my .bash_profile:

```
#!/bin/bash

project_location="$HOME/discovery"
# ip address of windows machine always in /etc/resolv.conf
windows_ip_address=$(grep "nameserver" /etc/resolv.conf | sed 's/nameserver //')

# update gdb commands with current windows ip address
find "${project_location}/src" -name openocd.gdb -exec sed -Ei "s/^target remote.+\$/target remote ${windows_ip_address}:3333/" {} \;
```

## ESP32C3

ivmarkov's [rust-esp32-std-hello](https://github.com/ivmarkov/rust-esp32-std-hello) is an excellent starting place for seeing all that running embedded Rust can do on top of the ESP32C3 running espressif OS, including connecting to WIFI. This example also makes getting build and flashing tools a cinch, since we can just rely on tools from crates.io. If we download the flashing tools as Windows binaries, they will run with access to USB ports. So my setup of this example went something like:

```
// use latest nightly toolchain
rustup toolchain add nightly
rustup update
rustup defaulty nightly

// install build tools
cargo install ldproxy

// download and cross-compile flashing and monitoring tools
sudo apt install mingw-w64
rustup target add x86_64-pc-windows-gnu
cargo install espflash --target x86_64-pc-windows-gnu
cargo install espmonitor --target x86_64-pc-windows-gnu // monitor that is quick to startup
cargo install cargo-pio --target x86_64-pc-windows-gnu // monitor that is slow on startup, but decodes stack traces
```

Then a common dev run would look like

```
// build and flash
cargo build && espflash.exe COM6 target/riscv32imc-esp-espidf/debug/rust-esp32-std-hello
// monitor board
cargo-pio.exe espidf monitor COM6 // or espmonitor.exe COM6
```

