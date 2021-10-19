+++
title = "Hack-a-thon and Embedded Rust"
date = 2021-08-16

[taxonomies]
tags = ["Rust", "Embedded"]
+++
This spring, A couple colleagues and I began to work through [The Discovery Book](https://docs.rust-embedded.org/discovery/), an introduction to embedded development in Rust. A few months into the book, I attended the SEP summer hackathon to use the lessons and code from the book to create a thermometer.

Last year, I had built a thermostat to turn a mini-fridge into a fermentation chamber. The thermostat ran on a Raspberry Pi, which connected to a Meros Smart Outlet and a SHT31D temperature and humidity sensor from Adafruit. The software was from the (then Mozilla) Webthings Framework. 

{{ resize_image(path="content/hack-a-thon-2021/rpi-overhead.jpeg", width=0,height=300, op="fit_height") }}

It was fun and it worked pretty well! But I felt provisioning a tiny Linux box for a thermostat seemed a bit overkill. Enter: the STM32FDISCOVERY.

{{ resize_image(path="content/hack-a-thon-2021/first-sensor.jpeg", width=0,height=300, op="fit_height") }}

My STM32DISCOVERY programs are running on bare metal: no operating system, files, Wifi, etc.. The Discovery authors provided a way to flash the program onto the board, debug the program as it runs on board, and capture log and error messages.

My goal for the weekend was to hook up and read from the SHT31D sensor. I split up the work in incremental steps.

First, I wanted to see my board communicate over I2C, the protocol the temperature sensors use. The easiest way was to get the board to read off of its on-board magnetometer. After a few tweaks to the Discovery book's example for my newer revision of the board, I got some readings:

{{ resize_image(path="content/hack-a-thon-2021/magnometer.jpeg", width=0,height=300, op="fit_height") }}

Second step: read from my colleague's Bosh BME280 sensor, which had a more popular open-source package than my SHT31D. There was an issue with the library: the factory function that produced the sensor object consumed a delay object. This meant when I tried to use the delay object to pause between readings, I would get an error like this: 

{{ resize_image(path="content/hack-a-thon-2021/delay-abstraction-problem.png", width=0,height=300, op="fit_height") }}

Rust does not allow for multiple mutable references. There was a pull request to solve the issue that had not been merged in a year. Luckily, cargo provides an easy way to bring in the code of this PR straight from Github:
 
{{ resize_image(path="content/hack-a-thon-2021/cargo-import-git.png", width=0,height=300, op="fit_height") }}

Third step: Use the SHT31D. This was straightforward, save for some minor wire repair.

Having accomplished my goal on Saturday, I decided to answer a question that seemed solvable with my remaining time: can the board read from both sensors at the same time?

[Adafruit explains](https://learn.adafruit.com/i2c-addresses#i2c-inter-integrated-circuit-communications-2630755-2) I2C buses have one master and one to many slaves existing on the same line.  Would the embedded-rust ecosystem allow it? At first glance: no. 

Bringing in both sensors in a program, both required the I2C bus object. Only one could have it, else I get an error. Luckily spotted in a Github thread: the shared-bus crate. This one could manage the references to the bus:

{{ resize_image(path="content/hack-a-thon-2021/shared-bus.png", width=0,height=300, op="fit_height") }}

In the end, I was able to read from both temperature sensors at the same time, on the same connection. I was pleasantly surprised to see both sensors giving similar readings.

That was my weekend! Had a blast, looking forward to the next one.

The code is on [my fork](https://github.com/pfesenmeier/discovery) of the Discovery Book, in folders `src/9{7-9}-.+`
