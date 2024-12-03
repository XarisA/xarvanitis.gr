# Hyksort Algorithm in C++: A K-Way Recursion Approach to Optimized Sorting on Distributed Memory Systems

HykSort Algorithm Implementation with ParallelSelect algorithm for finding splitters and AlltoAll_kway for all to all communication between processors, having as a result reduced complexity to O(N log p / log k).

A big part of the logic in this code was found in this project `2` and was rewritten in c++. 
I added some helper functions and distinguished the HykSort algorithm from `2` so that it can run independenly.

*[View On GitHub](https://github.com/XarisA/HykSort)*


## Abstract

>In this paper, we present HykSort, an optimized comparison sort for distributed memory architectures that attains more than 2× improvement over bitonic sort and samplesort. The algorithm is based on the hypercube quicksort, but instead of a binary recursion, we perform a k-way recursion in which the pivots are selected accurately with an iterative parallel select algorithm. The single-node sort is performed using a vectorized and multithreaded merge sort. The advantages of HykSort are lower communication costs, better load balancing, and avoidance of O(p)-collective communication primitives. We also present a staged communication samplesort, which is more robust than the original samplesort for large core counts. We conduct an experimental study in which we compare hypercube sort, bitonic sort, the original samplesort, the staged samplesort, and HykSort. We report weak and strong scaling results and study the effect of the grain size. It turns out that no single algorithm performs best and a hybridization strategy is necessary. As a highlight of our study, on our largest experiment on 262,144 AMD cores of the CRAY XK7 "Titan" platform at the Oak Ridge National Laboratory we sorted 8 trillion 32-bit integer keys in 37 seconds achieving 0.9TB/s effective throughput.
>![Hyksort](https://dl.acm.org/cms/asset/9ff292a3-57c4-4138-b968-08c6764e66ae/2464996.2465442.key.jpg)


**References**

1. Hari Sundar, Dhairya Malhotra, George Biros, [HykSort: a new variant of hypercube quicksort on distributed memory architectures](http://dx.doi.org/10.1145/2464996.2465442), Proceedings of the 27th international ACM conference on international conference on supercomputing (**ICS13**), 2013. 

2. [Utah Sorting Librrary](https://github.com/hsundar/usort)

3. [Distributed Bitonic Sort using MPI](https://github.com/steremma/Bitonic)
