---
layout: article
title: Raytracer
mathjax: false
show_subscribe: false
cover: "assets/projects_cover/raytracer.png"
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: "#FFFFFF"
  background_image:
    gradient: "linear-gradient(135deg, rgba(100, 100, 10 , .4), rgba(100, 10, 10, .4))"
    src: "assets/projects_cover/raytracer.png"
---

A simple raytracer program written in Rust. It is adapted from the C++ tutorial "Raytracing in one weekend" book series.

<!--more-->

`Rust`{:.info} `Raytracing`{:.info}

The [Raytracing in one weekend](https://raytracing.github.io/) book series is a really good start for learning about raytracing. It helps building a program that generates images with raytracing technics. This book gives the code for a C++ application, and I rewrote the code (and more!) in Rust. There is a Rust version of this book but I didn't use it for this project.

The final product is a crate one can use to setup a scene a generate an image of this scene from a specific point of view. I applied multithreading and optimization algorithms to have a fast render.

The code is available on Github: [Raytracer](https://github.com/AntoineRR/raytracer)