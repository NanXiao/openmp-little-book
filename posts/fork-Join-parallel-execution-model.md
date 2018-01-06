# Fork-Join parallel execution model
`OpenMP` uses Fork-Join parallel execution model. Take the following code as an example:  

	#include <stdio.h>
	#include <omp.h>
	
	int main(void)
	{
		printf("Before: total thread number is %d\n", omp_get_num_threads());
		
		#pragma omp parallel
		{
			printf("Thread id is %d\n", omp_get_thread_num());
		}
		
		printf("After: total thread number is %d\n", omp_get_num_threads());
		
		return 0;
	}

Build and run it on `2-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Before: total thread number is 1
	Thread id is 0
	Thread id is 1
	After: total thread number is 1

Let's analyze the code step by step:  

(1) After starting the program, there is only main thread in progress, and it is called **master thread**. `omp_get_num_threads()` is used to get current thread number. So the first output line is:  

	Before: total thread number is 1

(2) When **master thread** encounters the following **construct**:  

	#pragma omp parallel
	{
		printf("Thread id is %d\n", omp_get_thread_num());
	}
It will spawn a group of **child thread** (in our case, just `1` **child thread**) to run the code in **structured block** concurrently, so **master thread** is **parent thread** simultaneously. The **parent thread** and group of **child thread** are called **team** together. Every thread will get a thread ID, which can be obtained from `omp_get_thread_num()` function. **Master thread**'s ID is `0`, other threads get `ID` from `1`, `2`, .... So the following outputs from threads make sense:  

	Thread id is 0
	Thread id is 1

(3) After finish executing **structured block**, the progress transit from parallel to sequential running, and that's the meaning of "Fork-Join", so the last output shows only **master thread** is active here:  

	After: total thread number is 1
P.S., although the above log shows there is only one thread alive, but since most `OpenMP` implementation use thread pool to get performance gain, so the child thread is in idle state, not exit. Use `gdb` can verify it:  

(1) Build program with debug information:  

	# gcc -g -fopenmp parallel.c
(2) Use `gdb` to debug program and set breakpoint in "return 0;":  

	# gdb a.out
	......
	(gdb) b parallel.c:15
	Breakpoint 1 at 0x7f9: file parallel.c, line 15.
	(gdb) r

(3) After the breakpoint is hit, check the thread number:  

	(gdb) i threads
	  Id   Target Id         Frame
	* 1    Thread 0x7ffff7febc00 (LWP 24412) "a.out" main () at parallel.c:15
	  2    Thread 0x7ffff73cf700 (LWP 24416) "a.out" do_spin (val=8, addr=0x5555557566d4)
	    at /build/gcc/src/gcc/libgomp/config/linux/wait.h:56
We can see there are `2` threads.