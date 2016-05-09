---
layout: page
title: Wireframe
name: wireframe
permalink: wireframe/
comments: true
---

<div class="row">
	<div class="col-xs-2"><div class="thumbnail"><img src="/images/wireframe/icon.png" alt="..."></div></div>
	<div class="col-xs-9">
		<p>
			{% include descriptions/wireframe %}
		</p>
		<p>
			<a href="https://www.assetstore.unity3d.com/#/content/11638">Get from Unity Asset Store</a>.
		</p>
	</div>
</div>

### Features

* Six example materials, including single side, two-sided, solid (wireframe + fill color). Wireframe material can be mixed with any other material by simply assigning two
(or more) materials to the same object. This requires at least two draw calls though. For simple wireframe rendering with fill color there is a shader included that requires only one draw call.
* Line width can be changed.
* Lines are calculated in window space and don’t depend on object’s distance from the camera. So, the farther the object is from the camera, the denser its wireframe.
This behavior is the same as for classic wireframe (used, for example, in Unity editor).
* Lines are antialiased.
* Deferred and forward rendering support.

<div class="row">
	<div class="col-xs-12">
		<h2>Screenshots</h2>
		<div class="row text-center"><img src="/images/wireframe/screenshot2.png" class="margined20"></div>
		<div class="row text-center"><img src="/images/wireframe/screenshot4.png" class="margined20"></div>
		<div class="row text-center"><img src="/images/wireframe/screenshot5.png" class="margined20"></div>
	</div>
</div>
