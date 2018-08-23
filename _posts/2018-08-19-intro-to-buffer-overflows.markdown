---
layout: post
title: "A Brief Intro to Buffer Overflows"
date: 2018-08-19 11:13:45 -700
categories: jekyll update
---
A buffer overflow occurs when a program attempts to fit data into a 'buffer' (area of memory designated for storing such data), but the data given to the buffer is larger than the buffer was designed to hold.

The following is a C program which will simply take an argument from the command line and copy it, with strcpy(), to a buffer.

{% highlight c linenos %}
#include <string.h>

int main(int argc, char *argv){	
	char buffer[64];
	strcpy(buffer, argv[1]);
	return 0;
	}
{% endhighlight %}	
Looks innocent enough, and is also a pretty useless program. 
Running the program doesn't do anything interesting, assuming you give it a small bit of data.

$ ./test foobar

$

It just returns to the terminal.

But what if you give it more than it is programmed to hold? (I am using python to pass 100 "A"s to the program)

$ ./test $(python -c 'print "A"*100')

Segmentation fault (core dumped)

What is this error we are given? Segmentation fault?

A segmentation fault is when a program is attempting to read or write (in this case write) to an area of memory it doesn't have permission to. In this case our long string of "A" filled up the buffer, and just kept on writing past what it was allowed, and so a segmentation fault was triggered, crashing the program.

While crashes can be annoying, the real black magic of a buffer overflow comes in the form of overwriting specific areas of memory and even forcing the computer to execute whatever instructions you give to it. 