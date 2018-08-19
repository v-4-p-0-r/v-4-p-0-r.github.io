---
layout: post
title: "A Brief Intro to Buffer Overflows"
date: 2018-08-19 11:13:45 -700
categories: jekyll update
---
A buffer overflow occurs when a program attempts to fit data into a 'buffer' (area of memory designated for storing such data), but the data given to the buffer is larger than the buffer was designed to hold.

The following progam is a C program which will simply take an argument from the command line and copy it, with strcpy(), to a buffer.

{% highlight c linenos %}
#include <string.h>

int main(int argc, char *argv){	
	char buffer[64];
	strcpy(buffer, argv[1]);
	return 0;
	}
{% endhighlight %}	
Looks innocent enough, and is also a pretty useless program. But what if you give it more than it is programmed to hold?