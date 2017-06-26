---
layout: post
title:  "GSoC Phase1: Timeless Debugger Update"
date:   2017-06-28 15:29:42 +0900
categories: jekyll update
---
GSoC (Google Summer of Code) Phase1 has been finished. This post is to outline the work completed during
the Phase1 period and show you an overview of *radare2 Timeless Debugger*.

# Timeless Debugger
To illustrate the architecture of r2 Timeless Debugger, Firstly I need to explain, *"What is Timeless Debugging?"*.  
Timeless Debugging is a new paradigm of debugging, that is very similar to *"Reverse debugging"* or *"Record and Replay"*.
It observes a binary at any point of its execution state by saving entire execution log.
For example, `QIRA` execute every operations, like `mov rax, [0x1000]` and
records each logs like, `Write [0x1000] to rax`. After recording, user can restore program state
at any point of binary by using these records.  


To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
