+++
title = "WSL2 and Embedded Rust"
draft = true
date = 2021-10-16

[taxonomies]
tags = ["Rust", "Embedded"]
+++

WSL2 and embedded Rust, at first glance, do not seem compatible. [WSL2 does not](https://github.com/microsoft/WSL/issues/5158) have access to the host Windows machine's USB ports. In order to flash the binary onto the board, the program needs access to this port.

The trick is to have your flashing program installed on the Windows side, and invoke it from the WSL2 terminal. Here are two examples:

The dev setup outlined in the [Discovery Book](https://docs.rust-embedded.org/discovery/) uses openocd to connect to the usb port, and listen for any gdb connections that want to flash a binary onto the board.

WSL2 users can install openocd.exe on their Windows side, and invoke it in their terminal. A common dev setup will look like:

// terminal 1: Connect to board, listen for connections
// "bindto" tells the program to accept connections from other computer, like our WSL2 instance
openocd.exe -s ./scripts/ -f interface/stlink.cfg -f target/stm32f3x.cfg -c "bindto 0.0.0.0"
// terminal 2: Build binary using cargo build, then calls "gdb-multiarch -q -x ../openocd.gdb" 
cargo run
// terminal 3: Capture logging
// this still works, with openocd being a windows binary!
itmdump -F -f itm.txt

There is now a little overhead to get the openocd.exe and gdb-multiarch to connect.
We need to include "bindto 0.0.0.0" for the openocd.exe to accept connections from other machines (this time, our Ubuntu instance). Also we need to tell gdb-multiarch the ip address of the Windows machine. So in openocd.gdb, 'target remote :3333' becomes 'target remote 172.21.160.1:3333'. Problem is, WSL2 assigns the Windows instance a new IP address on computer startup.

My solution is to run a command like this in my .bash_profile:

#!/bin/bash

project_location="$HOME/discovery"
windows_ip_address=$(grep "nameserver" /etc/resolv.conf | sed 's/nameserver //')

# update gdb commands with current windows ip address
find "${project_location}/src" -name openocd.gdb -exec sed -Ei "s/^target remote.+\$/target remote ${windows_ip_address}:3333/" {} \;

Next, the esp32c3. This start for me at ivmarkov's [rust-esp32-std-hello](https://github.com/ivmarkov/rust-esp32-std-hello). This example uses tools available on crates.io to flash the program. We can cross-compile the flashing tool, which makes puts it on the Windows side and gives it visibility to the USB ports. So my setup of this example went something like:

```
// use latest nightly toolchain
rustup toolchain add nightly
rustup update
rustup defaulty nightly

// dependency to cross-compile Windows
sudo apt install mingw-w64
// add Windows target
rustup target add x86_64-pc-windows-gnu
cargo install ldproxy
cargo install espflash --target x86_64-pc-windows-gnu
cargo install espmonitor --target x86_64-pc-windows-gnu // if needed
cargo pio works?

cargo-pio.exe installpio
```

Then a common dev run would look like
// flash
// monitor board
```

