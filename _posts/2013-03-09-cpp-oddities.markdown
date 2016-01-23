---
layout: page
title:  Something you probably didn’t know about C/C++
date:   2013-03-09 00:00:00
permalink: odd-cpp
---

I love to collect mind-blowing and not too widely known C/C++ language constructions and facts. This post carries my collection and will be updated
when new entries appear.

C/C++
{% highlight cpp %}
// zero-sized bit fields
struct S
{
  int a;
  int b:2;
  int c:6,
  :0;
  int d;
};
{% endhighlight %}

<!--break-->

C-only
{% highlight cpp %}
// Exit code when there's no return statement?
#include <stdio.h>
int main()
{
  printf("Test\n");
}

// The program exit code will be... ? 5. When there is no return statement,
// the program will return the result of the last executed function.
// In this case it is 5, because printf returns the number of symbols actually printed.
{% endhighlight %}

C/C++
{% highlight cpp %}
// a[b] is practically the same as *(a + b)
char c = 4["Hello, world!"]; // c == 'o'
{% endhighlight %}

C/C++
{% highlight cpp %}
int x = 33;
int f()
{
  int x = 1045;
  {
    return x; // returns 1045
  }
}

int x = 33;
int f()
{
  int x = 1045;
  {
    extern int x;
    return x; // returns 33
  }
}
{% endhighlight %}

C-only
{% highlight cpp %}
// Passed array must have at least 10 elements (-Warray-bounds):
void f(int arr[static 10]) { }

// Free compile-time not-NULL check [-Wnonnull]
void f(int arr[static 1]) { }
{% endhighlight %}

C-only
{% highlight cpp %}
int x = 'FOO!'; // single quotes! codepad.org compiler says x == 1179602721
{% endhighlight %}

C/C++
{% highlight cpp %}
int x = y+++z; // parsed as y++ + z ("greedy lexer rule")
int a = b+++++c; // b++ ++ + c, not b++ + ++c
{% endhighlight %}

C-only
{% highlight cpp %}
// The shortest C-program that compiles and links. With a warning. Do not run!
main;
// No warnings. Do not run either!
int main;
{% endhighlight %}

C-only
{% highlight cpp %}
// Old-style function definition
int main(argc, argv)
int argc;
char *argv[];
{
  return 0;
}
{% endhighlight %}

C-only
{% highlight cpp %}
return ((int []){1,2,3,4})[1];
// returns 2
{% endhighlight %}

C-only
{% highlight cpp %}
// Initializer with designators hell
struct {
   int x;
   struct {
       int y, z;
   } nested;
} i = { .nested.y = 5, 6, .x = 1, 2 };  
// i.nested.y == 2, i.nested.z == 6
{% endhighlight %}

GCC C-only
{% highlight cpp %}
// omit second component of ?: operator
extern int f();
return f() ? : -1; // Returns f() if it's not zero, or -1 otherwise. Reminds me of C# operator ??
{% endhighlight %}

C/C++
{% highlight cpp %}
// zero-sized bit fields
struct S
{
  int a;
  int b:2;
  int c:6,
  :0;
  int d;
};
{% endhighlight %}

Most interesting parts of [this](http://nickdesaulniers.github.io/blog/2013/07/25/designated-initialization-with-pointers-in-c/)
article about designated initialization in C:
{% highlight cpp %}
struct point
{
  int x, y;
};
// ...
struct point p = { 1, 2 } // ok, p now has coordinates (1, 2)
{% endhighlight %}

{% highlight cpp %}
struct point p = { .y = 1, .x = 2 } // ok, p now has coordinates (2, 1)
{% endhighlight %}

{% highlight cpp %}
// Re-assign? No problems, but tricky
struct point p = { .x = 1, .y = 2 };
// ...
p = { .x = 10, .y = 20 }; // compilation error
p = (struct point){ .x = 10, .y = 20 }; // works
p = (struct point){ 10, 20 }; // this works too
{% endhighlight %}

{% highlight cpp %}
int distance(struct point a, struct point b);
// ...
distance({ 1, 2 }, { 3, 4 }); // no-no, compilation error
distance((struct point){ 1, 2 }, (struct point) { 3, 4 }); // fine, this compiles
{% endhighlight %}

{% highlight cpp %}
// Initializing pointer members
struct node
{
  char *value;
  struct node *next;
};
// ...
struct node n =
{
  .value = "Hello, world!",
  .next = (&(struct node) { .value = "Next node" }) // wow!
}
{% endhighlight %}

Empty declarations in C:
{% highlight cpp %}
typedef;
int typedef const;
{% endhighlight %}
Won’t compile with `-Wall` though.

Some people write such code probably hoping that C/C++ has native support for tuples. No, it doesn’t.
{% highlight cpp %}
int func()
{
    int a = 1;
    int b = 2;
    int c = 3;
    return (a, b, c);
}

// ...

int d = func(); // d == c == 3, if () operator is not overloaded
{% endhighlight %}

This construction is strange in a way that you probably won't encounter it very often. For some reasons it's not commonly used. The [function-try-block](http://en.cppreference.com/w/cpp/language/function-try-block):
{% highlight cpp %}
int f(int n = 2) try {
  ++n; // increments the function parameter
  throw n;
} catch(...) {
  ++n; // n is in scope and still refers to the function parameter
  assert(n == 4);
  return n;
}
{% endhighlight %}
