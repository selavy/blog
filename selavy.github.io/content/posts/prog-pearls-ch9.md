+++
title = "Programming Pearls: Chapter 9"
date = 2019-02-18T08:30:07-08:00
draft = false
tags = []
categories = []
+++

Problems from Column #9: Code Tuning

Note: I did skip several problems from several columns because I didn't find them particularly useful. Several of the questions here on out are also pretty dated so I'm just going to jump around to the ones that I like.

# Lessons

* Speed comes from knowing your data and specializing.
    ** This comes with what can be a huge cost in terms of maintainability. For most programs this speed is only required in a small part of the program.
* Profile first.

# Problem #3

What special properties of the "juggling" rotation algorithm allowed us to replace the % remainder operator with an *if* statement, and not a more costly *while* statement?

We knew that with the given algorithm, the values would remain in the range [0, 2n) so subtracting *n* would get us back to the range [0, n). Note: if *n* is a power of 2, then x % n ==> x & (n-1), which likely will be faster than the *if* statement and subtraction.

# Problem #6

C and C++ libraries provide character classification functions such as *isdigit*, *isupper*  and *islower* to determine the types of characters. How would you implement these functions.

1) This is probably more complicated with locales, and I think these functions make actually touch singleton locale data, but I'm going to ignore that. It's just interesting if you are using them in a multi-threaded context. 2) For the simple american english ASCII case, we know that these functions only apply to signed 8-bit characters, where the MSB is 0. So I would make a 128 entry lookup table; if the upper bit is set, then return false, other LUT.

# Problem #7

Given a very long sequence (say, billions or trillions) or bytes, how would you efficiently count the total number of one bits? (That is how many bits are turned on in the entire sequence?)

I'd build a 512-entry LUT, which would yield code like:

``` cpp
Sequence bytes;
uint8_t LUT[512]; // contains # of set bits in each value [0, 512)
uint64_t count = 0;
for (int i = 0; i < N; ++i) {
    count += LUT[bytes[i]];
}
```

# Problem #8

How can sentinels be used in a program to find the maximum element in an array?

We can avoid the i < N, most of the time (for many inputs), if we set the last element to INT_MAX, and only compare to INT_MAX *if* vs[i] is greater than the current max. This saves comparisons except in the sorted inputs case, where we only added a small amount of overhead.

```cpp
int find_max(std::vector<int> vs) {
    int N = vs.size();
    int hold = vs[N-1];
    vs[N-1] = INT_MAX;
    int result = INT_MIN;
    for (int i = 0; ; ++i) {
        assert(i < N && "sentinel failed!");
        if (vs[i] > result) {
            if (vs[i] == INT_MAX) {
                break;
            }
            result = vs[i];
        }
    }
    if (hold > result) {
        result = hold;
    }
    vs[N-1] = hold;
    return result;
}
```

# Problem #9

Because sequential search is simpler than binary search, it is usually more efficient for small tables. On the other hand, the logarithmic number of comparisons made by binary search implies that it will be faster than the linear time of sequential search for larger tables. The break-even point is a function of how much each program is tuned. How long and how high can you make that break-even point? What is it on your machine when both programs are equally tuned?

# Problem #10

D. B. Lomet observes that hashing may solve the 1000-integer search problem more efficiently than the tuned binary search. Implement a fast hashing program and compare it to the tuned binary search; how do they compare in terms of speed and space?

# Problem #11

In the early 1960's, Vic Berecz found that most of the time in a simulation program at Sikorsky Aircraft was devoted to computing trigonmetric functions. Further investigation showed that the functions were computed only at integral multiples of five degrees. How did he reduce the run time?

Either cache the results, or make a LUT. You only need 360 / 5 = 72 elements per function.

# Problem #12

One sometimes tunes programs by thinking about mathematics rather than code.

To evaluate the polynomial `y = a_n*x**n + a_{n-1}*x**{n-1} + ... + a_1*x**1 + a_0`

the following code uses 2n multiplications. Give a faster function.

```
y = a[0]
xi = 1
for i = [1, n]
    xi = x * xi
    y = y + a[i]*xi
```

Use horner's method:

```
y = a[n]
for i = [n-1 .. 1] # N.B. backwards
    y *= x
    y += a[i]
```

Like the usual atoi function:

``` c
const char atoi(const char* const s) {
    int result = 0;
    while (*s) {
        result *= 10;
        result += *s++;
    }
    return result;
}
```
