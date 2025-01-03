# Parallel and Grid Computing

## References

### About C++

* [cppreference](https://en.cppreference.com/w/): The best C++ online reference around.
* [isocpp](https://isocpp.org/): Some blog posts about C++, tips and reference papers.
* [Programming -- Principles and Practice Using C++ (3rd Edition)](https://www.stroustrup.com/programming.html): Book for C++ beginners.
* [A Tour of C++, third edition](https://www.stroustrup.com/tour3.html): Book if you already know basics of C++, it covers C++20.

### About Parallel Programming

* [E. A. Lee, “The Problem with Threads”](https://digitalassets.lib.berkeley.edu/techreports/ucb/text/EECS-2006-1.pdf): A seminal paper on why shared-state parallelism is hard.
* [The Art of Multiprocessor Programming](https://shop.elsevier.com/books/the-art-of-multiprocessor-programming/herlihy/978-0-12-415950-1): General introduction to parallelism.
* [P. E. McKenney, “Is Parallel Programming Hard, And, If So, What Can You Do About It?”](https://cdn.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html): Advanced textbook on parallelism.


## C++ Crash Course

### Cellular Automaton: Rule 184

1. `1-introduction-cpp/exercises/rule184.cpp`: Implement the [Rule184](https://en.wikipedia.org/wiki/Rule_184).
You write the simulation in a loop using two arrays of cells (the current array and the next array).
To see the evolution of the queue, pause the program 1 second between two iterations.
2. Decompose the program into several functions:
```cpp
print(cells);
simulate(steps, current, next);
simulate_step(current, next);
```
3. Add a functionality to compute the longest size of consecutive '1' at any iteration of the simulation (it represents the longest traffic jam).

### Modularity

See the files in `demo/modular` for setting up a multi-files project with headers (.hpp) and .cpp files.

### Data Structure: Vector

1. `1-introduction-cpp/exercises/int_vector.hpp`: Implement the necessary constructors, destructors and methods given in the file. For additionnal documentation see [std::vector](https://en.cppreference.com/w/cpp/container/vector), we try to follow it as close as possible.
2. `1-introduction-cpp/exercises/vector.hpp`: Update the previous example using a templated class.
3. Extend (2) with a move constructor.
4. [Optional⭐⭐] Extend (3) to allow the type `T` to lack a default constructor (e.g. `Vector<S> x; x.reserve(10);` with `struct S { S(int x) {} };` should work.
  See [placement new](https://en.cppreference.com/w/cpp/language/new) and [std::malloc](https://en.cppreference.com/w/cpp/memory/c/malloc).

### Lambda Function

1. `1-introduction-cpp/exercises/vector.hpp`: Implement a map function taking a vector and applying a function `f` to each of its component.
Suppose an integer vector `v` with the values `1,-5,6`, then `map(v, [](int x) { return x * 2; })` modifies `v` in-place and double each value.
2. `1-introduction-cpp/exercises/vector.hpp`: Implement a left-fold function (aka. "reduce") which takes a vector, an accumulator and a function `f`.
For instance, `fold_left(v, 0, [](int accu, int x) { return accu - x; })` returns the value `-2` (the difference of the integers in vector `v` from left to right).
[More information](https://en.wikipedia.org/wiki/Fold_(higher-order_function)).
3. [Optional⭐] Same as (2) but with `fold_right`, so it is from right to left. On the previous example, it also returns `-2`.
4. [Optional⭐] Rewrite the loops in `Rule184.cpp` using `map`. If it is not possible, introduce a new generic combinator similar to map to achieve your aim.

## C++ Parallelism with std::thread

### Map/Reduce

* `2-std-thread/demo/parallel_map.cpp`: Do some experiments with different sizes of array. When is it interesting to have 2 threads? Why?
* `2-std-thread/demo/parallel_map.cpp`: Use the function `more_work` and redo the experiments.
* [Optional⭐] `2-std-thread/demo/parallel_map.cpp`: Extend `parallel_map` to any number of threads.
* `2-std-thread/exercises/parallel_fold_left.cpp`: Implement `parallel_fold_left_commutative`  where 2 threads compute on half the array, wait for completion and then both results are merged. Note that the operation must be commutative otherwise you might not get the same result as in sequential computation.
* [Optional⭐] `2-std-thread/exercises/parallel_fold_left.cpp`: Extend `parallel_fold_left` to any number of threads.
* `2-std-thread/exercises/parallel_map_reduce.cpp`: Perform a map in parallel, followed by a fold. Find a (simple) way to reuse the threads from the map to compute the fold. Think about the pros and cons of your design, can we do better?

### Histogram

* `2-std-thread/exercises/histogram.cpp`: Given an array of size N taking value in the range [0,1000], implements a sequential algorithm `histogram_seq` which computes the frequency of each value and store the result in another array `histogram`.
* Parallelize the algorithm using thread and mutex when writing in the histogram array.
* [Optional⭐⭐] Lock-free algorithm: Use atomic to write values in the histogram, in order to avoid locks.

### Project: Parallelizing Rule 184

* Using the C++ threads, parallelize the simulation algorithm of Rule 184. Print only the final iteration and the size of the longest queue obtained.
* The goal is to perform the most iterations in 1 minute for different values of the array.
* Plot your results using Python or any tool to see the impact of 1, 2, ..., N threads on the efficiency according to the size of the array.
* Discuss your results and design in a small PDF report.
* Send the code and report in ".zip" before the next course begins (4th October 13h59) by email with the subject "[MHPC][PGC] Project".
* You can discuss your design and your results on Discord or orally, but please don't share your code.
* This is a solo project.

## Synchronization Problems

### [Optional⭐] Dining Philosophers Problem

* Implement the dining philosopher for 2 philosophers by simply numbering the forks (you can read [this code](https://github.com/ffelten/ocaml-snippets/blob/main/shared_memory_II/bin/philosophers_lock.ml)).
* Generalize your solutions for N philosophers.
* Modify the algorithm to use coarse-grained locking.

### Producer-consumer Problem

* Implement a thread-safe version of `vector.cpp` (add a mutex as a private member, and use a `lock_guard` to guard the operations).
* Suppose you have 1 thread consuming data from the vector (`pop_back`), and another thread producing data (`push_back`). Create two threads that produces and consumes integers. Is the class `vector` sufficient for this situation?
* Create a `buffer` class wrapping a `vector` to solve with two methods `produce` and `consume`. Use a [semaphore](https://en.cppreference.com/w/cpp/thread/counting_semaphore) to synchronize in case the vector is empty.
* With many consumers and producers, can this implementation be a bottleneck when parallelizing?
* [Optional⭐] Add an upper limit on the size of the vector, so a producer will wait when the buffer is full.
* [Optional⭐⭐] View the `vector` as a queue instead of a stack, so the first element produced will be consumed first (and not last like now).

### [Optional⭐⭐] Reader-Writer Problem

Suppose you have a shared resources (e.g. a web page) among many readers (e.g. the people browsing the web) and writers (e.g. the developers updating the page).

* Implement a class with a method `read` and a method `write`. For the purpose of the exercise, the resource can simply be an integer.
* Many readers can concurrently access the resource, but only one writer can modify the resource at a time (therefore no reader must be currently reading the resource).
* Does your implementation take into account the problem of many readers, such that there is always at least a reader accessing the resource? What can you do about it?

## Task parallelism

### N-Queens Problem (`4-task-parallelism/nqueens.cpp`)

1. Execute the sequential code for N from 10 to 16 and note the number of explored nodes, the number of solutions and the execution time for each instance.
2. Parallelize the sequential code following a single-pool multi-thread approach.

*Hints:*
- *The number of threads should be read from the command-line;*
- *Threads (`std::thread`) should be stored in a `std::vector`;*
- *Accesses to the pool must be protected with `std::mutex`.*
3. Re-execute question 1. considering the parallel version implemented in 2. and varying the number of threads. What do you observe? Why?
4. Extend the parallel version implemented in 2. using multiple pools (one pool per thread).

*Hints:*
- *The victim selection policy will be random (uniform);*
- *The work stealing granularity will be "steal-half".*
5. Re-execute question 1. considering the parallel version implemented in 4. and varying the number of threads. Compare with the results obtained in 3. What do you observe? Why?

## OpenMP

### Map-reduce

* `5-openmp/exercises/scalar_product.cpp`: Using OpenMP pragmas, parallelize the function `scalar_product`.
* `5-openmp/exercises/reduction_min.cpp`: Write a parallel reduction function computing the minimum value.
* `5-openmp/exercises/filter_leq.cpp`: Write a parallel function `filter_leq(v, k)` taking a vector `v` of integers and an integer `k` which returns a new array with the sum of all elements less or equal to `k` in `v`.

### [Optional⭐] N-Queens

* `5-openmp/exercises/nqueens.cpp`: Parallelize the N-queens code seen in the previous course using OpenMP tasks. Hint: check `5-openmp/demo/tasks_fib.cpp`.

### Project Big Graph

The project description is [available here](https://github.com/ptal/parallel-and-grid-computing-uni.lu/tree/main/5-openmp/project/README.md).

## Partitioned Global Address Space (Chapel)

### Mandelbrot set computation (`6-pgas-chapel/mandelbrot_seq.chpl`)

1. Install Chapel on your system using the provided script (it can take some minutes). Command:
```
cd 6-pgas-chapel/
source chapel_install.sh
```
2. Read the introductory file [available here](https://github.com/ptal/parallel-and-grid-computing-uni.lu/tree/main/6-pgas-chapel/exercises/introduction.pdf).
3. Implement a parallel approach where each thread computes a block made of
contiguous lines of the global image. What do you observe in terms of speed-up compared to the sequential version?
4. The goal is to improve the load balancing between threads. Implement a variant of the previous algorithm where each processor computes a block made of alternated lines. More precisely, we will assume that the *k*-th thread (*0 <= k < n*) takes in charge the lines indexed by *k + i n*.

*Hints:*
- *The number of threads `n` can be set in command-line using `CHPL_RT_NUM_THREADS_PER_LOCALE=n`;*
- *Compile using the `--fast` optimization flag: `chpl --fast foo.chpl`;*
- *You can verify that two images are similar using the output of `diff image1 image2`.*

### [Optional⭐] N-Queens

* `6-pgas-chapel/exercises/nqueens.chpl`: Parallelize the N-queens code seen in a previous course using Chapel.

## MPI

* `7-mpi/exercises/ping_pong.cpp`: Given `N` processes, we pair processes `(0,1), (2,3), ...` such that `0` sends "ping" to `1` which replies with "pong". This is repeated `num_ping_pong` times as specified by a program's argument.
* `7-mpi/exercises/ping_pong_ring.cpp`: Given `N` processes organized in a ring, a process send "ping" to its right, receive "ping" from its left, send "pong" to its left and receive "pong" from its right. This is repeated `num_ping_pong` times as specified by a program's argument.
* `7-mpi/exercises/stop_when.cpp`: Given 3 processes, process 1 and 2 waits for a random number of seconds, and afterwards they send a message to the process 0 (can use `MPI_ANY_SOURCE`). Once process 0 receives the message, it broadcasts a termination message to everybody, and all processes *immediately* stop.
* `7-mpi/exercises/sum.cpp`: Suppose we have `N` processes. The process 0 generates a random array `arr`. Using `MPI_Scatter`, the array is distributed among the processes. They all compute the local sum of the array. Using `MPI_Reduce` we retrieve the global sum and print it in the process 0.
* Read [this paper](http://www1.cs.columbia.edu/~sedwards/papers/kahn1974semantics.pdf) on Kahn network.

### Project: Maze Creation and Testing

In this project, you will develop a parallel algorithm which creates a maze and test how many paths can enter and exit the maze.

* Use randomized depth-first search to create a maze, or with any technique of your choice (see [here](https://en.wikipedia.org/wiki/Maze_generation_algorithm#Randomized_depth-first_search)).
* Given the beginning and exit of the labyrinth, design three different algorithms with MPI (different communication and/or data-splitting strategies) finding *all different paths* in the labyrinth.
* Compare the paths found (if the maze is small enough) or the number of paths found (otherwise) among the algorithms. You must obtain the same results, otherwise an algorithm is wrong.
* Write a short report explaining the intuitions behind the three algorithms, and evaluate those algorithms on a set of generated maze (from small to large).
* Give hypothesis on why an algorithm is working better than another.
* The goal is to practice with MPI and different topologies/strategies. It is OK if your algorithm is slower than the others as long as you can explain why.
* For students in the Master in HPC: it is mandatory to run some experiments on the HPC with the number of nodes >= 2.
* The code you produce must be your own and *you cannot copy* an existing MPI algorithm from internet.
* You can work in team of 3, each member must design and implement a different algorithm, and you must explicitly say in the report who wrote which algorithm.
* Send the project before the exam at pierre.talbot@uni.lu with the subject "[MHPC][PGC] Project MPI".
* *Optional*: Use the [Boost.MPI library](https://www.boost.org/doc/libs/1_86_0/doc/html/mpi.html) to create your application.
