---
layout: page
title:  Building multiple linked binaries in CLion on Mac OS X
date:   2014-10-19 00:00:00
permalink: clion-multiple-binaries
comments: true
---

<div class="row text-center"><img src="/images/clion_logo.png" class="margined20"/></div>

<a href="https://www.jetbrains.com/clion/">CLion</a> is a new C/C++ IDE from JetBrains, the creators of ReSharper, IntelliJ IDEA and many other great products for software developers. CLion is claimed to support a full-featured refactoring workflow for C++, similar to how it is already implemented in other JetBrains products. At this moment the IDE is available in early access mode, which means that we can download and use it freely, keeping in mind that the product is not stable yet, so bugs are possible.

I decided to move one of my projects from XCode to CLion. The refactoring capabilities of XCode for C++ are far from being good (in fact, there is no refactoring support for C++), and after using ReSharper for my C# projects, I simply don't understand how to work without "Rename", "Find usages", "Generate missing members", smart highlighting and a plenty of other super-useful features.

CLion uses CMake to describe and manage its projects (at least yet). If you're not very familiar with CMake (like me), creating and building anything more complex than the default Hello world will be sort of a nightmare. In this post I'll share my experience in building a static library and a project that uses this library in CLion, hoping that this will save a lot of googling time for anyone having the similar task.

<!--break-->

For example, we want to build a static library named *solo* and a test project *Test1* that links against it. My project structure looks like this:

<div class="row text-center"><img src="/images/solo_project.png" class="margined20"/></div>

The *src* folder contains the library source files and the *include* folder contains the interface headers. The *Test1* project resides inside the *tests/test1* folder. I'm using a single CMakeLists.txt for everything for the sake of simplicity. As the project grows bigger, you might consider splitting it into several CMakeLists' to make it modular and easier to manage.

Lets examine the CMakeLists.txt file. The first two lines are added by CLion by default:
{% highlight cpp %}
cmake_minimum_required(VERSION 2.8.4)
project(solo)
{% endhighlight %}
They state that we want CMake version to be at least 2.8.4, and our project is called *solo*.

Here comes the first interesting part: we tell CLion where to save the output binaries. By default, they are put into some wierd location like */Users/&lt;username&gt;/Library/Caches/clion10/cmake/generated/707a014b/707a014b/Debug*. Here we want to store them in some more conventional place like the *build* folder inside the project folder:
{% highlight cpp %}
set(EXECUTABLE_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/build/")
set(LIBRARY_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/build/")
{% endhighlight %}

The next line tells CMake to pass specific flags to the Clang compiler:
{% highlight cpp %}
add_compile_options(-std=c++0x)
{% endhighlight %}

Our library needs some headers to compile, so we tell CMake to search them in a number of directores:
{% highlight cpp %}
include_directories(
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/external/SDL/2.0.3/include)
{% endhighlight %}
Here the *${PROJECT\_SOURCE\_DIR}* refers to the root project folder.

We also need to specify where to look for the external libraries when linking our library against them:
{% highlight cpp %}
link_directories(${PROJECT_SOURCE_DIR}/external/SDL/2.0.3)
{% endhighlight %}

Next we tell CMake to find all \*.cpp files inside the *src* folder and put them into the *sources* variable:
{% highlight cpp %}
file(GLOB sources "src/*.cpp")
{% endhighlight %}

In the next chunk of code we tell CMake to find some frameworks that we're going to be using, and save their paths to the *frameworks* variable:
{% highlight cpp %}
find_library(carbon_lib Carbon) # we look for the Carbon framework and use carbon_lib as an alias for it
find_library(iokit_lib IOKit)
find_library(forcefeedback_lib ForceFeedback)
find_library(cocoa_lib Cocoa)
find_library(audiounit_lib AudioUnit)
find_library(coreaudio_lib CoreAudio)
find_library(opengl_lib OpenGL)
find_library(corefoundation_lib CoreFoundation)

set(frameworks
    ${carbon_lib}
    ${iokit_lib}
    ${forcefeedback_lib}
    ${cocoa_lib}
    ${audiounit_lib}
    ${coreaudio_lib}
    ${opengl_lib}
    ${corefoundation_lib})
{% endhighlight %}

Finally, we say that we want to build a static library from the given sources and link it against the *SDL2* library and the frameworks that we found before:
{% highlight cpp %}
add_library(solo STATIC ${sources})
target_link_libraries(solo SDL2 ${frameworks})
{% endhighlight %}

Adding the second binary (the *Test1* executable) is as easy as it is. We tell CMake to build it from the *Main.cpp* file and link agains the *solo* library:
{% highlight cpp %}
add_executable(Test1 tests/test1/Main.cpp)
target_link_libraries(Test1 LINK_PUBLIC solo)
{% endhighlight %}

Now if you hit *Cmd + F9* (Build), you should see two new files inside the *build* folder: *Test1* and *libsolo.a*.

The whole CMakeLists.txt file:
{% highlight cpp %}
cmake_minimum_required(VERSION 2.8.4)
project(solo)

set(EXECUTABLE_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/build/")
set(LIBRARY_OUTPUT_PATH "${PROJECT_SOURCE_DIR}/build/")

add_compile_options(-std=c++0x)

include_directories(
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/external/SDL/2.0.3/include)

link_directories(${PROJECT_SOURCE_DIR}/external/SDL/2.0.3)

# Engine

file(GLOB sources "src/*.cpp")

find_library(carbon_lib Carbon)
find_library(iokit_lib IOKit)
find_library(forcefeedback_lib ForceFeedback)
find_library(cocoa_lib Cocoa)
find_library(audiounit_lib AudioUnit)
find_library(coreaudio_lib CoreAudio)
find_library(opengl_lib OpenGL)
find_library(corefoundation_lib CoreFoundation)

set(frameworks
    ${carbon_lib}
    ${iokit_lib}
    ${forcefeedback_lib}
    ${cocoa_lib}
    ${audiounit_lib}
    ${coreaudio_lib}
    ${opengl_lib}
    ${corefoundation_lib})

add_library(solo STATIC ${sources})
target_link_libraries(solo SDL2 ${frameworks})

# Tests
add_executable(Test1 tests/test1/Main.cpp)
target_link_libraries(Test1 LINK_PUBLIC solo)
{% endhighlight %}

### Conclusion
This article gives a recipe for writing a simple CMake file for building two linked binaries in CLion. Keep in mind that I'm not an expert in CMake, neither in Clion, so the given file might not be the best that you can make to get the job done. However, it gives a good start for working with CLion, which, as I have noticed, is not well covered in the online tutorials yet.
