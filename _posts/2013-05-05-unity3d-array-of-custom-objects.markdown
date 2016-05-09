---
layout: page
title:  Unity3D tip&#58; edit arrays of custom objects in object inspector
date:   2013-05-05 00:00:00
permalink: unity3d-array-of-custom-objects
comments: true
---

Say I have the following component:
{% highlight csharp %}
public class Test: MonoBehaviour
{
    public Vector3[] PivotPoints;
}
{% endhighlight %}

It’s OK to edit the PivotPoints field in object inspector, because Unity natively understands
`Vector3` struct and displays it properly. Things get more complicated when I add field representing an array of custom objects:
{% highlight csharp %}
class NodeSetup
{
    public Vector3 Position;
    public Vector3 Weight;
}

...

public class Test: MonoBehaviour
{
    public Vector3[] PivotPoints;
    public NodeSetup[] NodeSetups;
}
{% endhighlight %}

Unity ignores field NodeSetups. First I thought I should write a custom editor in order to edit such fields.
Fortunately the problem has simple solution – just mark your custom class with Serializable attribute:
{% highlight csharp %}
[Serializable]
class NodeSetup
{ ... }
{% endhighlight %}
