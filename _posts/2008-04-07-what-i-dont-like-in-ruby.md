---
layout: post
title: What I don’t like in Ruby
date: 2008-04-07 11:02
author: Giacomo Graziosi
comments: true
categories: [ruby]
tags: [c, cplusplus, cpp, gil, metaprogramming, programming, programming, python, ruby]
---
I’m beginning to learn the well hyped Ruby language, in many ways it looks useful, elegant and pleasant, still it has something that, at least at a first glance, looks odd to me. I will use this post to take track of those odd things that I will meet when studying the language.

So, here we go:

<strong>1) Ends</strong>, take a look on this snippet:
{% highlight ruby %}
if !r.nil? then
  rr = r.right
  if !rr.nil? then
    if ((r.color == RedBlackTree::RED)and(rr.color == RedBlackTree::RED)) then
      x = rot_left()
      copy(x) unless x.nil?
      @color = RedBlackTree::BLACK
      l = @left
      l.color = RedBlackTree::RED unless l.nil?
    end
  end
end
{% endhighlight %}
and here comes the Python equivalent:
{% highlight python %}]if (right != None):
    rr = right.__right
    if (rr != None):
        if ((right.__color == RedBlackTree.red)and(rr.__color == RedBlackTree.red)):
            x = self.__rotLeft()
            if (x != None):
                self.__copy(x)
            self.__color = RedBlackTree.black
            left = self.__left
            if (left != None):
                left.__color = RedBlackTree.red
{% endhighlight %}
The Python way looks just the best way in my opinion. What’s the reason for the ends when you’ll indent the code anyway (and if you don’t then you deserve nothing else than suffering and pain)? Even the <strong>{</strong> C/C++/Java approach <strong>}</strong> appears way better or at least less intrusive.

<strong>2) “Fuzzy” expressions terminators.</strong> Try to compile and execute the following C++ code:
{% highlight cpp %}#include <iostream>
using namespace std;

class Foo {
    public:
    void bar() {
        int x = 3;
        int y = 2;
        cout << x
        +y;
        cout << endl;
    }
};

int main() {
    Foo *f = new Foo;
    f->bar();
    delete(f);
    return 0;
}
{% endhighlight %}
What do you expect it to print? Right, it will correctly print <em>5</em>. Now Ruby’s turn:
{% highlight ruby %}
class Foo
  def bar
    x,y = 3,2
    puts x
    +y
  end
end

f = Foo.new()
f.bar
{% endhighlight %}
Now what output do you expect? The answer is <em>3</em>. This is because Ruby doesn’t use semicolons as C, C++, Java, PHP and many other languages do to terminate statements. It is still the better way to go, semicolons are redundants useless things almost everytime, still you have to pay attention: note that the above Ruby code isn’t wrong so the interpreter will not warn you, it will just print <em>3</em> instead of <em>5</em> and the method <em>bar</em> will return the value<em>2</em> (remember that in Ruby the last expression in a method or function is the actual return value of it so if you replace <em>f.bar</em> with <em>puts f.bar</em> it will print the “missing” <em>2</em>).

<strong>3) No method overloading.</strong> The following code is self-explanatory:
{% highlight ruby %}
class Foo
    def bar(a, b)
        puts b
    end

    def bar(a)
        puts a
    end
end

gh = Foo.new()
gh.bar("hello", "world")
{% endhighlight %}
if you try to execute it then you will get something like this:
<blockquote>asd.rb:12:in `bar’: wrong number of arguments (2 for 1) (ArgumentError)
from asd.rb:12</blockquote>
<strong>4) GIL (global interpreter lock).</strong> To me it’s nothing else than an alternative reading for “fake threading”. If you are on a multicore/multicpu system try to execute this snippet while monitoring the cpu activity, it will start 8 threads and compute fibonacci(40) on each thread:
{% highlight ruby %}
class Fibonacci
    def fib(i)
        if (i <= 2)
            return 1
        else
            return fib(i - 1) + fib(i - 2)
        end
    end
end

threads = []
8.times { |i| threads[i] = Thread.new {Fibonacci.new().fib(30) } }
threads.each {|t| t.join}
{% endhighlight %}
If you come from C/C++ or Java you may expect this code to use up to 8 cores/cpus on your system at the same time… wrong! It will execute the code using only one core/cpu at once because the GIL prevents the threads from taking control of the interpreter at the same time. Cool, isn’t it?

<strong>5) Method invocation hooking.</strong> The following is the best way I found to intercept whenever a method is called:
{% highlight ruby %}
class Foo
  def bar1(gh1)
    puts "bar1 #{gh1}"
  end
end

class Foo
  def bar1_hook(*args)
    puts "bar1 hooked!"
    bar1_asd(args)
  end

  alias_method(:bar1_asd, :bar1)
  alias_method(:bar1, :bar1_hook)
end

a = Foo.new
a.bar1("foobar")
{% endhighlight %}
So <em>bar1</em> is the method to be hooked: you have to monkey patch the <em>Foo</em> class defining the hooking method and hack it all together with alias_method. Doing this you are ripping the original class apart: if someone will edit the<em>bar1</em> method after your hook he/she will actually be editing the <em>bar1_hook</em> method without even noticing it (well, he will notice the unexpected behaviour and maybe try to debug his/her own code… have fun :-|), I really hope to be wrong about this point…
