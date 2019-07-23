---
layout: post
title: Writing a Specific Value to a Target: protostar format2
date: 2019-7-23
---
Now we're going to move on to writing specific values to a target. 
{% highlight c%}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int target;

void vuln()
{
  char buffer[512];

  fgets(buffer, sizeof(buffer), stdin);
  printf(buffer);
  
  if(target == 64) {
      printf("you have modified the target :)\n");
  } else {
      printf("target is %d :(\n", target);
  }
}

int main(int argc, char **argv)
{
  vuln();
}
{% endhighlight %}
The goal here is to set the target equal to 64. 

This code uses a secure version of gets, so there's no overflow here. 
The vulnerable line is the printf for our buffer, that we supply through this aformentioned secure gets function, fgets.

First, we will find the address of target. This is the same process as in the previous challenge. We use objdump -t.
![format2-1](/assets/format2-1.png)
Now we know that our target is at 0x080496e4.

Next we will figure out how far away our input is on the stack. 
![format2-2](/assets/format2-2.png)
It looks like our input is 4 bytes away. We can see this because the very beginning of our input, the 4 As we supplied appear 4 bytes later (as hex 41, their ascii value).

Now instead of writing As, we will put in our target's address.
![format2-3](/assets/format2-3.png)
We have modified the variable. Now the stack is reading up to the point where we wrote the target address, then writing the number of characters written to that address. 

Now all we have to do is add some more characters in and get the target to equal 64. 
![format2-4](/assets/format2-4.png)
There we go. The stack takes the target address, 41 As, and then skips 3 addresses onto our target, and writes the number of characters to said target. 
![format2-5](/assets/format2-5.png)
The next level will continue on this same kind of exploit.