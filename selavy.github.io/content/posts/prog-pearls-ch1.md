+++
title = "Programming Pearls: Chapter 1"
date = 2019-01-19T20:47:53-08:00
draft = false
tags = ["pearls", "exercises"]
categories = ["exercises"]
+++
Some notes and solutions to the Column #1: "Cracking the Oyster" in Programming Pearls by Jon Bentley.

# Lessons

* Ask the right question. Similar to ["5 Whys"](https://en.wikipedia.org/wiki/5_Whys); make sure you understand the real problem.
* Look for simple designs

    > "A designer knows he has arrived at perfection now when there is no longer anything to add, but when there is no longer anything to take away." -- Antoine de Saint-Exupéry

# Problem #1

Q: If memory were not scarce, how would you implement a sort in a language with libraries for representing and sorting sets?

This solution only works if no duplicates or don't care about them, but actually mirrors the solution that Jon gives so I think the question is poorly worded.

``` cpp
std::vector<int> set_sort(const std::vector<int>& input)
{
    std::set<int> s;
    for (int v: input) {
        s.insert(v);
    }
    std::vector<int> rv;
    for (int v: s) {
        rv.push_back(v);
    }
    return rv;
}
```

# Problem #2
Q: How would you implement bit vectors using bitwise logical operators (such as and, or and shift)?

In a C-style:

``` cpp
[[nodiscard]]
constexpr uint32_t make_mask(size_t bit) noexcept
{
    assert(bit < 32);
    return static_cast<uint32_t>(1u) << bit;
}

[[nodiscard]]
constexpr uint32_t setbit(uint32_t bitfield, size_t bit) noexcept
{
    return bitfield | make_mask(bit);
}

[[nodiscard]]
constexpr uint32_t clearbit(uint32_t bitfield, size_t bit) noexcept
{
    return bitfield & ~make_mask(bit);
}

[[nodiscard]]
constexpr bool isset(uint32_t bitfield, size_t bit) noexcept
{
    return (bitfield & make_mask(bit)) != 0;
}
```

In C++-style, which supports bitvectors that are larger than 64-bits:

``` cpp
namespace detail {

template <size_t Bytes>
class Storage {
public:
    constexpr Storage() noexcept : val{0u} {}

    constexpr void set(size_t bit) noexcept
    {
        const size_t idx = bit / 8;
        const size_t off = bit % 8;
        val[idx] |= mask(off);
    }

    [[nodiscard]]
    constexpr bool isset(size_t bit) noexcept
    {
        const size_t idx = bit / 8;
        const size_t off = bit % 8;
        return (val[idx] & mask(off)) != 0u;
    }

    constexpr void clear() noexcept
    {
        for (int i = 0; i < n_bytes; ++i) {
            val[i] = 0u;
        }
    }

    [[nodiscard]]
    constexpr bool none() noexcept
    {
        for (int i = 0; i < n_bytes; ++i) {
            if (val[i] != 0) {
                return false;
            }
        }
        return true;
    }

    [[nodiscard]]
    constexpr bool all() noexcept
    {
        const uint8_t allset = ~uint8_t{0};
        for (int i = 0; i < n_bytes; ++i) {
            if (val[i] != allset) {
                return false;
            }
        }
        return true;
    }

private:
    [[nodiscard]]
    constexpr uint8_t mask(size_t off) noexcept
    {
        assert(off < sizeof(uint8_t)*8);
        return uint8_t{1} << off;
    }

    static constexpr size_t n_bytes = Bytes;
    std::array<uint8_t, n_bytes> val;
};

template <class T>
struct StorageBase {
    using value_type = T;
    static_assert(std::is_unsigned_v<T> == true, "Type must be unsigned");

    constexpr StorageBase() noexcept : val{0} {}

    constexpr void set(size_t bit) noexcept
    {
        val |= mask(bit);
    }

    [[nodiscard]]
    constexpr bool isset(size_t bit) noexcept
    {
        return (val & mask(bit)) != 0u;
    }

    constexpr void clear() noexcept
    {
        val = 0;
    }

    [[nodiscard]]
    constexpr bool none() noexcept
    {
        return val == value_type{0};
    }

    [[nodiscard]]
    constexpr bool all() noexcept
    {
        return val == ~value_type{0};
    }

private:
    [[nodiscard]]
    constexpr value_type mask(size_t off) noexcept
    {
        assert(off < sizeof(value_type)*8);
        return value_type{1} << off;
    }

    value_type val;
};

template <> class Storage<1> : public StorageBase<uint8_t>  {};
template <> class Storage<2> : public StorageBase<uint16_t> {};
template <> class Storage<4> : public StorageBase<uint32_t> {};
template <> class Storage<8> : public StorageBase<uint64_t> {};

} // namespace detail

template <size_t MaxIndex>
class BitVector : public detail::Storage<MaxIndex / 8> {};

// usage
TEST_CASE("Set and clear bits first BitVector", "[column1_2]")
{
    BitVector<32> bv;
    REQUIRE(bv.isset(0) == false);
    REQUIRE(bv.none() == true);
    REQUIRE(bv.all() == false);

    bv.set(4);
    assert(bv.isset(4) == true);
    assert(bv.isset(3) == false);
    assert(bv.none() == false);
    assert(bv.all() == false);
}
```

# Problem #3
Q: Run-time efficiency was an important part of the design goal, and the resulting program was efficient enough. Implement the bitmap sort on your system and measure its run time; how does it compare to the system sort and to the sorts in Problem 1? Assume that *n*=10,000,000, and that the input file contains 1,000,000 integers.

## BitMap Sort

``` cpp
#include <cstdio>
#include <cstdlib>
#include "bit_vector.h" // from problem #2

constexpr size_t MaxValue = 10000000;

int main(int argc, char** argv) {
    BitVector<MaxValue> vec;
    int val;
    char input[256];
    while (fgets(&input[0], sizeof(input), stdin) != NULL) {
        val = atoi(&input[0]);
        vec.set(val);
    }
    for (int i = 0; i < MaxValue; ++i) {
        if (vec.isset(i)) {
            printf("%d\n", i);
        }
    }
    return 0;
}
```

## Timings

| Method              | Time (sec) |
|:--------------------|------------|
| Debug Set Sort      | 1.588      |
| Release Set Sort    | 0.705      |
| Debug Bitmap Sort   | 0.133      |
| Release Bitmap Sort | 0.118      |
| System Sort         | 0.973      |

# Problem 4

If you take Problem 3 seriously, you will face the problem of generating *k* integers less than *n* without duplicates.  The simplest approach uses the first *k* positive integers.  This extreme data set wo't alter the run time of the bitmap method by much, but it might skew the run time of a system sort. How could you generate a file of k unique random integers between 0 and *n* - 1 in random order? Strive for a short program that is also efficient.

## Numpy solution

``` python
import numpy as np

M = 10000000
N = 1000000
vals = np.random.choice(range(M), N, replace=False)
print('\n'.join([str(x) for x in vals]))
```

## C++ Solution

``` cpp
#include <cstdio>
#include <random>

int main(int argc, char** argv)
{
    const int k = 10000000;
    const int n = 1000000;

    // generate numbers 0..k-1
    std::vector<int> range;
    range.reserve(k);
    for (int i = 0; i < k; ++i) {
        range.push_back(i);
    }

    // pick 2 random indices and swap k/2 times
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<> dis(0, k-1);
    for (int i = 0; i < k / 2; ++i) {
        int p1 = dis(gen);
        int p2 = dis(gen);
        std::iter_swap(&range[p1], &range[p2]);
    }

    // take first n elements as output
    for (int i = 0; i < n; ++i) {
        printf("%d\n", range[i]);
    }

    return 0;
}
```

# Problem 5

The programmer said that he had about a megabyte of free storage, but the code we sketched uses 1.25 megabytes. He was able to scrounge the extra space without much trouble. If the megabyte had been a hard and fast boundary, what would you have recommended? What is the run time of your algorithm?

My initial thought is to just make multiple passes over the input file.  This technique doesn't work if you need an online algorithm though.

``` cpp
#include <cstdio>
#include <cstdlib>
#include <cstdint>
#include <cstring>

constexpr int Limit = 1 << 20; // 1 MB
constexpr int MaxValue = 10000000;
int main(int argc, char** argv)
{
    if (argc != 2)
    {
        fprintf(stderr, "Usage: %s <FILE>\n", argv[0]);
        exit(1);
    }

    FILE* fp = fopen(argv[1], "r");
    if (!fp)
    {
        fprintf(stderr, "Unable to open file: '%s'\n", argv[1]);
        exit(1);
    }

    uint32_t bitmap[Limit/32];
    char input[256];
    int val;
    for (int base = 0; base < MaxValue; base += Limit)
    {
        rewind(fp);
        memset(&bitmap[0], 0, sizeof(bitmap));
        const int lo = base;
        const int hi = base + Limit;

        while (fgets(&input[0], sizeof(input), fp) != NULL)
        {
            val = atoi(&input[0]);
            if (!(val >= lo && val < hi))
                continue;
            val -= base;
            int idx = val / 32;
            int off = val % 32;
            bitmap[idx] |= static_cast<uint32_t>(1) << off;
        }

        for (int i = 0; i < Limit; ++i)
        {
            int idx = i / 32;
            int off = i % 32;
            if ((bitmap[idx] & (static_cast<uint32_t>(1) << off)) != 0)
            {
                printf("%d\n", i + base);
            }
        }
    }

    fclose(fp);
    return 0;
}
```

# Problem 6

What would you recommend to the programmer if instead of saying that each integer could appear at most once, he told you that each integer could appear at most ten times? How would your solution change as a function of the amount of available storage?

Idea is to keep 10 million 4-bit cells, each time a number is seen, increment the appropriate cell.

## Implementation #1: bitfields using explicit widths

We can let the compiler generate the shifts and masks for us.

```cpp
#include <cstdio>
#include <cstdlib>
#include <cstdint>
#include <cassert>
#include <cinttypes>
#include <cstring>
#include <memory>

constexpr int MaxValue = 10000000;
constexpr int Elements = MaxValue / 2; // using 4-bits per number

struct EE {
    uint8_t lo:4; // even
    uint8_t hi:4; // odd
};
static EE cnts[Elements];


int main(int argc, char** argv) {
    char input[256];
    memset(&cnts[0], 0, sizeof(cnts));
    while (fgets(&input[0], sizeof(input), stdin) != NULL) {
        int val = atoi(&input[0]);
        int idx = val / 2;
        if (val % 2 == 0) {
            ++cnts[idx].lo;
            assert(cnts[idx].lo <= 10 && "More than 10 duplicates!");
        } else {
            ++cnts[idx].hi;
            assert(cnts[idx].hi <= 10 && "More than 10 duplicates!");
        }
    }

    for (int i = 0; i < Elements; ++i) {
        for (int p = 0; p < cnts[i].lo; ++p) {
            printf("%d\n", 2*i);
        }
        for (int p = 0; p < cnts[i].hi; ++p) {
            printf("%d\n", 2*i+1);
        }
    }

    return 0;
}

```

## Implementation #2: shift and mask by hand

``` cpp
static uint8_t cnts[Elements];

int main(int argc, char** argv) {
    char input[256];
    memset(&cnts[0], 0, sizeof(cnts));
    while (fgets(&input[0], sizeof(input), stdin) != NULL) {
        const int val = atoi(&input[0]);
        const int idx = val / 2;
        uint8_t lo = (cnts[idx] >> 0) & 0x0Fu;
        uint8_t hi = (cnts[idx] >> 4) & 0x0Fu;
        if (val % 2 == 0) {
            ++lo;
        } else {
            ++hi;
        }
        cnts[idx] = (hi << 4) | (lo << 0);
    }

    for (int i = 0; i < Elements; ++i) {
        int lo = (cnts[i] >> 0) & 0x0Fu;
        int hi = (cnts[i] >> 4) & 0x0Fu;
        for (int p = 0; p < lo; ++p) {
            printf("%d\n", 2*i);
        }
        for (int p = 0; p < hi; ++p) {
            printf("%d\n", 2*i+1);
        }
    }

    return 0;
}
```


## Implementation #3: Safety off

Since we are storing the odd-value counters in the upper 4-bits of each byte, we can take advantage of the fact that adding 2^4 (=16) will never increment bits in the lower nibble (except in the case of overflow).  This is analogous to adding 10^x to a base 10 number; repeatedly adding 10,000 to a number will never change the bottom 4 digits.

This method is slightly faster, but is less safe because we aren't verifying that there aren't more than 10 duplicates.

``` cpp
static uint8_t cnts[Elements];

int main(int argc, char** argv) {
    char input[256];
    memset(&cnts[0], 0, sizeof(cnts));
    while (fgets(&input[0], sizeof(input), stdin) != NULL) {
        const int val = atoi(&input[0]);
        const int idx = val / 2;
        if (val % 2 == 0) {
            cnts[idx] += 1u << 0;
        } else {
            cnts[idx] += 1u << 4;
        }
        assert(((cnts[idx] >> 0) & 0x0F) <= 10 && "More than 10 duplicates!");
        assert(((cnts[idx] >> 4) & 0x0F) <= 10 && "More than 10 duplicates!");
    }

    for (int i = 0; i < Elements; ++i) {
        int lo = (cnts[i] >> 0) & 0x0F;
        int hi = (cnts[i] >> 4) & 0x0F;
        for (int p = 0; p < lo; ++p) {
            printf("%d\n", 2*i);
        }
        for (int p = 0; p < hi; ++p) {
            printf("%d\n", 2*i+1);
        }
    }

    return 0;
}
```

## Additional Thoughts

I thought at first that it should be possible to use a struct like:
```cpp
struct EE {
    uint8_t count: 4;
} __attribute__((packed));
```

but that doesn't appear to be the case as you can see in this [Compiler Explorer](https://gcc.godbolt.org/z/UVcr2y) example. I did stumble on a [library](https://github.com/gpakosz/PackedArray) that might help with this situation, but I didn't really explore it.

# Problem 7

[R. Weil] The program as sketeched has several flaws.  The first is that it assumes that no integer appears twice in the input. What happens if one does show up more than once? How could the program be modified to call an error function in that case? What happens when an input integer is less than zero or greater than or equal to n? What if an input is not numeric? What should a program do under those circumstances? What other sanity checks could the program incorporate? Describe small data sets taht test the program, including its proper handling of these and other ill-behaved cases.

# Problem 8

When the programmer faced the problem, all toll-free phone numbers in the United States had the 800 area code. Toll-free codes now include 800, 877 and 888, and the list is growing. How would you sort all of the toll-free numbers using only a megabyte? How can you store a set of toll-free numbers to allow very rapid lookup to determine whether a given toll-free number is available or already taken?

# Problem 9

One problem with trading more space to use less time is that intializing the space can itself take a great deal of time. Show how to circumvent this problem by designing a technique to initialize an entry of a vector to zero the first time it is accessed. Your scheme should use constant time for initialization and for each vector access, and use extra space proportional to the size of the vector. Because this method reduces initialization time by using even more space, it should be considered only when space is cheap, time is dear and the vector is sparse.

``` cpp
struct Array {
    // 3 parallel arrays
    int* data;
    int* to;
    int* from;
    // next available element
    int top{0};

    void initialize(size_t i, int value) noexcept
    {
        from[i] = top;
        to[top] = i;
        data[i] = value;
        ++top;
    }

    bool is_initialized(size_t i) const noexcept
    {
        return from[i] < top && to[from[i]] == i;
    }
};
```

# Problem 10

Before the days of low-cost overnight deliveries, a store allowed customers to order items over the telephone, which they picked up a few days later. The store's database used the customer's telephone number as the primary key for retrieval (customers know their phone numbers and the keys are close to unique). How would you organize the store's database to allow orders to be inserted and retrieved efficiently?

Use last n-digits as a hash.

# Problem 11

In the early 1980's Lockheed engineers transmitted daily a dozen drawings from a Computer Aided Design (CAD) system in their Sunnyvale, California, plant to a test station in Santa Cruz. Although the facilities were just 25 miles apart, an automobile courier service took over an hour (due to traffic jams and mountain roads) and cost a hundred dollars per day. Propose alternative data transmission schemes and estimate their cost.

# Problem 12

Pioneers of human space flight soon realized the need for writing implements that work well in the extreme environment of space. A popular urban legend asserts that the United States National Aeronautics and Space Administration (NASA) solved the problem with a million dollars of research to develop a special pen. According to the legend, how did the Soviets solve the same problem?

A pencil.