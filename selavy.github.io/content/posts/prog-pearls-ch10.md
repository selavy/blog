+++
title = "Programming Pearls: Chapter 10"
date = 2019-02-22T18:30:05-08:00
draft = false
+++

Problems from Column #9: Squeezing Space

# Problem #1

In the late 1970's Stu Feldman build a Fortran 77 compiler that barely fit in a 64-kilobyte code space. To save space he had packed some integers in critical records into four-bit fields. When he removed the packing and stored the fields in eight bits, he found that although the data space increased by a few hundred bytes, the overall size of the program went down by several thousand bytes. Why?

Likely his code became simpler so the code section decreased in size.

# Problem #2

How would you write a program to build the sparse-matrix data structure described in Section 10.2? Can you find other simple but space-efficient data structures for this task?

# Problem #3

How much total disk space does your system have? How much is currently available? How much RAM? How much RAM is typically available? Can you measure the sizes of the various caches on your system?

commands to use:
```
free
df
cat /proc/cpuinfo
cat /proc/meminfo
```

# Problem #4

Study data in non-computer applications such as almanacs and other reference books for examples of squeezing space.

# Problem #5

In the early days of programming, Fred Brooks faced yet anothe problem of representing a large table on a small computer (beyond that in Section 10.1). He couldn't store the entire table in an array because there was room for only a few bits for each table entry (actually, there was one decimal digit available for each entry -- I said that it was in the early days!). His second approach was to use numerical analysis to fit a function through the table. That resulted in a function that was quite close to the true table (no entry was more than a couple of units off the true entry) and required an unnoticeably small amount of memory, but legal constraints meant that the approximation wasn't good enough. How could Brooks get the required accuracy in the limited space?

Use both approaches: use the function, then keep a table of the differences to get the correct result.

# Problem #6

The discussion of Data Compression in Section 10.3 mentioned decoding 10a + b with / and % operations. Discuss the time and space tradeoffs involved in replacing those operations by logical operations or table lookups.

# Problem #7

In a common type of profiler, the value of the program counter is sampled on a regular basis. Design a data structure for storing those values that is efficient in time and space and also provides useful output.

# Problem #8

Obvious data representations allocate eight bytes for a date (MMDDYYYY), nine bytes for social security number (DDD-DD-DDDD), and 25 bytes for a name (14 for last, 10 for first, and 1 for middle initial). If space is critical, how far can you reduce those requirements?

# Problem #9

Compress an online dictionary of English to be as small as possible. When counting space, measure both the data file and the program that interprets the data.

# Problem #10

Raw sound files (such as .wav) can be compressed to .mp3 files; raw image files (such as .bmp) can be compressed to .gif or .jpg files; raw motion picture files (such as .avi) can be compressed to .mpg files. Experiment with these file formats to estimate their compression effectiveness. How effective are these special purpose compression formats when compared to general-purpose schemes (such as gzip)?

# Problem #11

A reader observes, "With modern programs, it's often not the code that you write, but the code that you use that's large." Study your programs to see how large they are after linking. How can you reduce that space?
