---
layout: page
title:  Unity3d tip&#58; tasks concept
date:   2013-08-10 00:00:00
permalink: unity3d-tasks
---

You are coding a game with Unity3D. What do you normally do when you need to perform some non-instant, continuous action, which will last for a certain
amount of time (during a number of frames)? For example, you need to move an object 10 units forward. I bet you write a special component that will be
responsible for this particular action. Then you add this component to that object and fill the necessary parameters:
{% highlight csharp %}
var action = obj.AddComponent<SomeAction>();
action.duration = 2;
action.speed = 3;
// and so on...
{% endhighlight %}

<!--break-->

You could go further and write a static factory method in SomeAction class, which will do all that stuff and keep your code cleaner:
{% highlight csharp %}
var action = SomeAction.Create(duration, speed);
// ...
{% endhighlight %}

Okay, you’ve done it. But soon you come across another problem which could be solved by this component, but the component must be slightly
altered to fit the new needs. For example, you’ve always moved your object in the same direction, but now you want to set an arbitrary movement
direction. Well, you add another parameter, update factory method usages and code happily again… Until there comes another component tuning. Now you
need your code to perform an action, wait for it’s completion and do something afterwards. You introduce callbacks…

At some point you realise that you’d better make another component and have two, suitable for different small tasks. The number of such components grows, and soon you
begin to drown in a pool of tiny task-oriented classes. I didn’t even mention names yet. It can be next to impossible to give those classes short and comprehensible names.

For me I found a simple solution to the problem of a large number of small task-oriented components. I wrote a single component
Task capable to run any code I want. It can be used as follows:
{% highlight csharp %}
// wait 3 seconds and play sound
Task.Create(3).EndCallback(() => audioSource.Play()).Run();
// Can also be accomplished with StartCoroutine + yield statement, but hey, this is one line! :)

// material color transition from black to white in a second
Task.Create(1).UpdateCallback(time => material.color = new Color(time, time, time, 1)).Run();

// or anything
Task.Create(10)
    .StartCallback(() => Debug.Log("I started"))
    .UpdateCallback(time => Debug.Log(time))
    .EndCallback(() => Debug.Log("Done"))
    .Run();
{% endhighlight %}

In a nutshell the Task class simply creates en empty host object “Tasks” (if it does not exist yet), adds Task component to that object and fills its parameters.
From that moment the new task knows how long to run and what action to perform. You can also subclass the Task class to incapsulate more complex logic and use it like this:
{% highlight csharp %}
// make scene fade to black in 3 seconds
Task.Create<FadeOut>(3).Run();
{% endhighlight%}

Tasks class should not be misused. When you feel that action you’re going to code will be run only from one place in your surrounding application code, it’s OK to use the inline version
{% highlight csharp %}
Task.Create(1)
	.UpdateCallback(time =>
	{
		// do something
		// and more
		// and even more...
	})
	.EndCallback(() =>
	{
		// react to task completion
	}
	.Run();
{% endhighlight %}

But as soon as you need some action to be run from several places, you should go the classic way and create a separate component. Do not copy-paste you code! Subclass the Task class
instead, and you will be provided with a convenient way to run it:
{% highlight csharp %}
public class CustomTask: Task
{
	protected override void UpdateImpl()
	{
		Debug.Log(_time);
		Debug.Log(_duration);
	}

	protected override void StartImpl()
	{
		Debug.Log("Started");
	}

	protected override void EndImpl()
	{
		Debug.Log("Stopped");
	}
}

Task.Create<CustomTask>(3)
	.UpdateCallback(time => Debug.Log("Whatever") // custom tasks also allow you to use callbacks!
	.EndCallback(() => { /* ... */ })
	.Run();
{% endhighlight %}

Below is the code of my current Task class version. Use it freely if you find it useful.
{% highlight csharp %}
using System;
using UnityEngine;

public class Task : MonoBehaviour
{
	protected float _duration;
	protected float _time;

	private Action _start;
	private Action _update;
	private Action _end;
	private bool _started;
	private bool _autoDestroy;

	private static GameObject _host;

	public static Task Create(float duration, bool autoDestroy = true)
	{
		InitHost();
		var job = _host.AddComponent();
		job._duration = duration;
		job._autoDestroy = autoDestroy;
		job._started = false;
		return job;
	}

	public static T Create<T>(float duration, bool autoDestroy = true) where T : Task
	{
		InitHost();
		var task = _host.AddComponent<T>();
		task._duration = duration;
		task._autoDestroy = autoDestroy;
		return task;
	}

	public Task StartCallback(Action callback)
	{
		_start = callback;
		return this;
	}

	public Task UpdateCallback(Action<float> callback)
	{
		_update = callback;
		return this;
	}

	public Task EndCallback(Action callback)
	{
		_end = callback;
		return this;
	}

	public void Run()
	{
		_started = true;
		if (_start != null)
			_start();
		StartImpl();
	}

	protected virtual void UpdateImpl()
	{
	}

	protected virtual void StartImpl()
	{
	}

	protected virtual void EndImpl()
	{
	}

	private static void InitHost()
	{
		if (!_host)
			_host = new GameObject("Tasks");
	}

	// ReSharper disable UnusedMember.Local
	private void Update()
	// ReSharper restore UnusedMember.Local
	{
		if (!_started)
			return;
		if (_update != null)
			_update(_time);
		UpdateImpl();
		// Tasks having _autoDestroy == false will continue running after _time >= _duration,
		// but their _time will stop increasing
		if (_time >= _duration)
			return;
		_time += Time.deltaTime;
		if (_time >= _duration)
		{
			if (_end != null)
				_end();
			EndImpl();
			if (_autoDestroy)
				Destroy(this);
		}
	}
}
{% endhighlight %}
