+++
title = "Programming Pearls: Chapter 2"
date = 2019-01-22T19:01:57-08:00
draft = false
tags = ["pearls", "exercises"]
categories = ["exercises"]
+++

<head>
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
</head>

Problems for Column #2: "Aha! Algorithms"

# Lessons

* Break problem down into primitives for which there are known algorithms.
* "A Problem Solver's Perspective": Good programmers are a little bit lazy: they sit back and wait for an insight rather than rushing forward with their first idea.

# Problem #1

Consider the problem of finding all the anagrmas of a given input word. How would you solve this problem given only the word and the dictionary? What if you could spend some time and space to process the dictionary before answering any queries?

The key is to generate a hash for each word that is the same regardless of order of the letters so that anagrams will hash to the same value. The most straightforward solution is to sort the letters.

``` cpp
using Dictionary = std::vector<std::string>;
const Dictionary dict = {
    "deposit",
    "dopiest",
    "posited",
    "topside",
    "peter",
    "word",
    "sail",
    "smile",
    "ant",
    "tan",
};

auto&& sign = [](std::string w) -> std::string {
    std::sort(w.begin(), w.end());
    return w;
};

using Anagrams = std::vector<std::string>;

Anagrams find_anagrams(const Dicionary& dict, std::string word)
{
    Anagrams anagrams;
    const std::string sig = sign(word);
    for (auto&& w: dict) {
        if (sig == sign(w)) {
            anagrams.push_back(w);
        }
    }
    return anagrams; // NOTE: should get NRVO
};
```

If you can preprocess the data, then it is worth setting up a map of Hash -> Vector[Words]. Then you can find the anagrams in O(1).


# Problem #2

Given a sequential file containing 4,300,000,000 32-bit integers, how can you find one that appears at least twice?

4.3 TB * 4B ~= 16TB so we can't hold all the values in memory. First sort the integers (probably using merge sort).  Then, traverse the integers and look for any values that are equal to the previous value.

# Problem #3

We skimmed two vector rotation algorithms that require subtle code; implement each as a program.  How does the greatest common divisor of i and n appear in each program?


## Rotate implementation:

* C++ Technique: Use a tag to dispatch to different implementations of your algorithmn based on what iterator concept you've been passed.
* C++ Technique: STL has a std::iter_swap(), which makes several of these algorithms much cleaner.
* Comment: C++ function signatures are getting so noisy, especially when you have attribute like [[nodiscard]], constexpr, and conditional noexcept.
    * I'm still trying to find a formatting style that I like for this.

``` cpp
namespace my {

template <typename BidirectionalIt>
void reverse(BidirectionalIt first,
             BidirectionalIt last,
             std::bidirectional_iterator_tag
             ) noexcept(noexcept(std::iter_swap(first, last)))
{
    while ((first != last) && (first != --last)) {
        std::iter_swap(first++, last);
    }
}


template <typename RandomAccessIt>
void reverse(RandomAccessIt first, RandomAccessIt last,
             std::random_access_iterator_tag
             ) noexcept(noexcept(std::iter_swap(first, last)))
{
    if (first == last) {
        return;
    }
    while (first < --last) {
        std::iter_swap(first++, last);
    }
}

template <typename BidirectionalIt>
void reverse(BidirectionalIt first,
             BidirectionalIt last
             ) noexcept(noexcept(std::iter_swap(first, last)))
{
    using iterator_category = typename std::iterator_traits<BidirectionalIt>::iterator_category;
    reverse(first, last, iterator_category());
}

} // namespace my
```

## Rotate Implementation #1: using rotate

``` cpp
template <class BidirectionalIt>
void rotate(BidirectionalIt first,
            BidirectionalIt n_first,
            BidirectionalIt last
            ) noexcept(noexcept(my::reverse(first, n_first)))
{
    my::reverse(first, n_first);
    my::reverse(n_first, last);
    my::reverse(first, last);
}
```

## Rotate Implementation #2: shufflin'

``` cpp
template <class I>
I gcd(I m, I n) noexcept
{
    while (n != 0) {
        I t = m % n;
        m = n;
        n = t;
    }
    return m;
}

// NOTE: Only works for random access iterators
template <class It>
void rotate2(It first, It n_first, It last) noexcept
{
    using Distance = typename std::iterator_traits<It>::difference_type;
    using ValueType = typename std::iterator_traits<It>::value_type;

    Distance n = last - first;
    Distance i = n_first - first;
    Distance cycles = gcd(i, n);
    for (Distance u = 0; u < cycles; ++u) {
        Distance j = i + u;
        Distance k = j + i;
        Distance z = k % n;
        ValueType t = *(first + u);
        *(first + u) = *(first + j);
        do {
            *(first + j) = *(first + z);
            j = z;
            k += i;
            z = k % n;
        } while (z != u);
        *(first + j) = t;
    }
}
```

In this algorithm, gcd gives the number of shuffles that are required. Note, that the STL has implementations for std::forward_iterator as well.

# Problem 4

Several readers pointed out that while all three rotation algorithm require time proportional to n, the juggling algorithm is apparently twice as fast as the revresal algorithm: it stores and retrieves each element of the array just once, while the reversal algorithm does so twice. Experiment with the functions to compare their speeds to real machines; be especially sensitive to issues surrounding the locality of memory references.

# Problem 5

Vector rotation functions change the vector _ab_ to _ba_; how would you transform the vector _abc_ to _cba_? (This models the problem of swapping nonadjacent blocks of memory).

Use the fact that $$(abc)^r = (a^rb^rc^r)^r$$

# Problem 6

In the late 1970's, Bell Labs deployed a "user-operated directory assistance" program that allowed employeeds to look up a number in a company telephone directory using a standard push-button telephone.

![Push Button Telephone image](/img/push_button_telephone.png)

This problem is solved just like the anagram problem.

``` cpp
// NOTE: other time vs. space tradeoffs exist, could build full
//       256 entry to avoid AND and sub
const char ButtonMap[64] = {
    '0', '2', '2', '2', '3', '3', '3', '4',
    '4', '4', '5', '5', '5', '6', '6', '6',
    '7', '7', '7', '7', '8', '8', '8', '9',
    '9', '9', '9', '0', '0', '0', '0', '0',
    '0', '2', '2', '2', '3', '3', '3', '4',
    '4', '4', '5', '5', '5', '6', '6', '6',
    '7', '7', '7', '7', '8', '8', '8', '9',
    '9', '9', '9', '0', '0', '0', '0', '0',
};

[[nodiscard]] char Lookup(char c) noexcept
{
    return ButtonMap[(c & 0x7F) - 64]; // to be safe, clear the top bit
}

[[nodiscard]]
std::string telesign(std::string first, std::string last) noexcept
{
    std::string rv;
    rv.reserve(last.size() + 3);
    for (auto c: last) {
        rv += Lookup3(c);
    }
    rv += '*';
    if (!first.empty()) {
        rv += Lookup(first[0]);
    }
    rv += '*';
    return rv;
}
```

# Problem 7

In the early 1960's, Vic Vyssotsky worked with a programmer who had to transpose a 4000-by-4000 matrix stored on magnetic tape (each record had the same format in several dozen bytes). The original program his colleague suggested would have taken fifty hours to run; how did Vyssotsky reduce the run time to half an hour?

Answer given in book: If matrix is row-major, then prepend the column and row to each element, sort, remove the column and row.

# Problem 8

[J. Ullman] Given a set of _n_ real numbers, a real number _t_, and an integer _k_, how quickly can you determine whether there exists a k-element subset of the set that sums to at most _t_?

Sum the smallest _k_ elements in the set and compare to _t_.  Should be able to use quick select, but could just use sort then choose _k_ elements.

# Problem 9

Sequential search and binary search represent a tradeoff between search time and preprocessing time. How many binary searches need be performed in an _n_-element table to buy back the preprocessing time required to sort the table?

Sorting is O(nlog(n)), Linear search is O(n), Binary Search is O(log(n)).

$$s = Searches Needed$$

$$s*lg(n) + n*lg(n) < s*n$$

$$n*lg(n) < s*n - s*lg(n)$$

$$s > n*lg(n)/(n-lg(n))$$


Need O(n*lg(n) / (n - lg(n))) searches to make it worth it (ignoring constant factors, which is probably not correct).

# Problem 10

On the day a new research reported to work for Thomas Edison, Edison asked him to compute the volume of an empty light bulb shell. After several hours with calipers and calculus, the fresh hire returned with the answer of 150 cubic centimeters. In a few seconds, Edison computed and responded "closer to 155" -- how did he do it?

Fill the light bulb with water.
