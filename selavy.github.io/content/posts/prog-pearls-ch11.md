+++
title = "Programming Pearls: Chapter 11"
date = 2019-02-26T19:32:27-08:00
draft = false
+++

Problems from Column #11: Sorting

First implementation of quicksort:

``` cpp
template <class I, class C>
bool is_partitioned(I begin, I end, I mid, C cmp) {
    while (begin < end) {
        if (!cmp(*begin++, *mid)) return false;
    }
    return true;
}

template <class I>
I partition(I begin, I end) {
    // partition [begin, end) around *begin
    auto lo = begin + 1;
    auto hi = end - 1;
    while (lo < hi) {
        // invariant: [begin + 1, lo ) <= *begin
        // invariant: (hi       , end) >= *begin
        assert(is_partitioned(begin + 1, lo , begin, std::less_equal<>{}));
        assert(is_partitioned(hi    + 1, end, begin, std::greater_equal<>{}));

        if (!(*lo > *begin)) {
            ++lo;
        } else if (!(*hi < *begin)) {
            --hi;
        } else {
            std::iter_swap(lo++, hi--);
        }

        assert(is_partitioned(begin + 1, lo , begin, std::less_equal<>{}));
        assert(is_partitioned(hi    + 1, end, begin, std::greater_equal<>{}));
    }

    auto mid = lo - 1;
    if (*begin > *lo) {
        mid = lo;
    }
    std::iter_swap(begin, mid);

    // assert is partitioned
    assert(is_partitioned(begin  , mid, mid, std::less_equal<>{}));
    assert(is_partitioned(mid + 1, end, mid, std::greater_equal<>{}));

    return mid;
}

template <class I>
void qsort(I begin, I end) {
    auto count = end - begin;
    if (count <= 1) {
        return;
    }
    auto mid = partition(begin, end);
    qsort(begin, mid);
    qsort(mid + 1, end);
}
```

Note to self: don't get lazy. Was easy to establish the loop invariants for partiion(), but I then took too much time correctly setting mid. Just slow down to work it out.

Pretty cool that it's easy with C++14 generic lambdas to use std::is_partitioned(). My tests cases looked like:

``` cpp
TEST_CASE("Partition Tests", "[partition]") {
    std::vector<std::vector<int>> tests = {
        { 5, 3, 6, 4, 8, 9 },
        { 1, 1, 1, 1, 1, 1 },
        { 5, 3, 6, 4, 8, 9, 3 },
        { 1, 2 },
        { 1, 2, 3 },
    };

    for (auto&& vs : tests) {
        auto mid = ch11::partition2(vs.begin(), vs.end());
        REQUIRE(std::is_partitioned(vs.begin(), vs.end(), [mid](auto x) { return x < *mid; }) == true);
    }
}
```

Note that in C++ you can still do it with decltype, it's just ugly:
``` cpp
REQUIRE(std::is_partitioned(vs.begin(), vs.end(), [mid](decltype(*vs.begin()) x) { return x < *mid; }) == true);
```

But probably the better option is:

``` cpp
template <class Iter>
bool IsPartitioned(Iter begin, Iter end) {
    using ValueType = typename std::iterator_traits<Iter>::value_type;
    return std::is_partitioned(begin + 1, end, [begin](ValueType x) { return x < *begin; });
}
```

Another version of partition:

``` cpp
template <class I>
I partition(I begin, I end) {
    auto lo = begin;
    auto hi = end;
    for (;;) {
        do { ++lo; } while (lo < end && *lo < *begin);
        do { --hi; } while (            *hi > *begin);
        if (!(lo < hi)) {
            break;
        }
        std::iter_swap(lo, hi);
    }
    std::iter_swap(begin, hi);
    return hi;
}
```

# Problem #1

Like any other powerful tool, sorting is often used when it shouldn't be and not used when it should be. Explain how sorting could be overused or underused when calculating the following statistics of an array of *n* floating point numbers: minimum, maximum, mean, median, and mode.

minimum, maximum, mode, and mean can all be calculated online so no sorting needed. Median can be calculated with the quickselect algorithm, but sorting isn't a completely unreasonable implementation.

Very, very simple implementation of mode:
``` cpp
double calc_mode(std::vector<double> vs) {
    std::map<double, int> m;
    for (auto v : vs) {
        m[v]++;
    }
    auto it = std::max_element(m.begin(), m.end(), [](auto x, auto y) { return x.second < y.second; });
    return it->first;
}
```

``` cpp
double calc_mean(std::vector<double> vs) {
    double sum = std::accumulate(vs.begin(), vs.end(), 0.);
    return sum / vs.size();
    }
```

# Problem #2

Speed up Lomuto's partition scheme by using x[l] as a sentinel. Show how this scheme allows you to remove the *swap* after the loop

# Problem #3

How could you experiment to find the best value of *cutoff* on a particular system?

# Problem #4

Although Quicksort uses only *O(log n)* stack space on the average, it can use linear space in the worst case. Explain why, then modify the program to use only logarithmic space in the worst case.

# Problem #5

Show how to use Lomuto's partitioning scheme to sort varying-length bit strings in time proportion to the sum of their lengths.

# Problem #6

Use the techniques of this column to implement other sorting algorithms. Selection sort first places the small value in x[0], then the smallest raemaining value in x[1], and so forth. Shell sort (or "diminishing increment sort") is like Insertion sort, but moves elements down *h* positions rather than just one position. The value of *h* starts large, and shrinks.

Note to self: implement merge sort and timsort

# Problem #9

Write a program for finding the kth-smallest element in the array *x* in O(n) expected time. Your algorithm may permute the elements of *x*.

This is the quickselect algorithm.


