---
layout: post
title: "Sort of a format string vulnerability: protostar format0"
date: 2019-7-18
---

We have since finished the stack overflow section of protostar, and now it is time to move onto format strings.

The weird thing is that the very first level is not exactly a very common format string vulnerability, and can be solved both the intended format string AND a stack overflow. 

{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void vuln(char *string)
{
  volatile int target;
  char buffer[64];

  target = 0;

  sprintf(buffer, string);
  
  if(target == 0xdeadbeef) {
      printf("you have hit the target correctly :)\n");
  }
}

int main(int argc, char **argv)
{
  vuln(argv[1]);
}
{%endhighlight%}
If you have been paying any attention to the stack challenges, you should see target is right after buffer[].

We can do it the old school way and just overflow buffer with 64 bytes, and then overwrite target with deadbeef (using little endian byte order of course).
![format0-1](/assets/format0-1.jpg)
But we have been doing that for forever, and plus, the challenge description does say to complete the challenge with less than 10 bytes of input. 

The way to do this is through format string exploits. 

I am no expert in these specific vulnerabilities, so I don't know how many different forms one may take, but 

A common example of this is shown below
{% highlight c %}
printf("%d", variable); //correct
printf(variable); //incorrect
{% endhighlight %}
The first line is a correct format string, and the second is an incorrect one. 

The vulnerable line is the line with the sprintf() function.
The first parameter in sprintf is a character array, in this case, buffer. The second is meant to be a format string, but instead of formatting it as something like 
{% highlight c %}
sprintf(buffer, "%s", string)
{% endhighlight %}
It was done without formatting. And since we control this variable called string, we can use this vulnerability.
![format0-2](/assets/format0-2.jpg)
In the command line, one can see that the program was run, and due to user input being taken as argv[1], we know that we need to supply the program with our input through the command line. So we use the old $() to call another program within the initial call. 

We call python to execute one line, which simply prints out a %64x, and then appends the bytes to spell "deadbeef". 

The most important piece of this is the %64x, which is essentially padding 64 bytes of space before the rest of the input. It has the same effect, in this instance, of overflowing the buffer via the stack, but this time we are able to do it with far smaller input. 

The same effect can be achieved by changing the x to other functioning format strings. For example, %64s or %64d would both function the same. 