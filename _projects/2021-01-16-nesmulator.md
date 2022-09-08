---
layout: article
title: Nesmulator
mathjax: false
show_subscribe: false
cover: "assets/projects_cover/nesmulator.png"
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: "#FFFFFF"
  background_image:
    gradient: "linear-gradient(135deg, rgba(100, 100, 10 , .4), rgba(100, 10, 10, .4))"
    src: "assets/projects_cover/nesmulator.png"
---

A simple Nintendo Entertainment System (NES) emulator written in the Rust programming language. The purpose of this emulator was for me to learn how the NES works and to challenge myself.

<!--more-->

`Rust`{:.info} `Emulation`{:.info}

I always wondered how emulators worked. This is why I decided I should write my own NES emulator from scratch to finally get the answers I was looking for! I decided not to follow a step by step tutorial and to use the resources I found online to really understand how the NES run games and build an emulator for it. I chose to use the Rust programming language because I wanted to learn it and it was a big challenge to both learn Rust and emulation technics.

I separated the emulator in two repositories because I wanted a GUI that is not linked with the actual emulation.

Feel free to check my emulator on Github: [nesmulator-core](https://github.com/AntoineRR/nesmulator-core), [nesmulator-gui](https://github.com/AntoineRR/nesmulator-gui)