---
title: "x86-64-v3: Mixed Bag of Performance"
date: 2023-01-16
---

# x86-64-v3: Mixed Bag of Performance

The pursuit of performance has long been sought after by advanced users, from custom compiling select packages to
building your whole system from source. The inclusion of new variations to the default `x86_64` in the psABI has seen
this extended to the distribution level, where some distributions are looking (or in the process) to include higher
levels such as `x86-64-v2` or `x86-64-v3`. The belief in the performance improvements have even resulted in new
distributions being created to maximize performance.

The consensus on the value is often mixed, where we see benchmarks showing wild improvements that it can bring, but very
little on the drawbacks that come with it. Once we start shifting to building an entire distribution's packages with
`x86_64-v3`, then the performance of the every day package starts to matter, not just the benefit we see from encoding
audio or video files.

## A Lot of Hype and Marketing

One of the newer entries into the performance arena is CachyOS, which rebuilds some of the Arch Linux repos with
`x86-64-v3` (plus some further modifications to performance sensitive packages). There are other modifications as well
(which you can read on their [website](https://cachyos.org/)), but for this we are only interested in the impact of
providing these packages **without** any of the other changes.

On the [wiki](https://wiki.cachyos.org/en/home/features) it states that "if x86-64-v3 is detected it will automatically
use the optimized packages, which yields more than 10% performance improvement." This is a very powerful statement...but
is it true? In fairness, they are telling new users on discord that not all packages will benefit from using
`x86-64-v3`, but adding caveats doesn't make it sound nearly as good. Do package optimizations bring better performance
across the board?

## What Are we Measuring?

When interpreting test results, it is paramount to understand how the test is performed. If you do not understand what
the test is doing, then you do not understand the results! To target the impact of optimized packages we want to keep
as much as possible the same between both runs.

The host machine is running Arch Linux, while testing for both sets of packages is performed in an `arch-chroot` to
ensure the same kernel, environment and settings. The difference comes when switching the repos to install the
`x86-64-v3` packages from CachyOS and then updating and running the tests again. Therefore we are testing the
performance of the packaged binary packages from each distribution while keeping other factors constant.

Inspecting the CachyOS packages, it appears they are setting `-march=x86-64-v3 -mpclmul -O3` vs `-march=x86-64 -O2`
(other flags being the upstream Arch Linux defaults) in Arch Linux, so we are testing wider optimizations than just
switching to `x86-64-v3`. There are also some instances of `-mtune=skylake` which may have been set at one stage.
Increasing the `-march` level and optimization levels are usually touted as the easiest ways to improve performance.

### Methodology

The tests themselves are fairly basic and accessible via [benchmarking-tools](https://github.com/sunnyflunk/benchmarking-tools).
They run in RAM to remove the impact of disk latency and allow them to run as fast as possible (i.e. you may not even be
able to observe the benefits if your hard drive is really slow).

There are two variations on the testing, measuring the power consumption using RAPL and my machine, which is a
NUC8i5BEK, includes the full system energy use. I also re-ran the tests under severe power threshold limitations via
RAPL (more than halving the peak and average W targets), but as there was only a couple of tests that even hit these
limits I have only included the one instance where it made a difference.

But all that really matters is the results...less talk and more numbers!

## Results

All benchmarks are measured with time (seconds except pybench which is ms), so lower is better (or minus percentages
mean the CachyOS packages are faster or use less power). Power is measured in watts. The first two columns are the Arch
actual results and the percentages are the difference for the CachyOS results.

#### bzip2

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|       Compress Kernel (-3) | 47.01 | 689.8 | -2.5% | -1.5% |
|       Compress Kernel (-9) | 49.80 | 736.8 | -1.5% | -1.4% |
|          Decompress Kernel | 15.31 | 225.5 |  7.1% |  6.9% |

`bzip2` shows the conundrum of optimization. You can improve the performance of one use case to the detriment of
another. Here we see compression times decreasing slightly, but a large regression to decompression times!

Always make sure you test both!

#### flac

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|                     Decode | 0.71 | 10.5 | -10.1% | -12.1% |
|                Encode (-3) | 0.90 | 13.3 | -15.0% | -12.4% |
|                Encode (-8) | 2.95 | 49.1 | -20.2% | -23.4% |

To no surprise `flac` benefits a lot from optimization. The source code already includes AVX2 runtime functions to
improve the performance without requiring it to be enabled via the `-march` flag. Quite an improvement given it will
already use AVX2 on the Arch build.

#### gawk

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|                       Gawk | 2.88 | 42.9 | -2.3% | -0.7% |

Not a commonly thought about package for benchmarking and only a slim improvement.

#### gzip

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|       Compress Kernel (-3) | 10.33 | 152.1 | -9.5% | -10.0% |
|       Compress Kernel (-9) | 35.03 | 501.6 | -2.9% | -5.8% |
|          Decompress Kernel |  3.49 |  50.2 | -0.7% | -1.6% |

`gzip` also enjoys great benefits at the `-3` compression level, but the benefit is reduced at higher levels and for
decompression.

#### lz4

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|   Compress Kernel (--best) | 44.60 | 642.7 |  1.6% | 10.5% |
|       Compress Kernel (-5) |  9.70 | 141.7 |  2.9% | 10.2% |
|          Decompress Kernel |  0.73 |  12.1 | -5.4% | -2.9% |

Here we see some more regressions for compressing with `lz4`. We also see quite a jump in power consumed, likely due to
AVX2, to be much higher than the percentage change in performance.

#### python

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|                    Pybench | 1472 | 290 | 3% | 3.6% |

My understanding is that the CachyOS package also includes BOLT optimization on top of `x86-64-v3` (which is said by
upstream to improve performance by a % or two on such a benchmark). Even with additional optimizations it still
lagged behind performance wise.

#### r

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|             R-benchmark-25 | 39.41 | 2992.5 | 0% | 0.4% |

CachyOS doesn't rebuild the `r` package, but it does rebuild blas and lapack. According to `perf`, the test spends 86%
of the time in `blas`, yet we still see no benefit. But, the proper way to actually improve performance here is to use
`openblas` instead.

#### vorbis

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|                     Decode | 2.22 |  33.6 | -10.7% |  -8.4% |
|            Encode (-b 128) | 6.48 |  99.2 | -20.8% | -19.5% |
|             Encode (-q 10) | 8.48 | 129.4 | -14.7% | -12.8% |

Another great showing for optimization and what makes people excited by having them available across the entire repo.

#### xz

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|   Compress Kernel (-3 -T1) | 75.03 | 1149.5 | 0.6% | 0.7% |
|   Compress Kernel (-9 -T4) | 95.63 | 2638.2 | 1.2% | 8.9% |
|          Decompress Kernel |  5.76 |   88.3 | 0.6% | 1.3% |

A slight regression for `xz`, but a huge increase in power when using four threads! This is the only benchmark where
power restrictions had a notable impact on the result. The optimized packages were now 4% slower (only 1.2% slower
without power restrictions). Of note, despite taking about 10% longer with power restrictions, both runs required less
power overall to complete the task.

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|   Compress Kernel (-9 -T4) - Low Power | 104.97 | 2529 | 4% | 4.7% |

#### zstd

|                       | Arch – Time | Power | CachyOS – Time | Power |
|:----------------------|:-----------:|:-----:|:--------------:|:-----:|
|  Compress Kernel (-19 -T4) | 77.64 | 2443.8 |  -4.4% | -5.7% |
|   Compress Kernel (-8 -T1) | 11.09 |  181.8 |  -5.9% | -2.5% |
|          Decompress Kernel |  0.99 |   16.4 | -15.7% | -0.2% |

Here we see another interesting case, where the time taken for decompression decreases 15.7%, but no real improvement
in the power consumption.

Again we see why measuring power consumption is an integral part of benchmarking and we are seeing packages draw much
higher than expected power when built with `x86-64-v3`.

## A Mixed Bag, But a Win Overall

Optimized packages can provide considerable advantages over their generic `x86-64` counterparts for many packages. But
where software doesn't see much benefit from these newer instructions, we are often left with worse performance and
higher power consumption. This makes it a complex question over whether it's worthwhile to build every package with
greater optimizations.

![Results](https://raw.githubusercontent.com/sunnyflunk/sunnyflunk.github.io/main/_posts/img/20230116.png "Results")

This graph shows that overall the winners were bigger than the regressions. However, this post was intended to be more
about `x86-64-v3`, but some quick tests (which requires further analysis) suggest that CachyOS using `-O3` is what's
actually responsible for some of the larger gains rather than `x86-64-v3`.

When rolling out optimizations across an entire repo, one needs to take care to minimize the negatives while enjoying
the benefits. This requires substantial testing and breaking free from the belief that increasing the CPU instruction
set and optimization level always leads to greater performance. It can, but it's not universal and shouldn't be used as
a guide of what to expect across all packages. Over-optimization is generally not a good idea unless the compiler
combination and flags have been tested across a vast array of packages and proven to be a improvement with few (and only
small) regressions.
