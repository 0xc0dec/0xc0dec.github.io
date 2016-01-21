---
layout: page
title:  One line of code to make a better quick search in FAR Manager
date:   2014-03-29 00:00:00
---

<img src="/data/far.png" class="img-thumbnail pull-left margined20">

A few testimonials first. FAR Manager was the kind of software which I used a lot on a daily basis.
Really, it’s fast, has a rich feature set and looks cool (console-styled interface is quite competitive with those native Windows GUI’s that other managers have).
When it comes to reading large files like logs, FAR manager is beyond comparison.

However, one thing that sucks and always did in FAR is the quick search – the search that is activated when you press `Alt` key and start typing.
The concept itself is very useful and allows you to navigate files at the speed of light, but in FAR the whole idea is ruined by the way it’s implemented:
FAR seeks the input string only at the beginning of each file name. Let’s imagine that we have a program that consists of an .exe file and many libraries.
All file names have the same prefix. For example, they follow some awkward notation `<CompanyName>.<ProgramName>.<extension>`, which, you know,
is quite a real case in business software. As you might expect, there’s no way to quickly navigate the exe file in this directory – you literally have to print the entire file name to select it,
because most of the files differ only by extension.

<!--break-->

OK, I’m being a little biased here. As I figured out today, you actually can use masks when searching. Considering the case above, we could print `*exe` to select the executable file.
But note how many actions you have to perform: press Alt, then Shift, then `6` to print the `*` sign, and only then the essential string that you wish to find. Personally I think
it’s a serious usability problem, because FAR is used mostly by programmers and geeks who need fast access to their files, and pressing all those keys is a boilerplate.
I also never met anyone who knew that FAR supports wildcards in quick search.

Let’s see how other file manager do the job. I’ve used FreeCommander and TotalCommander – they do it in a most natural way: you print a string,
and they select those files that contain the string in their name (not necessary at the beginning). They also have options to tune the behaviour of this feature,
for example they can activate the search when you just start typing, no need to press any special key like Alt. Options also include filtering the files list to show only files that match your query.

Honestly, I don’t use FAR for my personal needs anymore. But at work I use it sometimes to read logs, so today I’ve hacked the program a bit to make the quick search more usable.
If you have Visual Studio and experience the same problems, go to Google code and download the source code. Navigate to the file `far_unicode/filelist.cpp` and add one line:

{% highlight cpp %}
int FileList::FindPartName(const string& Name,int Next,int Direct,int ExcludeSets)
{
#if !defined(Mantis_698)
	int DirFind = 0;
	string strMask = Name;

	if (!Name.empty() && IsSlash(Name.back()))
	{
		DirFind = 1;
		strMask.pop_back();
	}

+	strMask.insert(0, L"*");
	strMask += L"*";

	if (ExcludeSets)
	{
{% endhighlight %}

As you can see, FAR already turns your string into a mask by adding ‘*’ to the end. We go further and add ‘*’ to the beginning so that the input string
is searched somewhere in the middle of a file name. Select the required configuration (debug/release, x86/x64) and build the program. Fortunately you
don’t need to install any additional dependencies to build FAR, everything just works out of the box.

Further improvements may involve disabling the Alt key requirement. Really, why is it there? All FAR use cases I’ve seen so far involve a lot of file
navigation and editing and very few or no typing in the console. If you want to run some tool very often, go start cmd.exe directly from FAR and type
the necessary command there. That will allow you to start the tool fast and from the very same directory each time.
