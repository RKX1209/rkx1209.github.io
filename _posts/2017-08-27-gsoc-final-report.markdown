---
layout: post
title:  "GSoC Final: radare2 Timeless Debugger"
date:   2017-08-27 15:29:42 +0900
---
GSoC has almost been finished. I would like to summarize [my works](https://github.com/radare/radare2/commits/master?author=rkx1209):
how did I tackle with and what I've done.
If You are interested in this new feature, I recommend you to read the document
[radare2 Timeless (Reverse) Debugger](https://radare.gitbooks.io/radare2book/content/debugger/revdebug.html)

## What is Timeless Debugger?
Timeless Debugging is a debugging paradigm, which is very similar to **Reverse debugging** or **Record and Replay**. In general, almost all debugger can control program counter of debuggee process forward
by similar commands, like run, continue and stepout... Moreover some rich debugger, like gdb, have also ability to seek program counter **backward** by stepback, continueback commands.
This feature is called **Reverse debugging** and very usefull when you have missed some debuggee's activities. You can find what is missed by seeking backward and recovering state.
However, radare2 had not have this feature before.

## Basic concepts
In GSoC term, I'm working on implementation of r2 Timeless(Reverse) Debugger.  
The architecture of r2 Timeless Debugger is fundamentally based on traditional debugging concept, **Record and Replay (RnR)**. In RnR, at first you need to run debuggee process and record **non-deterministic behavior**.  

**NOTE:** Only **non** deterministic events need to be recorded, because deterministic events can be reproduced by running actual program.  

## At the beginning
Before GSoC begining, I've added simple record and replay system for radare2.
A new command `dts+` can save entire program state, like register and memory dump then,
another new command `dsb` can replay until the before address from current pc.
i.e. You can single step back from current pc.
Internally, `dsb` command restores program state from the record previously saved by `dts+`,
and then, run program from there to desired point.
That's how program counter can be seeked backward. You can try step backing like:
{% highlight ruby %}
[0x7fa9171ffcd0]> dcu main
Continue until 0x00400536 using 1 bpsize
hit breakpoint at: 400536
[0x00400536]> pd 5
            ;-- main:
            ;-- main:
            ;-- rax:
            ;-- rip:
            0x00400536      55             push rbp
            0x00400537      4889e5         mov rbp, rsp
            0x0040053a      bfd4054000     mov edi, str.Hello_World    ; 0x4005d
4 ; "Hello World"
            0x0040053f      e8ccfeffff     call sym.imp.puts
            0x00400544      b800000000     mov eax, 0
[0x00400536]> dts+      # Record program state at that point
[0x00400536]> 3dso
[0x00400536]> dr rip
0x0040053f              # Current PC is here
[0x00400536]> dsb       # Let's step back!
[0x00400536]> dr rip
0x0040053a              # Success. PC is step backed!
[0x00400536]> dsb
[0x00400536]> dr rip
0x00400537              # Again
[0x00400536]> dsb
[0x00400536]> dr rip
0x00400536              # And again...
{% endhighlight %}

It works fine, but, program records are implmented as **entire** memory and register dump.
When you record program states at many times, there are many large dumps in memory.

So, during the community bonding period, My works are mainly focused on **Reducing size of
program records**.

# Diff-style format for recording
Program records are used to restore before state. So it's not necessary to hold entire dump.
Only different pages from a previous record are enough for restoring.
So I've added new diff style recording format, which saves only changed memory area
from previous recording point.  
Architecture overview is as shown in the figure.  

**(Trace session architecture internal.)**
![r2snap]({{site.baseurl}}/images/r2snap.jpg)

OK. Let's see this feature.   
You can use `dts` command to show the all list of program records.  

{% highlight ruby %}
[0x7fb11b90bcc0]> dcu main
Selecting and continuing: 23742
hit breakpoint at: 400526
[0x00400526]> dts+                        # Save first session
[0x00400526]> 10dso
[0x00400526]> dts+                        # Save second session(saved only changed area)
Reading 135168 bytes from 0x00fe8000...
[0x00400526]> dts                         # Show the list of trace sessions
session: 0   at:0x00400526
session: 1   at:0x00400550
  - 0 0x00601000 - 0x00602000 size: 4096 (pages: 0 )
  - 1 0x7fb11b905000 - 0x7fb11b907000 size: 8192 (pages: 0 1 )
  - 2 0x7fb11b907000 - 0x7fb11b90b000 size: 16384 (pages: 0 )
  - 3 0x7fb11bb04000 - 0x7fb11bb07000 size: 12288 (pages: 1 )
  - 4 0x7fb11bb31000 - 0x7fb11bb32000 size: 4096 (pages: 0 )
  - 5 0x7ffdec7e7000 - 0x7ffdec809000 size: 139264 (pages: 31 32 )

{% endhighlight %}

As you can see the result of `dts`, session 1 saves only different pages from previous session 0.  


# Checkpoint
If you run `dsb` command just after long loop, you would find that perfomance become very slow. :(
So after GSoC Phase1 began, I've implemented simple checkpoint system, that can automatically save trace sessions among long execution. And fixed some bugs related to record and replay functions.

# Thanks
Finally, I would like to thanks people:
- radare community
- My mentors, [pancake](https://twitter.com/trufae) and [Álvaro Felipe](https://twitter.com/alvaro_fe)
- The GSoC organization

GSoC has been finished, bug I'd like to continue to develop for r2!  
Thank you.  
