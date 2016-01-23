---
layout: page
title:  Unity3d tip&#58; implementing pick-and-drag mechanics in a first-person game
date:   2014-09-07 00:00:00
permalink: unity3d-pick-and-drag
---

<div class="row text-center"><img src="/data/companion_cube.png" class="margined20"/></div>

Picking and dragging objects around is something that you can see quite often in various games with the first-person camera view.
This features adds a lot to the level of interactivity with the game environment, so developers build their game mechanics around
this feature for a good reason.

So, how to implement this inside Unity? I've seen some implementations ranging from the very simple to the more advanced ones. In this post
I'll try to explain what I don't like about some of them and propose my own solution with an interesting hack behind it.

<!--break-->

Lets assume that we have a camera object in the scene. It doesn't matter how the camera is controlled, in my case it is attached to the player object.
Let's say we want, while looking through this camera, to pick objects, hold them in front of the camera, carry them around while the camera is moving,
then put them to the ground or throw away. Assume picked objects to be rigid bodies. Here and throughout the rest of this article I will refer to the point to which
objects are attached to while moving as the pivot point.

__Solution 1__: calculate pivot point offset each frame and move the picked object by that offset.

Pros:

* Easy to implement.

Cons:

* Direct manipulation with the rigid body position may lead to the physics simulation instability and artifacts. It's always a good rule to move (non-kinematic) rigid bodies by applying forces.

__Solution 2__: make object be a child of the camera.

Pros:

* Easy to implement.

Cons:

* Same as for solution 1. (Non-kinematic) Rigid bodies should only be controlled by forces and not by direct position manipulations, which is what parent-child relationship actually does.

__Solution 3__: use joints. Construct a joint and attach one end of it to the rigid body, and the other end to the pivot point.

Pros:

* "Physically correct" way to control the rigid body. The body follows the pivot point and is able to react to collisions.

Cons:

* Object reaction to camera movements is laggy and twitching (lag amount depends on the joint type).
* Object starts to jump around the pivot point when camera makes sudden movements (true for spring joints, can be fixed by setting a high dumping value).
* Difficult to adjust the joint parameters to achieve somewhat of a believable result.

The solution utilising joints is the most complex of the three, but after a good tuning it produces believable results. Its main flaw for me, however, was the twitching object movement.
When you drag a body that is attached to the pivot point with a fixed joint, the body is supposed to strictly preserve its position relative to the pivot point. The thing is that
physical joints have "springy" nature (even if they are not spring joints), so the body will always behave a bit strange at high movement rates, twitching and slightly moving around the pivot point.
What I want is to have the picked object be directly at the same spot all the time when it is dragged.

## Solution
My solution doesn't rely on joints. Instead, it moves the body by changing its velocity, which is also an acceptable way of moving rigid bodies.
It also implements a hack that allows you to completely eliminate the body movement relative to the pivot point. You can see the final result in this video:

<div class="row text-center">
	<iframe class="margined20" width="640" height="360" src="//www.youtube.com/embed/zM_o6zB9B5s" frameborder="0" allowfullscreen></iframe>
</div>

The hack is to drag _two_ objects instead of one. The first object is the visual representation (avatar) of the initial body. It has no RigidBody component and is moved by directly manipulating its position.
The second object is not visible at all and represents the physical aspect of the body, having a RigidBody component attached to it. Fast camera movements might somehow
make it behave oddly, but the player doesn't see it anyway. There's no need to perform complex tweaking of the joint to make results look good (which is extremely hard from what I've experienced).
The only tricky part is when the physical avatar collides with other objects in the scene. When this occurs, we just align the visual avatar with the physical one so that both of them react to collisions.
When nothing happens, the visual avatar moves with the camera.

Below is the code with some explaining comments:
{% highlight csharp %}
public class HeldObject : MonoBehaviour
{
	public bool IsColliding { get; private set; }

	private void OnCollisionEnter(Collision collision)
	{
		IsColliding = true;
	}

	private void OnTriggerStay(Collider other)
	{
		IsColliding = true;
	}

	private void OnCollisionExit(Collision collision)
	{
		IsColliding = false;
	}
}


public class PlayerPick : MonoBehaviour
{
	// Maximum distance from the camera at which the object can be picked up
	public float MaxPickDistance;
	public float ThrowStrength = 10;
	public float HeldObjectFollowStrength = 50;

	private Player _player;
	private Camera _playerCam;
	private GameObject _pivot;

	private HeldObject _heldObject; // physical avatar
	private GameObject _heldBodyAvatar; // visual avatar
	private float _heldBodyAngularDrag;

	private RaycastHit? _raycast;

	private void Start()
	{
		_player = Player.Instance;
		_playerCam = Player.Camera;
		_pivot = Player.PickPivot;
	}

	private void Update()
	{
		Raycast();
		if (Input.GetKeyDown(KeyCode.E))
		{
			if (!_heldObject)
				TakeObject();
			else
				ReleaseObject();
		}
		HoldObject();
		if (Input.GetMouseButton(0) && _heldObject)
			ThrowObject();
	}

	private void Raycast()
	{
		_raycast = null;
		const int layerMask = 1 << 8;
		var raycastHits = Physics.RaycastAll(_playerCam.transform.position, _playerCam.transform.forward,
											 MaxPickDistance, ~layerMask);
		foreach (var hit in raycastHits)
		{
			if (hit.collider == _player.collider || !hit.collider.rigidbody) // avoid colliding with the player object itself
				continue;
			_raycast = hit;
		}
	}

	private void HoldObject()
	{
		if (!_heldObject)
			return;
		// Drag the physical avatar by changing its velocity
		_heldObject.rigidbody.velocity = HeldObjectFollowStrength * (_pivot.transform.position - _heldObject.transform.position);
		// When the physical avatar collides, we move the visual one to the same position. When this doesn't happen,
		// move the visuals back to the pivot point.
		_heldBodyAvatar.transform.position = _heldObject.IsColliding
			? _heldObject.transform.position
			: _pivot.transform.position;
		// If the physical avatar is colliding with something, change visuals rotation to correspond
		if (_heldObject.IsColliding)
			_heldBodyAvatar.transform.rotation = _heldObject.transform.rotation;
	}

	private void TakeObject()
	{
		if (!_raycast.HasValue)
			return;

		var heldBody = _raycast.Value.rigidbody;
		heldBody.transform.position = _pivot.transform.position;
		heldBody.useGravity = false;

		_heldBodyAngularDrag = heldBody.angularDrag;
		heldBody.angularDrag = 1; // so that the object doesn't continue rotating after each collision
		heldBody.renderer.enabled = false;
		_heldObject = heldBody.gameObject.AddComponent<HeldObject>();

		_heldBodyAvatar = new GameObject("Held body avatar");
		_heldBodyAvatar.transform.parent = _playerCam.transform;
		_heldBodyAvatar.transform.position = _pivot.transform.position;
		_heldBodyAvatar.transform.rotation = heldBody.transform.rotation;
		_heldBodyAvatar.transform.localScale = heldBody.transform.localScale;
		_heldBodyAvatar.AddComponent<MeshFilter>().sharedMesh = heldBody.gameObject.GetComponent<MeshFilter>().sharedMesh;
		_heldBodyAvatar.AddComponent<MeshRenderer>().sharedMaterial = heldBody.gameObject.renderer.sharedMaterial;

		Physics.IgnoreCollision(heldBody.collider, Player.Controller, true);
	}

	private void ReleaseObject(Action onRelease = null)
	{
		Physics.IgnoreCollision(_heldObject.collider, Player.Controller, false);
		_heldObject.rigidbody.useGravity = true;
		_heldObject.rigidbody.angularDrag = _heldBodyAngularDrag;
		_heldObject.rigidbody.transform.position = _heldBodyAvatar.transform.position;
		_heldObject.rigidbody.transform.rotation = _heldBodyAvatar.transform.rotation;
		_heldObject.rigidbody.renderer.enabled = true;
		_heldBodyAvatar.renderer.enabled = false;
		Destroy(_heldObject);
		Destroy(_heldBodyAvatar);
		if (onRelease != null)
			onRelease();
		_heldObject = null;
	}

	private void ThrowObject()
	{
		ReleaseObject(() => _heldObject.rigidbody.AddForce(_playerCam.transform.forward * ThrowStrength, ForceMode.Impulse));
	}
}
{% endhighlight %}

This algorithm could probably be improved by, for example, simplifying the `TakeObject` method. If the body that has been picked has complex visuals and a lot of components on it,
it might be easier not to copy everything into a new visual avatar. Instead, you can leave the visuals on the object, and just create a new physical avatar. In my case
the object itself remains physical, and all visuals go to the new object.
