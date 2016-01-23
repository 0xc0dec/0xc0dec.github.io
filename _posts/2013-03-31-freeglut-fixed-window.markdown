---
layout: page
title:  FreeGlut fixed window
date:   2013-03-31 00:00:00
permalink: freeglut-fixed-window
---

It seems like FreeGLUT has no support for fixed-size windows at the moment
(i.e. having normal border, but non-resizable). For my needs I wrote a small patch (for windows version only) which fixes this failing.

In FreeGLUT sources find the freeglut_ext.h file and modify it as follows (new lines are marked with ++):
{% highlight c %}
/*
 * GLUT API macro definitions -- the display mode definitions
 */
#define  GLUT_CAPTIONLESS                   0x0400
#define  GLUT_BORDERLESS                    0x0800
#define  GLUT_SRGB                          0x1000
++ #define  GLUT_FIXED_SIZE                    0x0040
{% endhighlight %}

Next, modify the freeglut_window.c file:
{% highlight c %}
else if( window->Parent == NULL )
{
    if ( fgState.DisplayMode & GLUT_BORDERLESS )
    {
        /* no window decorations needed */
    }
    else if ( fgState.DisplayMode & GLUT_CAPTIONLESS )
        /* only window decoration is a border, no title bar or buttons */
        flags |= WS_DLGFRAME;
    else
        /* window decoration are a border, title bar and buttons.
         * NB: we later query whether the window has a title bar or
         * not by testing for the maximize button, as the test for
         * WS_CAPTION can be true without the window having a title
         * bar. This style WS_OVERLAPPEDWINDOW gives you a maximize
         * button. */
         flags |= WS_OVERLAPPEDWINDOW;
++    if (fgState.DisplayMode & GLUT_FIXED_SIZE)
++	flags &= ~(WS_MAXIMIZEBOX | WS_THICKFRAME);
}
{% endhighlight %}

This is a dirty hack, but probably FreeGLUT developers will introduce a good solution soon.
