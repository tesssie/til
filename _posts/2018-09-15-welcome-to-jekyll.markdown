---
layout: post
title:  Understanding python parallel assignment
date:   2018-09-15 11:46:03 +0530
categories: python
---
## parallel assignment
parallel assignment is the process of assigning more than one variable in parallel.

{% highlight python %}
a, b = 1,2
{% endhighlight %}

A classic use case of parallel assignment swapping in python. If parallel assignment was not there,
we would have to use a temporary variable for swapping. With parallel assignment the code is more elegant and readable.

{% highlight python %}
a = 1
b = 2
a, b = b, a
print (a, b)
#=> print (2, 1)
{% endhighlight %}

## What happens behind the scenes?
A tuple is a collection of values separated by commas. tuples have two operations
* Packing
* Unpacking

### Tuple packing
packing allows to create tuples.

{% highlight python %}
  a = 1,2
  print type(a)
#=> print <type 'tuple'>
{% endhighlight python %}

### Tuple unpacking
Tuples unpacking allows to assign values from tuple to separate variables.

{% highlight python %}
  a, b = 1, 2
  print a
  #=> print 1
  print b
  #=> print 2
{% endhighlight python %}

Using python dis module, I am analysing the opcode to understand how it works.

{% highlight python %}
import dis
a, b, c, d = 1, 2, 3, 4
def parallel_assignment():
  e, f, g, h = a, b, c, d
 dis.dis(parallel_assignment)  
#            0 LOAD_GLOBAL              0 (a)
#            3 LOAD_GLOBAL              1 (b)
#            6 LOAD_GLOBAL              2 (c)
#            9 LOAD_GLOBAL              3 (d)
#            12 BUILD_TUPLE              4
#            15 UNPACK_SEQUENCE          4
#            18 STORE_FAST               0 (e)
#            21 STORE_FAST               1 (f)
#            24 STORE_FAST               2 (g)
#            27 STORE_FAST               3 (h)
#            30 LOAD_CONST               0 (None)
#            33 RETURN_VALUE
{% endhighlight %}

The RHS is evaluated first. The BUILD_TUPLE creates a tuple and pushes the result into the stack. Next the LHS is evaluated
The 'UNPACK_SEQUENCE' pops the tuples and pushes each of elements of stack from right to left into stack. STORE_FAST would store the top
of stack into the variable.

for assignment with two or three variables python would use the stack directly. This is a peephole optimisation in python.
Analysing using dis.

{% highlight python %}
import dis
a, b, c, d = 1, 2, 3, 4
def parallel_assignment():
  c, d = a, b
  print dis.dis(parallel_assignment)
  #print =>     0 LOAD_GLOBAL              0 (a)
  #             3 LOAD_GLOBAL              1 (b)
  #             6 ROT_TWO
  #             7 STORE_FAST                0 (c)
  #             10 STORE_FAST               1 (d)
  #             13 LOAD_CONST               0 (None)
  #             16 RETURN_VALUE
{% endhighlight %}

ROT_TWO opcode would just swap the topmost stack elements.

### Further Reading
* [Introduction to tuples](https://realpython.com/python-lists-tuples/#python-tuples)
* [OpCodes in python](https://docs.python.org/2/library/dis.html)
