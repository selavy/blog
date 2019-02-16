+++
title = "Programming Pearls Chapter 4"
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
int bsearch2(int t, int *x, int n)
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
        // m = (l + u) / 2; // l + ((u - l) / 2)
	    // to be correct for overflow:
		m = l + ((u - l) / 2)
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

The recursive version (I didn't realize that lambdas can't be used recursively):

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

    Randomly select two beans from the can. If they are the same color, throw them both out and insert an extra black bean. If they are different colors, return the white bean to the can and throw out the black.

Prove that the process terminates. What can you say about the color of the final remaining bean as a function of the number of black and white beans originally in the can?

If even white, then black final bean.

# Problem #7

A colleague faced the following problem in a program to draw lines on a bit-mapped display. An array of *n* pairs of reals (a_i, b_i) defined the *n* lines y_i = a_i*x + b_i. The lines were ordered in the x-iunterval [0, 1] in the sense that y_i < y_{i+1} for all values of *i* between 0 and n-2 and all values of *x* in [0,1]. Les formally, the lines don't touch in the vertical slab. Given a point (x,y), where 0<=x<=1, he wanted to determine the two lines that bracket the point. How could he solve the problem quickly?

