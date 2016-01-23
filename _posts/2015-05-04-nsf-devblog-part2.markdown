---
layout: page
title:  Not So Flat devblog, part 2
date:   2015-05-04 20:00:00
permalink: nsf-devblog2
---

<div class="row text-center"><img src="/data/notsoflat/player2.png" class="margined20"/></div>

<a href="/nsf-devblog">Part 1</a>

Whoa, that did happen! Not So Flat has finally been released and is <a href="http://0xc0dec.itch.io/notsoflat">available for free on itch.io</a>.

The only game before Not So Flat that also had a devblog about it was ultimately abandonded and never made it to the release.
I was afraid that Not So Flat would suffer the same fate, but things turned out (almost) very well.

This second part of the devblog is aimed at explaining the character movement implementation.
This includes two major topics: the actual movement (running and jumping) and the calculation of the space available for movement.

<!--break-->

### Distances calculation

Each frame, in order to calculate and apply a movement vector (position delta), I need to know how much room for the movement there is.
To get this information, several rays are cast in all four directions from the character.
For example, to calculate distance to the nearest obstacle below the character I cast three rays: one from the center of the bottom edge of the character, and two from the lower left and right corners.
Having three distances calculated, the final distance in this case would be the minimum of the three.
Three rays represent the character geometric dimensions and allow it, for example, to stand on a platform by touching it with a small area of the bottom edge:

<div class="row text-center"><img src="/data/notsoflat/screenshot1.png" class="margined20"/></div>

The three-ray approach, however, has its flaws.
For example, if I made a platform that has a width of, say, 1/4 of the character width, the character would fall through it once the platform is placed right between the casted rays.
For this reason, there are no platforms in the game that are narrower than the half of the character width.

Distance calculation is also required for determining whether or not the character should be wrapped around the corner.
For example, a ray that is cast from the middle of the left character edge in the right direction can be used to calculate the distance between the character center and the nearest wall on the right.
If the resulting distance is less than the character half-width then the character mesh gets adapted to fit into the corner.
This whole procedure is quite complex and has many nuances, so I won't go into further details here. An in-depth description probably deserves a separate blog post. Just to give you an idea of how many
rays are involved:

<div class="row text-center"><img src="/data/notsoflat/screenshot2.png" class="margined20"/></div>

### Running and jumping

When I was approaching the character movement implementation I knew beforehand that I want the jumps to have two main features:

* a bit of control in the air
* variable height

There was no special reason for this, the game wasn't built around these particular features. That was just my personal preference - I feel it very satisfying to have the ability to change movement direction while in the air.

The process can be split into two parts. The first part is responsible for the horizontal component of the movement. It is the easiest of the two and is implemented as follows:
{% highlight csharp %}
private const float groundAcceleration = 10f;
private const float airAccelerationX = 5f;
private const float maxSpeedX = 4;
...

var accelerationX = groundAcceleration;
if (ContinueJumping())
	accelerationX = airAccelerationX;
if (MoveLeft ^ MoveRight)
	targetSpeedX = MoveRight ? maxSpeedX : MoveLeft ? -maxSpeedX : 0;
else
	targetSpeedX = 0;
speedX = Mathf.Lerp(speedX, targetSpeedX, Time.deltaTime * accelerationX);
dx = Time.deltaTime * speedX;
{% endhighlight %}
First we define the current horizontal acceleration (`accelerationX`). If the character is in the air (i.e. is jumping, so that the ContinueJumping() function returns `true`), the acceleration gets a smaller value, which is exactly what "a bit of control in the air" stands for. The constants are adjustable, so I can make the character be super-controllable in the air as well as make it not controllable at all.

After we're done with the acceleration, we calculate the new horizontal speed `targetSpeedX` that the character tries to reach. Since the acceleration is finite, the character doesn't reach this speed at once - thus the name. `MoveLeft` and `MoveRight` flags indicate the directions in which the player wants to move the character (simply speaking, they indicate whether or not the left and right cursor keys are being pressed). The `targetSpeedX` can only have three values: 0, `-maxSpeedX` and `maxSpeedX`, which means that the character is either moving left, right, or is stopping horizontal movement at all.

The final step of the process is the calculation of the actual speed `speedX`. The is done fairly simple, by calling the Lerp function, which already performs all range checks and won't allow the speed value to go beyond the range `[-maxSpeedX, 0]` or `[0, maxSpeedX]`. Having the final speed value, we calculate the horizontal position delta `dx`.

The vertical component of the movement is a little bit trickier. The main difficulty comes from our wish to allow jumping by a variable height: the character jumps higher as the player presses the button longer.
Let's look at the code:
{% highlight csharp %}
private const float maxDownSpeed = 12;
private const float jumpExtraGravityTime = 0.5f;
private const float jumpAccelerationDuration = 0.2f;
private float jumpTime = jumpExtraGravityTime;

...

private void StartJumping()
{
	if (!grounded && !jumping)
		canJump = false;
	if (!Jump && grounded)
		canJump = true;
	if (Jump && !jumping && canJump)
	{
		canJump = false;
		jumping = true;
		jumpAccelerationDone = false;
		grounded = false;
		jumpTime = 0;
		speedY = maxUpSpeed;
		...
	}
}

...

private bool ContinueJumping()
{
	if (jumping)
	{
		jumpTime += Time.deltaTime;
		if (!Jump || jumpTime >= jumpAccelerationDuration)
			jumpAccelerationDone = true;
	}
	return jumping;
}

...


if (ContinueJumping())
...

if (jumpAccelerationDone)
{
	var acceleration = gravityAcceleration;
	if (!Jump)
		acceleration *= 2 - Mathf.Min(1, jumpTime / jumpExtraGravityTime);
	speedY = Mathf.Max(speedY - Time.deltaTime * acceleration, -maxDownSpeed);
}
dy = Time.deltaTime * speedY;
{% endhighlight %}
The idea here is as follows. When the jump starts, we instantly set the character vertical speed to a certain maximum (as many other games do). We also keep track of how long the jump lasts (`jumpTime`). The first jump phase is a special acceleration phase, which lasts for `jumpAccelerationDuration`. During this phase we don't apply gravity, so the character keeps moving up with a constant maximum speed. This phase ends when either of the two events occur: 1) the `jumpTime` exceeds the `jumpAccelerationDuration` 2) the player stops pressing the jump button.

Once the acceleration phase ends, we start applying the gravity. The trick here is that the gravity value depends on the duration of the acceleration phase. If the player has been pressing the jump button for a very short amount of time, the gravity will be stronger than it would be for a longer jump. This tuning is done in the line
{% highlight csharp %}
acceleration *= 2 - Mathf.Min(1, jumpTime / jumpExtraGravityTime);
{% endhighlight %}
I found that variable gravity, while being not realistic in theory, produces more enjoyable results in practice. As always in gamedev, I had to playtest a lot to figure out the implementation that works best for me.

Once the two components of the position delta vector are calculated, the last thing we do is determine if there's enough space left to apply this delta, and if not, we adjust it so that the character doesn't go through walls and other obstacles. That's it!

### Conclusion
This post covers briefly the basic ideas behind character movement in Not So Flat. The material ended up being not very detailed, but I believe that the part about jumping, which I find to be the most useful, explains the topic well. There are a few other topics that are probably worth covering, but currently I don't feel the need to do so. Time to work on a new game! :)
