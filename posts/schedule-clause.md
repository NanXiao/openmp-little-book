# Schedule clause

The `schedule` clause is defined as following:  

	schedule([modifier [, modifier]:]kind[, chunk_size])
It is used to determine how the iterations in loop constructs are distributed in threads. Check the following code:  

	#include <stdio.h>
	#include <unistd.h>
	#include <omp.h>
	
	void thread_fun(int seconds)
	{
		sleep(seconds);
		printf("Thread %d sleeps %d seconds\n", omp_get_thread_num(), seconds);
	}
		
	int main(void)
	{		
		#pragma omp parallel for
		for (int i = 1; i <= 8; i++)
		{
			thread_fun(i);
		}
		
		return 0;
	} 

Build and run it on `2-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 sleeps 1 seconds
	Thread 0 sleeps 2 seconds
	Thread 1 sleeps 5 seconds
	Thread 0 sleeps 3 seconds
	Thread 0 sleeps 4 seconds
	Thread 1 sleeps 6 seconds
	Thread 1 sleeps 7 seconds
	Thread 1 sleeps 8 seconds

We can see the first `4` iterations are executed in the first thread while the last `4` iterations are executed in the second thread. Modify the `loop` directive in above code:  

	#pragma omp parallel for schedule(static)
	for (int i = 1; i <= 8; i++)
	{
		......
	}
Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 sleeps 1 seconds
	Thread 0 sleeps 2 seconds
	Thread 1 sleeps 5 seconds
	Thread 0 sleeps 3 seconds
	Thread 0 sleeps 4 seconds
	Thread 1 sleeps 6 seconds
	Thread 1 sleeps 7 seconds
	Thread 1 sleeps 8 seconds
The same output as no `schedule` clause. If there is no `schedule` clause, the program will use default schedule policy which has the same behavior as `schedule(static)` in this case.  

Now we modify the `schedule` clause from `static`:  

	#pragma omp parallel for schedule(static)
	for (int i = 1; i <= 8; i++)
	{
		......
	}
to `dynamic`:  

	#pragma omp parallel for schedule(dynamic)
	for (int i = 1; i <= 8; i++)
	{
		......
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 sleeps 1 seconds
	Thread 1 sleeps 2 seconds
	Thread 0 sleeps 3 seconds
	Thread 1 sleeps 4 seconds
	Thread 0 sleeps 5 seconds
	Thread 1 sleeps 6 seconds
	Thread 0 sleeps 7 seconds
	Thread 1 sleeps 8 seconds
We can see the distribution policy is different from `static` case: The first thread executes the first iteration and the second thread runs the second iteration. When a thread finished its execution, it would grab the next iteration. Because the distribution is determined in runtime, so synchronization is needed.  

There is a special case of `schedule(dynamic)`: `schedule(guided)` which grabs iterations dynamically and reduces schedule overhead. Modify the code as following:  

	#pragma omp parallel for schedule(guided)
	for (int i = 1; i <= 8; i++)
	{
		......
	}
Build and run it:

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 sleeps 1 seconds
	Thread 0 sleeps 2 seconds
	Thread 1 sleeps 5 seconds
	Thread 0 sleeps 3 seconds
	Thread 0 sleeps 4 seconds
	Thread 1 sleeps 6 seconds
	Thread 0 sleeps 7 seconds
	Thread 1 sleeps 8 seconds
We can see it is not as the `schedule(dynamic)`, which every thread grabs the next iteration in turn: the first thread finished more tasks than the second one.  

We can also specify `chunk_size` in `schedule` clause. The default value of `chunk_size` in `schedule(dynamic)` and `schedule(guided)` is `1`, while in `schedule(static)` is undefined. Change the `schedule` to `schedule(dynamic, 2)`, then build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 sleeps 1 seconds
	Thread 1 sleeps 3 seconds
	Thread 0 sleeps 2 seconds
	Thread 1 sleeps 4 seconds
	Thread 0 sleeps 5 seconds
	Thread 1 sleeps 7 seconds
	Thread 0 sleeps 6 seconds
	Thread 1 sleeps 8 seconds
Compare the above output with the one of `schedule(dynamic)`:  we can see this time every thread will grab `2` iterations one time.  

There are `2` more options for `schedule` clause: `schedule(auto)` lets compiler and/or runtime system to determine the policy; `schedule(runtime)` makes scheduling deferred until run time, e.g., depends on `OMP_SCHEDULE` environment variable .