---
layout: article
title: Pokemon GAN
mathjax: false
show_subscribe: false
cover: "assets/projects_cover/pokemon_gan.png"
mode: immersive
header:
  theme: dark
article_header:
  type: overlay
  theme: dark
  background_color: "#FFFFFF"
  background_image:
    gradient: "linear-gradient(135deg, rgba(100, 100, 10 , .4), rgba(100, 10, 10, .4))"
    src: "assets/projects_cover/pokemon_gan.png"
---

I trained a Generative Adversarial Network (GAN) to generate images of new Pokémon. I created an API using Flask and a frontend to interact with the trained model.

<!--more-->

`GAN`{:.info} `Python`{:.info} `Flask`{:.info}

Machine learning is something really interesting I wanted to try. To do so, I decided to implement a Generative Adversarial Network (GAN) using tensorflow and Python based on documentation I found online. After building the model, I trained it using Google Colab with a extended Pokemon images dataset. The results I had were far from perfect but I couldn't train my model longer or with higher resolution images because of technical limitations. In the end, I decided it was good enough to show it off on a website I created.

The website is very simple and uses a Flask backend that loads the model with the trained weights and use it to generate images.

The Pokemon generator website can be found here: [Pokemon GAN](https://random-pokemon.herokuapp.com)