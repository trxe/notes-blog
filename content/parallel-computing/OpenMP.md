---
layout: default
usemathjax: true
permalink: /parallel/openmp
---

# Structure of OpenMP Program

`omp_get_thread_num()`: Thread ID [*Master thread* has `id == 0`]

`#pragma omp parallel` : a **parallel** region is marked in a scope

## Work-sharing constructs

### Type 1: Loop iterations

```cpp
#pragma omp parallel shared() private()
{
    #pragma omp for schedule (static chunksize)
    for (/* */) {

    }
}
```

- $n$ iterations divided into **chunks** of size `chunksize`
- assignment to threads is static 

### Type 2: Sections

```cpp
#pragma omp parallel 
{
    #pragma omp sections
    {
        #pragma omp section
        {
            work1();
        }

        #pragma omp section
        {
            work2();
        }

        #pragma omp master
        {
            master();
        }
    }
}
```

- manually declaring the sections to be sent to different threads for execution
- `master` construct for only the master section

## Synchronization constructs

```cpp
#pragma omp parallel 
{
    #pragma omp critical
    {
        // critical section
    }

    #pragma omp atomic
    // atomic statement (the atomic operation on the storage location 
    // designated as x is a unit of work)
}
```