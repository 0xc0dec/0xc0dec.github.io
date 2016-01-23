---
layout: page
title:  Unity3d tip&#58; spectator and FPS camera without overturning
date:   2012-11-28 00:00:00
permalink: unity3d-spectator-camera
---

It’s pretty simple to code a camera to move in a spectator-like (and FPS) fashion. The
only trick you should do to make this movement production-ready is to restrict camera movement around the horizontal axis.

Basically, spectator camera rotates either "left-right" (around global vertical Z axis), or "up-down"
(around its local X axis). This local X axis is always horizontal, i.e. lies in XZ global plane. When a camera
rotates around this axis, the player looks up or down. This leads to a camera overturning problem when he tries to look
upper and upper, or lower and lower. All games you’ve seen so far restrict this movement, so that the camera can only look
vertically up or vertically down in the "worst case".

<!--break-->

Below is the code I use in my projects for a spectator and FPS cameras. It’s not ideal, because it restricts vertical
rotation approximately, so that the camera stops rotating when it almost reached the vertical axis. Approximation amount is adjustable. Anyway, it does the job
{% highlight csharp %}
using UnityEngine;

public class FreeLook : MonoBehaviour
{
    public float rotationSpeed = 0.05f;
    public float normalSpeed = 10;
    public float highSpeed = 20;

    private bool _ownCursor = false;

    private void Start()
    {
        if (!Screen.lockCursor && Screen.showCursor)
        {
            Screen.lockCursor = true;
            Screen.showCursor = false;
            _ownCursor = true;
        }
    }

    private void OnDisable()
    {
        if (_ownCursor)
        {
            Screen.lockCursor = false;
            Screen.showCursor = true;
        }
    }

    private void Update()
    {
        Vector3 dp = Vector3.zero;

        if (Input.GetKey(KeyCode.W)) dp.z = 1;
        if (Input.GetKey(KeyCode.S)) dp.z = -1;
        if (Input.GetKey(KeyCode.A)) dp.x = -1;
        if (Input.GetKey(KeyCode.D)) dp.x = 1;
        if (Input.GetKey(KeyCode.Q)) dp.y = -1;
        if (Input.GetKey(KeyCode.E)) dp.y = 1;

        var speed = normalSpeed * (Input.GetKey(KeyCode.LeftShift) ? 5 : 1);

        dp.Normalize();
        dp *= speed * Time.deltaTime;
        camera.transform.Translate(dp.x, dp.y, dp.z, Space.Self);

        float rotY = rotationSpeed * Input.GetAxis("Mouse X");
        float rotX = -rotationSpeed * Input.GetAxis("Mouse Y");

        float toTop = Vector3.Angle(camera.transform.forward, Vector3.up);
        if (rotX < 0)
            rotX = -Mathf.Min(Mathf.Abs(rotX), Mathf.Max(0, toTop - 20));
        else
            rotX = Mathf.Min(Mathf.Abs(rotX), Mathf.Max(0, 160 - toTop));

        camera.transform.RotateAround(Vector3.up, rotY);
        camera.transform.RotateAround(camera.transform.right, rotX);
    }
}
{% endhighlight %}
