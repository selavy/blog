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

If you take Problem 3 seriously, you will face the problem of generating *k* integers less than *n* without duplicates.  The simplest approach uses the first *k* positive integers.  This extreme data set wo't alter the run time of the bitmap method by much, but it might skew the run time of a system sort. How could you generate a file of k unique random integers between 0 and *n* - 1 in random order? Strive for a short program that is also efficient.

## Generating *k* unique random integers between 0 and *n* - 1

Cheating with numpy:

``` python
import numpy as np

M = 10000000
N = 1000000
vals = np.random.choice(range(M), N, replace=False)
print('\n'.join([str(x) for x in vals]))
```

Actually solving it with C++:

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