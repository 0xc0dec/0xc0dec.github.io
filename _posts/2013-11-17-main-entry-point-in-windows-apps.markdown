---
layout: page
title:  Main entry point in Windows apps
date:   2013-11-17 00:00:00
permalink: windows-main
comments: true
---

Most of the time when you use external libraries to create window for your app (for example, graphics engines like OGRE),
the app doesn’t actually need to have WinMain entry point. Moreover, if you’re going to port the app to other platforms, you’ll have to adapt
the entry point to be main instead of WinMain. I think having the same main entry point on all platforms could be quite convenient.

On the other hand, only console apps normally have main entry point on Windows. They also auto-create console window, which isn’t
what we want in a GUI application. The following steps will help us to solve the problem (assuming that we use Visual Studio):

* Open project properties
* Change “Linker -> System -> SubSystem” option to WINDOWS
* Change “Linker -> Advanced -> Entry Point” option to mainCRTStartup

The System option set to WINDOWS makes application run with no console window. The Entry Point option allows us to specify what
entry point the compiler should expect to find in our code (main in this case).
