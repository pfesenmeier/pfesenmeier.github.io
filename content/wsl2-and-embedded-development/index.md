+++
title = "WSL2 and Embedded Rust"
date = 2021-10-16
draft = false

[taxonomies]
tags = ["Rust", "Embedded"]
+++

WSL2 and embedded development seem incompatible. A microcontroller plugged into a Windows USB port is [only visible to the host Windows machine](https://github.com/microsoft/WSL/issues/5158). However, it is possible to flash and debug a microcontroller right from the Linux terminal.

The trick is to have the tools that need access to the USB port installed as Windows binaries. Windows binaries invoked from the WSL terminal are [run as if](https://docs.microsoft.com/en-us/windows/wsl/filesystems#run-windows-tools-from-linux) in a Windows CMD prompt. Here are two examples:

{{ resize_image(path="content/wsl2-and-embedded-development/boards.JPEG", width=0,height=300, op="fit_height") }}
<small style="display: flex; justify-content: center;">
The STM32F3DISCOVERY and the ESP32c3
</small>

## STM32F3DISCOVERY

The project outlined in the [Discovery Book](https://docs.rust-embedded.org/discovery/) uses gdb to flash and debug the program on the board. Here is how the program is run:

```
# in /tmp/
  # Terminal 1: Start a gdb server that connects to board on usb port, listens for gdb client connections:
  openocd.exe -s ./scripts/ -f interface/stlink.cfg -f target/stm32f3x.cfg -c "bindto 0.0.0.0"

  # Terminal 2: Capture logging from running project:
  itmdump -F -f itm.txt

# in project folder
  # Terminal 3: Build project, send binary to gdb server, and start debug session:
  cargo run

```
`openocd.exe` runs as if from Windows, and finds the board on the USB port. `cargo` builds the project, then the gdb client in Linux opens a remote debugging session with the gdb server running on the Windows side. In order for logging to work, the `itmdump` and `openocd.exe` commands must be run in same directory. It turns out that this still works even now one is a Windows program!

The option "bindto 0.0.0.0" tells the gdb server to accept connections from other (virtual) machines. The line 'target remote 172.21.159.1:3333' in openocd.gdb tells the Linux gdb client where to find the gdb server running on Windows.

Important to note that the Windows IP address is not static. My solution is to run a command like this from my .bashrc:

```
#!/bin/bash

project_location="$HOME/discovery"

# find ip address of windows machine
windows_ip_address=$(grep "nameserver" /etc/resolv.conf | sed 's/nameserver //')

# update gdb commands with current windows ip address
find "${project_location}/src" -name openocd.gdb -exec sed -Ei "s/^target remote.+\$/target remote ${windows_ip_address}:3333/" {} \;
```

## ESP32C3

ivmarkov's [rust-esp32-std-hello](https://github.com/ivmarkov/rust-esp32-std-hello) has a straighforward setup to develop a part c / part Rust project on the esp32 series of boards. All tools are from crate.io. All WSL users have to do differently to setup a development environment is cross-compile the flashing and monitoring tools:

```
// use latest nightly toolchain
rustup toolchain add nightly
rustup update
rustup defaulty nightly

// install linker proxy 
cargo install ldproxy

// download and cross-compile flashing and monitoring tools
sudo apt install mingw-w64
rustup target add x86_64-pc-windows-gnu
cargo install espflash --target x86_64-pc-windows-gnu
cargo install espmonitor --target x86_64-pc-windows-gnu // monitor that is quick to startup
cargo install cargo-pio --target x86_64-pc-windows-gnu // monitor that is slow on startup, but decodes stack traces
```

Then running the program looks like:

```
// build and flash
cargo build && espflash.exe COM6 target/riscv32imc-esp-espidf/debug/rust-esp32-std-hello
// monitor board
cargo-pio.exe espidf monitor COM6 // or espmonitor.exe COM6
```

