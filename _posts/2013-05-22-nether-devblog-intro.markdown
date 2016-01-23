---
layout: page
title:  Project Nether devblog. Intro
date:   2013-05-22 00:00:00
permalink: nether-devblog
---

I’ve decided to start a series of posts describing how things are going on with the development of my current game projects.
I hope this activity will motivate me not to abandon those projects and, probably, receive some feedback from readers.

This first post is about my new game called **Nether**. Currently it’s a working title and might change in future.
Meanwhile it’s a beautiful word, which somehow describes the game look and feel, and is also easy to print in
terminal when doing some work with Git, `cd`ing, `ls`ing and so on, you know.

<!--break-->

**Nether** is not about story, it’s about action. You play an abstract “spaceship” flying in an abstract world,
everything is clean and simple. Your purpose is to collect strange artifacts (lets call them *gems*) which fly around.
These gems comprise a path that player must follow to complete each level. Not all gems are visible at once, there is
only a small fixed amount of them visible at level startup. The player has to collect gems sequentially, because each
collected gem reveals the next hidden one, thus unveiling the next step on the path. The more gems you miss, the less
is your chance to complete the level. Other challenges include various obstacles and player’s speed that increases from
level to level.

I really wanted to make a simple, fun and intuitive game that will test player’s reaction, coordination and accuracy
of movements. My previous game [Uncopy](/uncopy/) was a puzzle, so now I feel I need to make something that won’t require much brain
activity to play :) Note here that I don’t plan this game to have many levels. It’s going to be hardcore, forcing player
to approach each level many-many times until he completes it.

<div class="row text-center"><img src="/data/nether_gameplay.png" class="margined20"/></div>

When making art for my games, I pursue one important purpose: they must look stylish. This is not only cool, but also
has some pragmatic reason to it. If you’re a programmer making game, you basically have two realistic ways to make the
game attractive. You either hire an artist or make all graphics yourself, but make it simple and stylish. Although I do
have some background in 3D modelling and vector drawing, I don’t have much experience in other aspects of artistic work
(for example, making textures), so I choose the second way, because hiring an artist would be pretty expensive for me.

<div class="row text-center"><img src="/data/nether_menu.png" class="margined20"/></div>

Currently **Nether** has about three levels, and I think ten would be enough for the first release. There are no menus yet,
except the very raw draft of the main menu. I’m concentrating on making levels and switch to other activity only when
I find myself stuck at some point without fresh ideas. Changing activity helps in such cases.

So, this was an intro post with brief info about **Nether**, the project I’m currently working on.
In later posts I’ll describe in details various aspects of development process,
including technical problems I face on my way. Stay tuned!
