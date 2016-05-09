---
layout: page
title:  C++ embedded scripting&#58; ChaiScript recipes
date:   2015-07-07 00:00:00
permalink: chaiscript-recipes
comments: true
---

[Go to recipes]({% post_url 2015-07-07-chaiscript-recipes %}#recipes)

Currently there are several options for embedding scripting into your C++ application. Among the most well-known, from my experience, are
[Lua](http://www.lua.org/), [AngelScript](http://www.angelcode.com/angelscript/) and the recent addition - [ChaiScript](http://chaiscript.com/).
I've been evaluating these languages for some time recently and my main impressions are:

<!--break-->

* Lua has many binding libraries, but very few of them support smart pointers and std containers like `std::string` at the same time (a must-have I think). Many of them
are abandoned, even the most well-known LubBind (sadly). Some libraries require Boost to compile, which might not be an option for some projects.
Some libraries don't compile in MSVS 2013 (even with November CTP) due to the extensive use of C++11/14. Others have poor documentation...
I personally can recommend [luawrapper](https://github.com/tomaka/luawrapper) because it has one of the richest feature sets and a fairly good documentation.
Unfortunately it requires Boost and compiles in MSVS 2013 Nov CTP only after some modifications (removing `constexpr` and ref-value qualifiers).
* AngelScript has a built-in mechanism for embedding into C++ programs, but it looks bulky and (probably - not sure) you have to register smart pointers manually
in order to use them. Overall, the syntax leaves you with many ways to make a mistake in your bindings.
* ChaiScript is a header-only engine (!), heavily template-based and increases the compilation time/binary size significantly. It is, however, designed
for embedded into C++ programs and thus provides a very straightforward and transparent way of doing this. Most of the time you understand what's going on
and why. The language syntax looks like a mix of JavaScript and C++.

If you are reading this article chances are you're deciding between those three languages, or have already chosen ChaiScript, but
can't find some necessary information in the official documentation. In this (updateable) post I'll gather a few ChaiScript tips and code snippets
that were most difficult for me to discover. They come either from long series of experiments or from the official forum and
careful reading of examples and tests. This article assumes that you have already read the documentation, looked through the
[cheat sheet](https://github.com/ChaiScript/ChaiScript/blob/master/cheatsheet.md), [samples](https://github.com/ChaiScript/ChaiScript/tree/master/samples)
and [unit tests](https://github.com/ChaiScript/ChaiScript/tree/master/unittests).

<div id="recipes"></div>
Bootstrapping a project (for impatient):
{% highlight cpp %}
#include <chaiscript.hpp>
#include <chaiscript_stdlib.hpp>

int main()
{
    chaiscript::ChaiScript engine(chaiscript::Std_Lib::library());
    // ... register API for your scripts
    engine.eval_file("script.chai");
    return 0;
}
{% endhighlight %}

Attaching C++ function to a script object:
{% highlight cpp %}
// Host program

void logMsg(chaiscript::Boxed_Value& obj, const std::string& s)
{
    std::cout << s << std::endl;
}

// ...

engine.add(chaiscript::fun(&logMsg), "log");

// Script

class Test
{
    def Test() {}
}

var t := create("Test");
t.log("abc");

// Output

abc
{% endhighlight %}

Inheritance, `shared_ptr` to base class, polymorphic function call:
{% highlight cpp %}
// Host program

struct Base
{
    virtual ~Base() {} // ChaiScript requires classes to be polymorphic for inheritance to work
};

struct A: Base
{
    void func()
    {
        LOG("A::func");
    }
};

struct B: Base
{
    void func()
    {
        LOG("B::func");
    }
};

// Yes, you can pass shared_ptr to your script
std::shared_ptr<Base> getSmth(const std::string& name)
{
    if (name == "A")
        return std::make_shared<A>();
    if (name == "B")
        return std::make_shared<B>();
    return nullptr;
}

// ...

engine.add(chaiscript::user_type<Base>(), "Base");
engine.add(chaiscript::user_type<A>(), "A");
engine.add(chaiscript::user_type<B>(), "B");
engine.add(chaiscript::base_class<Base, A>());
engine.add(chaiscript::base_class<Base, B>());
engine.add(chaiscript::fun(&A::func), "func");
engine.add(chaiscript::fun(&B::func), "func");
engine.add(chaiscript::fun(&getSmth), "getSmth");


// Script

var a = getSmth("A");
a.func();

var b = getSmth("B");
b.func();

// Output

A::func
B::func
{% endhighlight %}

Creating script object from C++, storing it, calling its functions, passing it back to script:
{% highlight cpp %}
// Host program

chaiscript::Boxed_Value obj;

chaiscript::Boxed_Value& create(const std::string& className)
{
    obj = engine.eval<chaiscript::Boxed_Value>(className + "()");
    return obj;
}

void callFunc()
{
    // Member functions require an implicit first parameter - pointer to the object
    auto func = engine.eval<std::function<void(chaiscript::Boxed_Value&)>>("func");
    func(obj);
}

//...

engine.add(chaiscript::fun(&create), "create");
engine.add(chaiscript::fun(&callFunc), "callFunc");


// Script

class Test
{
    var id;

    def Test()
    {
        this.id = 0;
    }

    def func()
    {
        print(this.id);
    }
}

// ":=" here is important. If it were "=", ChaiScript would create a deep copy
// and you'll end up having two copies of Test object
var t := create("Test");
t.id = 100500;
t.func();
callFunc();

// Output

100500
100500
{% endhighlight %}

Return any object from C++ function to ChaiScript:
{% highlight cpp %}
// Use Boxed_Value as return value
chaiscript::Boxed_Value func(const std::string& token)
{
	if (token == "Foo")
		return chaiscript::Boxed_Value(std::make_shared<Test>());
	if (token == "Bar")
		return chaiscript::Boxed_Value(std::string("Hello, world!"));
	// .. and so on
	return chaiscript::Boxed_Value(123);
}

// Register function as normal
script.add(chaiscript::fun(&func), "func");
{% endhighlight %}
