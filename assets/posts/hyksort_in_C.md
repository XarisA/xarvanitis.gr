# Hyksort Algorithm Explained

Below is the algorithmic explanation of Hyksort and the techniques I used to implement it. In addition to the Hyksort algorithm and the complementary algorithms it uses, some helper functions have been created in order to execute, and control the operations of the algorithm. In the main program it is possible to measure the performance of the algorithm, using different functions for various phases. 

>My implementation has been done in C language using mpich as MPI library with mpicc compiler. 

[View on Github](https://github.com/XarisA/HykSortInC)

## Abstract
Hyksort is a parallel distributed memory classification algorithm, which is a variant of samplesort, but with two substantial improvements. The first lies in the way the pivots are selected, and the second in the way the communication between processing elements is done to redistribute the elements.
To be more specific, the algorithm is based on the hypercube quicksort, but instead of a binary recursion, we perform a k-way recursion in which the pivots are selected accurately with an iterative parallel select algorithm.

>The advantages of HykSort are lower communication costs, better load balancing, and avoidance of O(p)-collective communication primitives. 

## 1. Introduction

In my approach, I designed a program that can run the  hyksort algorithm and some variations and measure the execution times. 

In my application ***Hyksort in C***, the are three different variations of the algorithm and the main purpose is to measure and analyse execution times for large ammounts of data and finally calculate the algorithmic complexity and draw conclusions.

1. In the first variation (Hyksort v1), I implemented the SampleSplitters function and I did redistributed the data with the [MPI_Alltoall](https://www.mpich.org/static/docs/v3.2/www3/MPI_Alltoall.html) function.

2. In the second vatiation (Hyksort v2), the SampleSplitters was replaced with ParallelSelect

3. In the third variation (Hyksort), eventually the MPI_Alltoall was replaced Alltoall_kway which is actually the algorithm described in the paper *HykSort: A New
Variant of Hypercube Quicksort on Distributed Memory Architectures, 2013, Hari Sundar , Dhairya Malhotra , George Biros.* [1]


The default execution of the program is for the final version (Hyksort), however, the user has the ability to switch between the other variations.


### Hyksort (pseudocode)

In high level the pseudocode of the algorithm can be described in the following table. As you can se in the steps 4 and 8 we can choose a variation.


| Steps  &nbsp; &nbsp; | Hyksort  &nbsp; &nbsp;  &nbsp; &nbsp;| Comments  &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|
| --- | -------------------- | -------------------------------------- |
| **In**                | Ar, n, p, rank, comm  | Ar: local table to sort, n: number of elements Ar, p: the number of processes, rank: the process  that calls Hyksort, comm: the communicator                              |
| **Out**               | GlB                   | GlB: Sorted table with the elements of the Ar tables of all processes. |
| 1:                    | N ← n \* p            | Array B count         |
| 2:                    | Ar **← localsort**(Ar,n)               | Classification of table Ar |
| 3:                    | k←p                   | I set the number of separations I want to use in ParallelSelect |
| 4:                    | Spl←**ParallelSelect**(Ar,n,p,rank,comm) | Find (k-1) separators by using the ParallelSelect function; an alternative implementation would be with the SampleSplitters function              |
| **or**                | Spl←**SampleSplitters**(Ar,n,p,rank) |
| 5:                    | Buckets←**Rank**(Ar,Spl) | Segment ar elements based on Spl separators |
| 6:                    | **while** (p\>1)      |                       |
| 7:                    | \[BucketBuffer,comm\]←**SendRecieve\_kway**(Buckets,rank,comm) | Redistribute Bucket items between processors.           |
| 8:                    | **end while**         |                       |
| **or**                | BucketBuffer←**MPI\_A lltoall**(Buckets,n+1) | |
| 9:                    | B←{BucketBuffer\[**i**\]**\|** i \> 0 } | Create a table B that contains all non-zero bucketbuffer items    | |
| 10:                   | Bsize ← {+1 ¥ j ∈ B :j \> 0} | Create a local Bsize variable that contains the number of non-zero elements in B.                 |
| 11:                   | counts ← **MPI\_Gather**(Bsize,rank =0)             | Create a table ofcounts in process 0 that contains the  Bsize of each process (practically the sizeof table B of each process)              |
| 12:                   | displs←{0 for j=0 ∪dipls\[i-1\] +counts\[i-1\] for j\>0 } ¥ j∈0,\..., p-1   | Create a table with the offset; it  practically contains the location where  the message will enter each procedure. |
| 13:                   | GlB ← **MPI\_Gatherv**(B, counts,displs) | Create a GlB table in process 0 that contains all the sub-arrays B from all other processes.      |



In the following figure we can see a graphic representation of the basic idea in hyksort algorithm as it was published in [1].

![Hyksort high level representation](assets/images/hyksort_key.jpg)


## 2. Methodology

### main.c (pseudocode)

The pseudocode for the main program can be found below.


| main function (pseudocode)        | Comments                          |
| --- | --- |
| **Input:** N , p                  | N: number of table items, p: number of processes            |
| **Output:** GlB                   | GlB: classified table             |
| 1: A ← **create\_array**(N,a,b)   | Create an N-sized A array with random elements in (a,b\]\* |
| 2: n ← N/p                        | n: number of elements in local table A                           |
| 3: **if** (p = 1)                 | Check if I have only one process  |
| 4: t ← **count\_time** , B ← **quicksort**(A,n)     | Measure time and sort the table with quicksort                    |
| 5: **else if** (p \>1)            | If I have more than one process   |
| 6: Ar ←**MPI\_Scatter**(A,n)      | Disperse n elements in Table A in each process                      |
| 7: t ← **count\_time** , GlB←**Hyksort**(Ar ,n, p, rank,comm)| Measure time and sort the table with Hyksort |
| 8: **return** B                   | Return of the sorted table        |
| 9: **print** t                    | Print time on the **stdout** screen    |

*\*a and b have default values of 1 and 10\^5 accordingly.*

### Partition Example

Bellow is described the partitioning, step by step, for an array of 64 unclassified elements, when the algorithm can use 4 processes.

1. The following one dimensional array in given as input in the processor p0.
(For ease of display the data are shown in multiple lines)

![Input Array A in p0](assets/images/hyksort_1.png)

2. Each processor takes n=16 items in table A and saves them in its local table Ar.

![Array Ar](assets/images/hyksort_2.png)

3. The Ar table is sorted locally in each processor using the quicksort algorithm. This step is not necessary for the algorithm, but it helps in the next steps and it does not increase significantly the algorihtmic complexity. 

![Sorted Ar](assets/images/hyksort_3.png)

4. Pivots are selected either with the SampleSplitters  or with the ParallelSelect algorithm. Both algorithms have been implemented in this application. 
The SampleSplitters algorithm  is followed by a simple sorting algorithm. I used bitonicsort algorithm for this local sort.

![Splitters r](assets/images/hyksort_4.png)

5. The next step is the creation of a local histogram in each processor where the elements of *Ar* are counted for each splitter ranges [a,b).

![Local histogram rAr](assets/images/hyksort_5.png)

6. In the next step the local arrays are gathered in the process p0 and their elements are summed in each index position. The resulting array is scattered to every processor.

![Global histogram GlrAr](assets/images/hyksort_6.png)

7. From this array the k-1 separators are selected locally in each processor, followed by a reduction, merging all the elements into a scalar result.

![Global Splitters Spl](assets/images/hyksort_7.png)

8. Based on these separators and with the help of the Rank function, the elements of the original table are divided locally into k buckets. 
For ease of display the data are shown in two parts even though it is a one-dimensional array with size N+k = 68 elements.
The first element for every n=16 elements represents the size of the bucket.

![Buckets](assets/images/hyksort_8.png)
![Bucket size](assets/images/hyksort_8b.png)

9. This step redistributes the elements between the processors. Either with the mpi_Alltoall function  or with the alltoallv_kway algorithm.

![Buckets redistributed](assets/images/hyksort_9.png)

10. Next we remove the zero elements and we sort each array locally.

![Buckets Sorted](assets/images/hyksort_10.png)

11. The final step is to collect the arrays from each process into a single processor. To do so, we create two helper arrays so that we can use the MPI_Gatherv function  to collect the elements in processor 0. The output (B array) is one-dimensional and has size N=64; 
(Ouput is presented in multiple lines for ease of display.)

![helper arrays](assets/images/hyksort_11.png)
![Output](assets/images/hyksort_11b.png)


As you can see the unsorted Input it is now sorted

![Input](assets/images/Input.png)
![Output](assets/images/Output.png)


## 3. Analysis

This section presents some measurements made at the application runtime for different sizes of the input array. 

![Speedup Diagram](assets/images/Figure1.png)
![Time Improvement Chart](assets/images/Figure2.png)
![Data Table](assets/images/Table.png)

## 4. Conclusions

- From the time improvement chart (Figure 2) we can see a slight increase as we proceed from the 1 core to the 8 and then a sharp polynomial increase.
- This time increase (after the 8th processor), improves when the program is executed using integer multiples of physical processors (16 or 32 processes).
- The algorithm was also executed using the function MPI_Alltoall versus Alltoall_kway and corresponding results were observed.
- The mathematical improvement of the Hyksort algorithm versus Samplesort is obvious, but the practical improvement cannot be ascertained by conventional machines with low processing power. Moreover, this can be seen from graph 1 of the paper [1].


## References

1. Hari Sundar, Dhairya Malhotra, George Biros, [HykSort: a new variant of hypercube quicksort on distributed memory architectures](http://dx.doi.org/10.1145/2464996.2465442), Proceedings of the 27th international ACM conference on international conference on supercomputing (**ICS13**), 2013.

2. [Utah Sorting Librrary](https://github.com/hsundar/usort)

