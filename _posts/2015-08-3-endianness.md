---
layout: post
title: Endianness
date: 2015-08-03
---

an int of 1 would have two options:

Little enddian:
<pre>
x01 00000001
x02 00000000
x03 00000000
x04 00000000
</pre>
Big enddian:
<pre>
x01 00000000
x02 00000000
x03 00000000
x04 00000001
</pre>

C program to test if your system is either one.

<pre>
#include <stdio.h>

int main() {

}
</pre>
