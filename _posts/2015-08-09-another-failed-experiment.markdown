---
layout: page
title:  Another failed experiment
date:   2015-08-09 00:00:00
permalink: hypermino
---

It's time for me to accept that I've failed again.

<div class="row text-center"><img src="/images/hypermino/screenshot1.png" class="margined20"/></div>

During the past week I've been working on a game prototype in which I implemented a gameplay idea that came to my mind recently.
Today I realised that this gameplay is no longer fun to me. When I first started, it was interesting to quickly prototype the idea
to get how it feels, looks and works. Many indiedevs are correct saying that you need to make a working prototype as soon as you can,
scrapping away all the stuff that doesn't add to the gameplay. This basic prototype will show whether the game is fun to play or not, and maybe it would also
give you some ideas on how to make things even more fun.

<!--break-->

In my case the prototype convinced me that the idea is not worth spending next few months finishing and polishing it.

<div class="row text-center"><img src="/images/hypermino/2.gif" class="margined20"/></div>

The GIF pretty much explains what I'm talking about. As it usually happens in indie games, the main game elements are once again cubes.
The goal of the game is to roll a figure made of cubes and attach more cubes to it so that no cubes are left on the game field.
I wanted to make several win conditions in the game, for example:

* Assemble a figure of any shape.
* Assemble a figure of a predefined shape.
* Any of the previous plus an additional requirement: in the final shape all special cubes (yellow ones) must be neighbours to each other.

After testing it turned out that the gameplay is quite boring. The game doesn't require any analytical thinking or strategic planning, but rather
a bruteforce approach and some memorizing of what moves are wrong so that you don't make them again. The only interesting element of the game is how
the figure behaves when rolling. Notice that some parts of it "sink" below the ground level depending on the figure shape. Didn't help anyway :)

<div class="row text-center"><img src="/images/hypermino/1.gif" class="margined20"/></div>

To wrap this post up I would say that coding this prototype was a great experience regardless of the fact that the idea didn't evolve into a finished game.
Always implement the core mechanics ASAP and don't waste your time on things that you think won't be fun.
