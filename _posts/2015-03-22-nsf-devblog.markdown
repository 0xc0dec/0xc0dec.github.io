---
layout: page
title:  Not So Flat devblog
date:   2015-03-22 13:00:00
permalink: nsf-devblog
---

<div class="row text-center"><img src="/data/notsoflat/player.png" class="margined20"/></div>

<a href="/nsf-devblog2">Part 2</a>

Not So Flat is the name of the new game that I'm currently working on. When I first started the development (late December 2014),
it's been already more than half a year since my previous game <a href="/cubicroll/">Cubic Roll</a> came out. At that time I was struggling to come up with
a unique gameplay idea that I've never seen before, but which could still be quite feasible to implement within an acceptable time bounds.

In search of inspiration I was looking through my old prototypes and found one that was dated back to the beginning of 2013.
The idea was quite interesting but at the time the demo was made I had lost my inspiration because I wasn't able to implement the
concept in a way that I could be satisfied with (that was technically a bit challenging). Looking at that demo again I've decided to give it
another try and see what happens this time. I had no ideas better than the one already implemented, so the decision to continue the project was easy to make.

<!--break-->

Not So Flat is a game about a square-shaped character wandering in a schematic world of figures and lines. At first the game might look like an ordinary
2D platformer, but since I'm not really into the 2D games, NSF is made as something more than simple 2D.
Instead of being a flat surface with obstacles on it, each level is rather represented by a shape made of orthogonal planes,
through which the character can travel almost seamlessly. While he resides within a single plane, the character moves and interacts with the surroundings
just like in other platformers. When he moves between the planes, however, the third dimension comes into action, allowing some unconventional puzzles to be implemented.

<div class="row text-center"><img src="/data/notsoflat/1.gif" class="margined20"/></div>

The first big challenge for me was to implement the character movement and interaction with the environment. The amount of nuances was just incredible!
At first, I didn't expect the algorithm to become that complicated, but still there's hardly a solution easier than the one that I came up with. With each level I started making, new bugs
always inevitably came out because the level geometric configuration progressed in difficulty from level to level. It was simply impossible to foresee all the conditions in which
the character might find itself in the next level. I believe that this topic deserves a separate blog post, so I won't go into the details here.

<div class="row text-center"><img src="/data/notsoflat/2.gif" class="margined20"/></div>

The second great deal of work was (and still is) required by the very levels. There are four currently and the rate at which I make them impresses me with its slowness.
Each new level not only introduces a bunch of new bugs that are difficult to reproduce, but also makes it harder to actually design the level itself.
Since the game mechanics permits it, I try to make multiple uses of some elements in a level. For example, if there is an immovable block acting like a platform,
I might want to plan the character path to follow through this block several times. At first time the character might encounter this platform while moving from left to right, jumping onto
one of the sides of the platform. As the path continues, the character might come across this block again while moving in the orthogonal direction, and this time the active side of the block would be different.
This trick is used a lot, thereby the design process gets more complicated. When I add another block to the level I have to keep an eye on the existing geometry and try not to break the existing character paths, nor make some locations reachable easier that they were designed previously.

<div class="row text-center"><img src="/data/notsoflat/3.gif" class="margined20"/></div>

The graphics in NSF had one major revamp since the game started four months ago. In first versions is was all white and black, with white being the primary color and black being the color of the elements like platforms, level borders and the player. After the revamp the game got its current look, featuring the dark blue background, yellow and green glowing elements and UI, plus some other, less represented colors.
In future this might change as well, but for now I'm OK with this appearance.

In conclusion, I can say that I'm still motivated in the development of this game, so it's likely to be finished (not sure when but ASAP). In future posts, if there would be any, I'll make an attempt to describe various aspects of the game like character movement or materials and shaders being used.
Stay tuned!
