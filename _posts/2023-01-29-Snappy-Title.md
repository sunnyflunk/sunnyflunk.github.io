---
title: "Snappy Title"
date: 2023-01-29
---

# Snappy Title

The [previous benchmarks](https://sunnyflunk.github.io/2023/01/15/x86-64-v3-Mixed-Bag-of-Performance.html) were intended
to highlight the difference compiling with `x86-64-v3` makes.

#### lz4

| Compress Kernel (--best) | x86-64 – Time | Power | x86-64-v2 – Time | Power | x86-64-v3 – Time | Power |
|:---------------------|:-------------:|:-----:|:----------------:|:-----:|:----------------:|:-----:|
| -O2 | 44.24 | 642.9 | 5.8% | 9.4% | 2.9% | 1.1% |
| -O3  | 1.4% | 0.3% | 13.2% | 11.5% | -0.5% | 4% |

| Compress Kernel (--best) | x86-64 – Time | Power || x86-64-v2 – Time | Power || x86-64-v3 – Time | Power |
|:---------------------|:-------------:|:-----:||:----------------:|:-----:||:----------------:|:-----:|
| -O2 | 44.24 | 642.9 || 5.8% | 9.4% || 2.9% | 1.1% |
| -O3  | 1.4% | 0.3% || 13.2% | 11.5% || -0.5% | 4% |
