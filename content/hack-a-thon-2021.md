+++
title = "Hack-a-thon and embedded-hal"
date = 2021-08-16

[taxonomies]
tags = ["Rust", "Embedded"]
+++
Several months ago, a couple SEP collegues and I started a study group around [The Discovery Book](https://docs.rust-embedded.org/discovery/), an introduction to embedded development in Rust. It's an excellent resource, especially for someone like me without any experience with low-level concepts. So when hack-a-thon came around, I felt poised to answer something that had been in the back of my mind: what would happen if I stepped outside of our book and tried something on my own?

Last year, I had built a thermostat to turn a mini-fridge into a fermentation chamber. The thermostat was built out of a Raspberry Pi, a Meros Smart Outlet, and a SHT31D temperature and humidity sensor from Adafruit. The software was from the (then Mozilla) Webthings Framework. It was fun and it worked pretty well! But I felt provisioning a tiny Linux box for a thermostat seemed a bit overkill. Enter: the STM32FDISCOVERY.

The important thing to know about programmingthe STM32DISCOVERY is that we're running on bear metal.  

So, like before in the Raspberry Pi, developing on the STM32DISCOVERY was made possible by the work of the book authors. Already I was given a project setup that takes care of the details of setting up the ability to flash programs onto the board, debug using the on-chip debugging, and capture log and error messages on my development machine.  

My goal for the weekend was to hook up and read from the SHT31D.First, I wanted to see a running example of the board reading from an I2C interface, the communication protocol that my sensor happened to implement. Luckily, the Discovery Book had an example of reading its on board Magnometer through I2c. After a few tweaks to get the example working for my newer revision of the board, I got some readings.

Second goal: read from my colleague's Bosh BME280. Luckily, this was done by pulling down the crate. However, there was an issue: the original implementer had the factory function for the sensor object take a delay object. This meant I could no longer use the delay object (to say, pause between readings). I was perplexed, and I saw a pull request to solve the issue that had not been merged in a year. Luckily, cargo provides an easy way to bring in the code of this PR straight from Github. 

Third goal: Use the SHT31D. Straightforward. I may have gotten some problems because my wire had a stubby connector end that did not fit in the bread board. After accidentily breaking the end, I stipped the wire and added a proper connecter from a wire. Once it panicked because the connection was not good. Just pushing it in a little made the program run.

On Sunday, having accomplished my goal twice over, and with not much time left, I decided to answer a question that seemed solvable with my remaining time: could I read from both sensors from at the same time?

Reading about I2c buses, hardware-wise, this seemed totally doable. Would the embedded-rust ecosystem allow it?

At first glance: no. Bringing in both sensors in a program, both required the i2c bus object. Only one could have it, else I get the error: ''. 

Luckily spotted in a github thread: the shared-bus crate. This one could manage the sharing of the bus.

That was my weekend! Had a blast, looking forward to the next one.

