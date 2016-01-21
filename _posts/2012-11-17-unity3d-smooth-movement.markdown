---
layout: page
title:  Unity3d tip&#58; smooth accelerated GameObject movement
date:   2012-11-17 00:00:00
---

To make a GameObject smoothly translate from one point to another (accelerating first, then decelerating),
the easiest way is to use [Vector3.SmoothDamp](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.SmoothDamp.html) method.
This method produces exactly that effect and does not require you to write those awkward auxiliary field variables in your class (various timers, accelerations, speeds, etc.),
which you probably used when trying to simulate this kind of movement.

One use of this method could be a camera following an object. Each frame you update camera position using the described method first, then LookAt the target:
{% highlight csharp %}
Vector3 unused;
camera.transform.position = Vector3.SmoothDamp(camera.transform.position, targetTracker.transform.position, ref unused, 0.3f, float.MaxValue, Time.deltaTime);
camera.transform.LookAt(target.transform);
{% endhighlight %}

Here the `targetTracker` is a child object of the target object, having some non-zero local coordinates.
This object acts as an attractor for the camera, making it move towards it all the time until it reaches the tracker position.