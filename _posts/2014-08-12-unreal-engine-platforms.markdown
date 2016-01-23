---
layout: page
title:  Making moving platforms in Unreal Engine 4
date:   2014-08-12 00:00:00
permalink: unreal-platforms
---

<div class="row text-center"><img src="/data/ue_logo.png" class="margined20"/></div>

When Epic Games introduced their new type of license for the Unreal Engine 4 (the so-called EaaS, i.e. "engine as a service"), it was only a matter of time for me to start testing and using the engine.
To be honest, I had certain difficulties understanding some basic concepts of various things that make up the engine editor. Most importantly, the blueprints. It is indeed a very powerful system, and judging by
various tutorials that float around the Internet (including the official ones), this system can do almost anything that you previously needed to write code for.

<!--break-->

The main drawback for me, however, was that there's not much documentation out there that covers HOWTOs for a number of common basic tasks that almost every game developer deals with.
The most comprehensive tutorials are represented as videos, so you cannot simply google your problem and get a bunch of links to simple and clear answers.
Most of the time it's watching videos and gathering pieces of information from them. The situation is quite reasonable, because the engine is new, the community is not very mature,
so it has to pass some time before all basic questions will get their answers.

In this post I'll present a solution for a common task that I had an extremely hard time figuring out how to solve: a platform that moves automatically from point A to point B, then to point A, then back again, and so on
(important note: it doesn't need any external signal to switch direction). If you google "unreal engine moving platform" or "unreal engine lift" or whatever similar,
you will get a number of links to articles and posts telling you how to make a platform that switches movement direction only when a certain trigger event occurs (for example,
the player steps on a button on the floor). That is, not automatically - not what I actually wanted to achieve.

My solution requires basic understanding of concepts such as matinee and level blueprint, so if these words are new to you, I recommend reading the [official wiki](https://wiki.unrealengine.com/Main_Page)
on these topics. Here is how the solution looks in the level blueprint editor:

<a class="thumbnail lightbox" rel="gallery" href="/data/platform_blueprint.png" target="_blank">
	<img src="/data/platform_blueprint.png" class="margined20"/>
</a>

These nodes have to be placed in the level blueprint. I wanted to maked them a separate component (a reusable blueprint), but failed to figure out how to do that. So, for every platform you have in the scene,
similar nodes should be created. Not an elegant solution, but at does the job.

How it works. For a platform actor we set up the matinee object that represents movement from point A to point B. When a matinee object triggers its Finished event, the event goes through the
FlipFlop operation, which alternates between the two outputs, when executed. When the first output is produced, we call
the Reverse function on the matinee. When the second output is produced, we execute the Play function, correspondingly. This causes the matinee to play animation in normal direction first, then reverse the direction when it is first
finished, then switch the playback direction back to normal, when it's finished again, and so on. Easy enough when you know what functions are available to you (like Play and Reverse), and when and how they can be called.
The FlipFlop flow control operation also has not very intuitive name, making it difficult to locate it when you're not sure how the whole process that you need to implement in called.

If you have any other solution of this problem, please share it in the comments below, so that we can discuss, compare and learn.
