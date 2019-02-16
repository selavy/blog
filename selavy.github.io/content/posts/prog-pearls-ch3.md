+++
title = "Programming Pearls: Chapter 3"
date = 2019-01-31T20:28:30-08:00
draft = false
tags = []
categories = []
+++

<head>
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
</head>

Problems from Column #3: Data Structures Programs

# Lessons

* Proper view of data structures programs
** The data a system is to process gives deep insight into a good module structure
** Understand the input, the output, and the intermediate data structures
* Separate data from control (e.g. Model+View+Controller)
* Don't write a big program when a little one will do
** The more general problem may be easier to solve (Polya's "Inventor's Paradox")

# Problem #1

As the second edition of this book goes to press, individual income in the United States is taxed at five different rates, the maximum of which is around forty percent. The situation was formerly more complicated, and more expensive. A programming text the following twenty-five *if* statements as a reasonable approach for calculating the 1978 United States Federal Income Tax. The rate sequence .14, .15, .16, .17, .18, ... exhibits jumps larger than .01 later in the sequence. Any comments?

```
if income <= 2200
    tax = 0
else if income <= 2700
    tax =         .14 * (income - 2200)
else if income <= 3200
    tax =    70 + .15 * (income - 2700)
else if income <= 3700
    tax =   145 + .16 * (income - 3200)
else if income <= 4200
    tax =   225 + .17 * (income - 3700)
    ...
else
    tax = 53090 + .70 * (income - 102200)
```

First thing to notice is that every calculation can be formulated as ```tax = base[i] + rate[i] * (income - offset[i])```. So the easiest solution is to set up a table of

| Limit   | Base  | Rate | Offset |
|---------|-------|------|--------|
| 2200    | 0     | 0    | 0      |
| 2700    | 0     | .14  | 2200   |
| 3200    | 70    | .15  | 2700   |
| 3700    | 145   | .16  | 3200   |
| 4200    | 225   | .17  | 3700   |
| INT_MAX | 53090 | .70  | 102200 |

Then we can just scan through the table:

```
for i in 0 .. len(table):
    if income <= limit[i]:
        tax = base[i] + rate[i] * (income - offset[i])
        break
```

It does look like the offset value is always the previous row's limit, so you could be more sophisticate about it.

| Limit   | Base  | Rate |
|---------|-------|------|
|    0    | 0     | 0    |
| 2200    | 0     | 0    |
| 2700    | 0     | .14  |
| 3200    | 70    | .15  |
| 3700    | 145   | .16  |
| 4200    | 225   | .17  |
| INT_MAX | 53090 | .70  |

```
for i in 1 .. len(table)
    if income <= limit[i]:
        tax = base[i] + rate[i] * (income - limit[i-1)
        break
```

# Problem #2

A *k-th* order linear recurrence with constant coefficients defines a series as:

$$ a\_n = c\_1*a\_{n-1} + c\_2*a\_{n-2} + ... + c\_k*a\_{n-k} + c\_{k+1}$$,

where \\(c\_1, ..., c\_{k+1}\\) are real numbers. Write a program that with input \\(k, a\_1, ..., a\_k, c\_1, ..., c\_{k+1} \\), and *m* produces the output \\(a_1\\) through \\(a_m\\). How difficult is that program compared to a program that evaluates one particular fifth-order recurrence, but does so without using arrays?

``` cpp
void recurrence(int K, int M, double* a, const double* c) noexcept {
    for (int n = K; n < M; ++n) {
        a[n] = c[K];
        for (int i = 0; i < K; ++i) {
            a[n] += a[n-i-1]*c[i];
        }
    }
}
```

Another solution (not really convinced it is better...):

``` cpp
using Real = double;

template <size_t K>
using Reals = std::array<Real, K>;

template <int K, int M>
Reals<M> recurrence(Reals<K> as, Reals<K+1> cs) noexcept
{
    Reals<M> output;
    memcpy(&output[0], &as[0], sizeof(as[0])*K);
    for (int n = K; n < M; ++n) {
        // Calculate the new value
        Real r = cs[K];
        for (int i = 0; i < K; ++i) {
            r += as[K-i-1]*cs[i];
        }
        // Propogate the values backward
        for (int i = 0; i < K-1; ++i) {
            as[i] = as[i+1];
        }
        as[K-1] = r;
        output[n] = r;
    }
    return output;
}
```

# Problem #3

Write a "banner" function that is given a capital letter as input and produces as output an array of character that graphically depicts that letter.

Would just make an array, and index that for each input.

# Problem #4

Write functions for the following date problems: given two dates, compute the number of days between them; given a date, return its day of the week; given a month and year, produce a calendar for the month as an array of characters.

To do this without the system utilities is actually a huge PITA. But you can use the `cal` command :)

# Problem #5

This problem deals with a small part of the problem of hyphenating English words. The following list of rules describes some legal hyphenations of words that end in the letter "c"
:

et-ic al-is-tic s-tic p-tic -lyt-ic ot-ic an-tic n-tic c-tic at-ic h-nic n-ic m-ic l-lib b-lic -clic l-ic h-ic f-ic d-ic -bic a-ic -mac i-ac

The rules must be applied in the above order; thus the hyphenations "eth-nic" (which is caught by the rule "h-nic") and "clinic" (which fails that test and falls through to "n-ic"). How would you represent such rules in a function that is given a word and must return suffix hyphenations?

# Problem #6

Build a "form-letter generator" that can prepare a customized document for each record in a database (this is often referred to as a "mail-merge" feature). Design small schemas and input files to test the correctness of your program.

Use jinja.

# Problem #7

Typical dictionaries allow one to look up the definition of a word, and Problem 2.1 describes a dictionary that allows one to look up the anagrams of a word. Design dictionaries for looking up the proper spelling of a word and for looking up the rhymes of a word. Discuss dictionaries for looking up an integer sequence (such as 1, 1, 2, 3, 5, 8, 13, 21, ...), a chemical sturcture, or the metrical structure of a song.

# Problem #8

[S. C. Johnson] Seven-segment devices provide an inexpensive display of the ten decimal digits: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9.
The seven segments are usually numbered as:

![Seven Segment Display figure](/img/seven_segment_display.png)

Write a program that displays a 16-bit positive integer in five seven-segment digits. the ouput is an rray of five bytes; bit *i* of byte *j* is one if and only if the *i-th* segment of digit *j* should be on.

I would again just have a pre-built array of 10 uint8_t where each bit set indicates if the line is on. Then I would do an algorithm like:

```
n = <input> base = 10000
while base > 0:
    digit = n // base
    print(digit_to_segments[digit])
    n = n % base
    base = base / 10
```
