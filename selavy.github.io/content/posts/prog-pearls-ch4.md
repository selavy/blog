+++
title = "Programming Pearls: Chapter 4"
date = 2019-02-14T18:51:49-08:00
draft = false
tags = []
categories = []
+++

Problems from Column #3: Data Structures Programs

# Lessons

* Consider using formal verification methods on tricky functions
* Loop invariants are a helpful tool

# Problem #1

As laborious as our proof of binary search was, it is still unfinished by some standards. How would you prove that the program is free of run-time errors (such as division by zero, word overflow, variables out of declared range, or array indices out of bounds)? If you have a background in discrete mathematics, can you formatlize the proof in a logical system?

# Problem #2

If the original binary search was too easy for you, try the variant that returns in *p* the position of the first occurence of t in the array x (if there are multiple occurences of t, the original algorithm returns an arbitrary one). Your code should make a logarithmic number of comparisons of array elements; it is possible to do the job in log_2(n) such comparisons.

# Problem #3

Write and verify a recursive binary search program. Which parts of the code and proof stay the same as in the iterative version, and which parts change?

First the iterative version of bsearch:

``` cpp
// NOTE: this is the interface that he describes in the
// book, but a half open interval is definitely the
// better way to code bsearch (see c++ standard lib)
int bsearch(int t, int *x, int n)
{
    /*
     * precondition: x[0] <= x[1] <= ... <= x[n-1]
     * postcondition:
     *     result == -1    => t not present in x
     *     0 <= result < n => x[result] == t
     */
    int l = 0;
    int u = n - 1;
    int m;
    while (l <= u) {
		m = l + ((u - l) / 2);
        if (x[m] < t) {
            l = m + 1;
        } else if (x[m] == t) {
            return m;
        } else { // x[m] > t
            u = m - 1;
        }
    }
    return -1;
}
```

The recursive version. Interesting note: I didn't realize that lambdas can't be used recursively:

``` cpp
int go(int t, int *x, int l, int u)
{
    if (l > u)        return -1;
    int m = l + ((u - l) / 2);
    if (x[m] < t)       return go(t, x, m+1, u);
    else if (x[m] == t) return m;
    else                return go(t, x, l, m-1);
}

int bsearch_recursive(int t, int *x, int n) { return go(t, x, 0, n-1); }
```

# Problem #4

Add fictitious "timing variables" to your binary search program to count the number of comparisons it makes, and use program verification techniques to prove that its run time is indeed logarithmic.

# Problem #5

Prove that this program terminates when its input *x* is a positive integer.

```
while x != 1 do
    if even(x)
        x = x/2
    else
        x = 3*x+1
```

# Problem #6

David Gries calls this the "Coffee Can Problem" in his *Science of Programming*. You are initially given a coffee can that contains some black beans and some white beans and a large pile of "extra" black beans. You then repeat the following process until there is a single bean left in the can.

    Randomly select two beans from the can.
    If they are the same color:
        throw them both out and insert an extra black bean.
    If they are different colors:
        return the white bean to the can and throw out the black.

Prove that the process terminates. What can you say about the color of the final remaining bean as a function of the number of black and white beans originally in the can?

If even white, then black final bean.

# Problem #7

A colleague faced the following problem in a program to draw lines on a bit-mapped display. An array of *n* pairs of reals (a_i, b_i) defined the *n* lines y_i = a_i*x + b_i. The lines were ordered in the x-iunterval [0, 1] in the sense that y_i < y_{i+1} for all values of *i* between 0 and n-2 and all values of *x* in [0,1]. Les formally, the lines don't touch in the vertical slab. Given a point (x,y), where 0<=x<=1, he wanted to determine the two lines that bracket the point. How could he solve the problem quickly?

# Problem #8

Binary search is fundamentally faster than sequential search: to search an *n*-element table, it makes roughtly log_{2}(n) comparisons while sequential search makes roughly n/2. While it is often fast enough, in a few cases binary search must be made faster yet. Although you can't reduce the logarithmic number of comparisons made by the algorithm, can you rewrite the binary search code to be faster? For definiteness, assume that you are to search a sorted table of n=1000 integers.

# Problem #9

As exercises in program verification, precisely specify the input/output behavior of each of the following program fragments and show that the code meets its specification. The first program implements the vector addition a = b + c.

```
i = 0
while i < n
    a[i] = b[i] + c[i]
    i = i + 1
```

input: array b with length n
       array c with length n
       output array a with length n
output: array a s.t. a[i] = b[i] + c[i]

### Inline Answer

```
i = 0
while i < n
    assert i in range(0, n-1)
    a[i] = b[i] + c[i]
    assert a[i] = b[i] + c[i] (as desired)
assert i = n, therefore all indices were visited
```

This function is so straightforward, I'm not really sure what kind of proof is necessary... The only thing really to prove is that i visits all values in the range [0, n-1], which proven by inspection.

(This code and the next two fragments expand the for i=[0, n) loop to a while loop with an increment at the end.) The next fragment computes the maximum value in the array *x*.

```
max = x[0]
i = 1
while i < n do
    if x[i] > max
        max = x[i]
    i = i + 1
```

### Inline Answer

```
max = x[0]
i = 1
while i < n do
    assert i in range(1, n-1)
    assert x is max element in range(0, i-1)
    if x[i] > max
        max = x[i]
    i = i + 1
assert i = n, therefore all indicies 0 .. n-1 were visited
```

I think the key here is the pre-condition that x is max element in the range(0, i-1) at every step, then via proof by induction by we can see that x will be the max value when the loop exits.

This sequential search program returns the position of the first occurrence of *t* in the array x[0..n-1].

```
i = 0
while i < n && x[i] != t
    i = i+1
if i >= n
    p = -1
else
    p = i
```

### Inline Answer

```
i = 0
while i < n && x[i] != t
    assert t not in range(0, i)
assert t == x[i] or t not in x
if i >= n
    assert t not in x
    return -1
else
    p = i
```

The loop invariant is "t not in range(0, i)", and we visit all values of i in the range [0, n).

This program computes the n-th power of x in time proportional to the logarithm of *n*. This recursive program is straightforward to code and to verify; the iterative version is subtle, and is left as an additional problem.

```
function exp(x, n)
        pre n >= 0
        post result = x^n
    if n = 0
        return 1
    else if even(n)
        return square(exp(x, n/2))
    else
        return x*exp(x, n-1)
```

### Inline Answer

```
function exp(x, n)
        pre n >= 0
        post result = x^n
    if n = 0
        return 1
    else if even(n)
        assert n % 2 == 0
        assert x^n == x^(n/2)^2
        return square(exp(x, n/2))
    else
        assert x^n == x*(x^(n-1))
        return x*exp(x, n-1)
```

The loop invariant is result = x^n, which we will decompose until n = 0, at which point we know x^0 = 1. The only thing to really prove is that n will reach 0: since on each iteration either n -> floor(n/2) or n -> n - 1, n will eventually reach 0.

The iterative version of this function is:

```cpp
int exp(int x, int n)
{
    int result = 1;
    for (;;) {
        // loop invariant: exp = x^n
        if (n % 2 == 1) {
            result = result * x;
        }
        n = n / 2;
        if (n == 0) {
            break;
        }
        x = x * x;
    }
    return result;
}
```

# Problem #10

Introduce errors into the binary search function and see whether (and how) they are caught by attempting to verify the buggy code.

# Problem #11

Write and prove the correctness of a recursive binary search in C or C++ with this declaration:

```
int binarysearch(DataType x[], int n)
```

Use this function alone; do not call any other recursive functions.

It is not at all clear to me what this question is asking. The way the function signature is written, it doesn't even take the element that we are searching for. For the recursive binary search, see solution to problem #3.
