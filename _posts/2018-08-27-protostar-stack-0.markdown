---
layout: post
title: "Buffer Overflow Example: protostar stack0"
date: 2018-08-27 13:39:48 -700
categories: jekyll update
---
Now we know a little bit about what a buffer overflow is, and so we can begin looking at from a security perspective.
To do this I will be using a level from exploit-exercises.com's virtual machine: protostar. I will specifically look at the level stack0.

The code below is the source code for the binary we are going to overflow.
{% highlight c linenos %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
{% endhighlight %}
Looking at this initially, we see that the code declares two variables: one an array of 64 characters, and one an integer which we see is set to 0.

The gets() function will take user input and store it into 'buffer', then the integer is compared to 0. If it is NOT 0, we get the message that we have completed the level, and have successfully changed the variable.

The user input has no way to reach this variable though, all we control is the buffer. Changing this variable should be impossible. Right?

$./stack0

test

Try again?

$

We don't get the desired result. Darn. But what if we try giving more data to the buffer than is allocated to it? I will use python to pass 65 "A"s into the buffer (one more character than is allocated.)

$ python -c 'print "A"*65' | ./stack0

you have changed the 'modified' variable

$

Awesome.

This works because the function gets() doesn't check to make sure that the data it is dealing with is of any specific size. Thus, like pouring too much water into a bucket, it will overflow. And if you notice, the variables are declared right next to one another, and are next to each other in memory.

The buffer is filled with "A" and since the modified variable is an integer, it is overwritten with the ascii value of "A" which is 0x41 (65 in decimal), thus, the variable is changed to something other than 0.