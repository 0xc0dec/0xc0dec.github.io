---
layout: page
title:  Macbook and wi-fi
date:   2012-05-10 00:00:00
---

If your Mac laptop fails to connect to Wi-fi without any visible reasons (endless "connecting…" Airport animation or "timeout" errors),
deleting these files might help you, as it once helped me
{% highlight bash %}
/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist
/Library/Preferences/SystemConfiguration/NetworkInterfaces.plist
/Library/Preferences/SystemConfiguration/com.apple.network.identification.plist
{% endhighlight %}

You will need root access to delete them. Don’t forget to reboot after deleting. Please note that these actions will clear all your saved network preferences
(solution found [here](http://forums.macrumors.com/showthread.php?t=858407)).