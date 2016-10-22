+++
title = "Calculating approximate logarithms"
description = """
  Logarithms are surprisingly useful, but can be quite expensive to calculate.
  In this post we explore an interesting method for approximating the value of
  base 2 logarithms extremely quickly."""
date = "2015-05-06T00:00:00.000Z"
categories = ["misc"]
tags = ["optimisation"]
comments = false
draft = false
showpagemeta = true
slug = ""
+++

There are quite a few instances in which one may wish to place values on a
logarithmic scale. One such case was encountered during an
[FPGA](https://en.wikipedia.org/wiki/Field-programmable_gate_array)
class at university where we were asked to display the amplitude response of
an FM radio frequency band. It turns out that there are some neat tricks which
make calculating approximate logs very efficient in this kind of scenario.

Here I will outline an algorithm for calculating approximate base 2 logs
quickly, focusing on an implementation for microprocessors. Code
examples are in Ruby for ease of reading.

### Course grain

Calculating the integer base 2 log of a number is as simple as finding the
position of its most-significant '1' bit. This is a happy result of how the
binary system of representing numbers works, and it shouldn't require too much
thought for you to convince yourself that this is indeed the case.

So, to calculate a course log, all we need to do is find the position of the
highest set bit!

You might already be thinking of a few ways to do this:

```ruby
# String conversion technique
number = 342
binary = number.to_s(2)
course_log = binary.length - 1
```

```ruby
# Bit shifting technique
number = 342
course_log = 0
while number != 0
  course_log += 1
  number >>= 1
end
course_log -= 1
```

Definitely avoid using the first technique, and you probably shouldn't use the
second either. Why? These days many processors have
[an assembly instruction](http://en.wikipedia.org/wiki/Find_first_set)
which can achieve the desired result much more efficiently.

For example, on modern CPUs you can use the BSR (bit scan reverse)
instruction and call it a day:

```x86asm
bsr RAX, number;
```

### Improving precision

In the previous section we demonstrated that a course log can taken using a
single assembly instruction. This might be enough for some applications, but
many others will require a little more precision.

To achieve more precision in our results we will use *n* bits following the
most significant '1' as a key to a lookup table. The value from the lookup
table will be added to our course log to get a more precise result. The larger
the value of *n* the more precise the result, but be aware that the size of
the lookup table grows exponentially (it will have 2<sup>n</sup> entries).

```ruby
n = 5
key = (number & ~(1 << course_log)) >> (course_log - n)
refined_log = course_log + lookup_table[key]
```

Fantastic! But what is `lookup_table`?

The lookup table should be a constant, hard-coded array of length 2<sup>n</sup>
which contains the decimal parts of logs. We can pre-calculate this table by
taking 2<sup>n</sup> linear steps between two adjacent powers of two
(in this case 1 and 2), taking exact logs (using an existing math library) and
recording the effect it has on the decimal portion fo the number.

```ruby
lookup_table = (0...2**n).map do |key|
  k = 1 + (key / 2.0**n)
  Math.log(k) / Math.log(2)
end
```

### Tying it all together

```ruby
N = 5

# Can be precomputed and hard-coded
LOOKUP_TABLE = (0...2**N).map do |key|
  k = 1 + (key / 2.0**N)
  Math.log(k) / Math.log(2)
end

def fast_log(number)
  # No inline assembly for Ruby, use bitshifts to find highest set bit
  tmp = number
  course_log = 0
  while tmp != 0
    course_log += 1
    tmp >>= 1
  end
  course_log -= 1

  # Improve precision
  key = (number & ~(1 << course_log)) >> (course_log - N)
  refined_log = course_log + LOOKUP_TABLE[key]

  return refined_log
end
```

### Other improvements

The techniques described here can easily be extended to work with fixed-point
arithmetic, which is a big improvement on hardware which does not have
native support for floating-point numbers. The lookup table can also be tweaked
in such a way that the value returned represents a mid-point, giving better
results on average (this will cause logs which normally return integers to be
slightly off, which may matter depending on your use case).

Finally, I discovered after writing this post that there's
[an article](http://www.ebaytechblog.com/2015/05/01/fast-approximate-logarithms-part-i-the-basics/)
about fast approximate logarithms on eBay Tech Blog. If you are using a CPU
where floating point arithmetic is cheap, you may want to consider using a
mathematical function similar to that described in the article (instead of a
lookup table) for greater accuracy at the cost of speed.
