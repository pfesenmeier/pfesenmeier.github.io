+++
title = "Hack-a-thon and embedded-hal"
date = 2021-08-16

[taxonomies]
tags = ["Rust", "Embedded"]
+++
This spring, A couple colleagues and I began to work through [The Discovery Book](https://docs.rust-embedded.org/discovery/), an introduction to embedded development in Rust. A few months into the book, I attended the summer hackathon to use the lessons and code from the book to create a thermometer.

Last year, I had built a thermostat to turn a mini-fridge into a fermentation chamber. The thermostat ran on a Raspberry Pi, which connected to a Meros Smart Outlet and a SHT31D temperature and humidity sensor from Adafruit. The software was from the (then Mozilla) Webthings Framework. 
// insert photo
It was fun and it worked pretty well! But I felt provisioning a tiny Linux box for a thermostat seemed a bit overkill. Enter: the STM32FDISCOVERY.
// insert photo

My STM32DISCOVERY programs are running on bare metal: no operating system or filesystem. The Discovery authors provided a way to flash the program onto the board, debug the program as it runs on board, and capture log and error messages.

My goal for the weekend was to hook up and read from the SHT31D sensor. I split up the work in incremental steps.

First, I wanted to see my board communicate over I2C, the protocol the SHT31D uses. The easiest way was to run the Discovery Book's example that read from the STM32's on-board magnetometer. After a few tweaks to get the example working for my newer revision of the board, I got some readings:
// insert photo

Second step: read from my colleague's Bosh BME280 sensor, which had a more popular open-source package than my SHT31D. There was an issue with the library: the factory function that produced the sensor object consumed a delay object:
// code snippet
This meant when I tried to use the delay object to pause between readings, I would get this error:
// code snippet
Rust does not allow for multiple mutable references. There was a pull request to solve the issue that had not been merged in a year. Luckily, cargo provides an easy way to bring in the code of this PR straight from Github.
 
// cargo.toml syntax

Third step: Use the SHT31D. This was straightforward, save for some minor wire repair:

Having accomplished my goal on Saturday, I decided to answer a question that seemed solvable with my remaining time: can the board read from both sensors from at the same time?

Adafruit explains I2C buses have one master and one to many slaves existing on the same line.  Would the embedded-rust ecosystem allow it? At first glance: no. 

Bringing in both sensors in a program, both required the i2c bus object. Only one could have it, else I get the error: 
// error

Luckily spotted in a Github thread: the shared-bus crate. This one could manage the references to the bus.
// code snippet

At the end, I was able to read from both temperature sensors at the same time, on the same connection. I was pleasantly surprised to see both sensors giving similar readings.

That was my weekend! Had a blast, looking forward to the next one.



