---
layout: page
title: Unity3D Hardware Cursor
name: hwcursor
permalink: hwcursor/
comments: true
---

<div class="row">
	<div class="col-xs-2"><div class="thumbnail"><img src="/images/hwcursor/logo.png" alt="..."></div></div>
	<div class="col-xs-9">
		<p>
		{% include descriptions/hwcursor %}
		</p>
		<p><b>Note:</b> intended to be used with Unity3d versions prior 4.0. Unity v.4.0 has native support for hardware cursor.</p>
		<p><a target="_blank" href="http://u3d.as/3eH">Get from Unity Asset Store</a>.</p>
	</div>
</div>

<div class="row">
<div class="col-xs-12">
	<h2>Installation</h2>
	Copy `HardwareCursor.dll` into your project Assets folder.

	<h2>Usage</h2>
	Create cursor, initialize it from file and set as active:
	{% highlight csharp %}
	using UnityEngine;
	using HardwareCursor;
	///...
	var cursor = Cursor.Create();
	var hotSpotX = 0f;
	var hotSpotY = 0f;
	cursor.InitFromFile("cursor.cur", hotSpotX, hotSpotY); // supports CUR, ICO, PNG
	cursor.SetActive();
	///...
	{% endhighlight %}

	Create cursor, initialize it from an asset and set as active:
	{% highlight csharp %}
	using UnityEngine;
	using HardwareCursor;
	//...
	var data = (Resources.Load("Textures/cursor") as Texture2D).EncodeToPNG();
	var hotSpotX = 0f;
	var hotSpotY = 0f;
	cursor.InitFromMemory(data, hotSpotX, hotSpotY);
	cursor.SetActive();
	//...
	{% endhighlight %}

	Reset to default cursor:
	{% highlight csharp %}
	cursor.ActivateDefaultCursor();
	{% endhighlight %}

	<b>Note:</b> to be able to use texture asset as a cursor image, the texture should be imported with Advanced texture type,
	should have Read/Write enabled set to true and its format set to ARGB 32 bit.

	<h2>Known bugs</h2>
	<ul>
	<li>Does not work in editor, only in standalone builds.</li>
	<li>Under Mac OS X the cursor is reset to default if game resolution is changed in runtime. You can work around this by re-activating the cursor.</li>
	</ul>
</div>
</div>
