# Barrier

Barrier is a point where each thread of the team will wait there until all threads arrive. There are implicit barriers at the end of `parallel` construct ("`#pragma omp parallel`") and the end of worksharing constructs(`loop`, `sections`, `single` and `workshare` constructs). Check following example:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		#pragma omp parallel
		{
			printf("Thread %d is running before implicit barrier\n", omp_get_thread_num());
		}
		printf("Back to main thread\n");
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is running before implicit barrier
	Thread 3 is running before implicit barrier
	Thread 1 is running before implicit barrier
	Thread 2 is running before implicit barrier
	Back to main thread
	# ./a.out
	Thread 0 is running before implicit barrier
	Thread 1 is running before implicit barrier
	Thread 3 is running before implicit barrier
	Thread 2 is running before implicit barrier
	Back to main thread

There is implicit barrier in "`#pragma omp parallel`" construct, so it means no matter how many times you run the program, "`Back to main thread`" is always the last output.  

Besides the implicit barriers, there is another explicit `barrier` construct:  

	#pragma omp barrier new-line
Check the following code:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		#pragma omp parallel
		{
			printf("Thread %d prints 1\n", omp_get_thread_num());
			printf("Thread %d prints 2\n", omp_get_thread_num());
		}
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 prints 1
	Thread 0 prints 2
	Thread 1 prints 1
	Thread 1 prints 2
	Thread 3 prints 1
	Thread 3 prints 2
	Thread 2 prints 1
	Thread 2 prints 2

We can see the outputs of "`...prints 1`"s and "`prints 2`"s are interleaved. Add "`#pragma omp barrier`" into the `parallel` construct:  

	#pragma omp parallel
	{
		printf("Thread %d prints 1\n", omp_get_thread_num());
		#pragma omp barrier
		printf("Thread %d prints 2\n", omp_get_thread_num());
	}

Build and run it again:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 prints 1
	Thread 3 prints 1
	Thread 2 prints 1
	Thread 1 prints 1
	Thread 3 prints 2
	Thread 1 prints 2
	Thread 2 prints 2
	Thread 0 prints 2
All "`...prints 1`"s will be printed before "`...prints 2`".  

Using `nowait` clause can "break" the barrier. Check following example:  

	#include <stdio.h>
	#include <omp.h>
	
	int main(void)
	{	
		#pragma omp parallel
		{
			#pragma omp for
			for (int i = 0; i < 4; i++) {
				printf("Thread %d is running in first loop\n", omp_get_thread_num());
			}
	
			#pragma omp for
			for (int i = 0; i < 4; i++) {
				printf("Thread %d is running in second loop\n", omp_get_thread_num());
			}
		}
	
		return 0;
	}

Build and run the program:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is running in first loop
	Thread 2 is running in first loop
	Thread 3 is running in first loop
	Thread 1 is running in first loop
	Thread 2 is running in second loop
	Thread 3 is running in second loop
	Thread 0 is running in second loop
	Thread 1 is running in second loop
	# ./a.out
	Thread 0 is running in first loop
	Thread 1 is running in first loop
	Thread 3 is running in first loop
	Thread 2 is running in first loop
	Thread 1 is running in second loop
	Thread 3 is running in second loop
	Thread 2 is running in second loop
	Thread 0 is running in second loop

Because of the implicit barrier in "`pragma omp for`" region, the second "`for-loop`" won't run until the threads in first "`for-loop`" all finish. Now add `nowait` clause in first "`for-loop`" construct:  

	#pragma omp for nowait
	for (int i = 0; i < 4; i++) {
		printf("Thread %d is running in first loop\n", omp_get_thread_num());
	}
Build and run program again:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is running in first loop
	Thread 0 is running in second loop
	Thread 3 is running in first loop
	Thread 3 is running in second loop
	Thread 1 is running in first loop
	Thread 1 is running in second loop
	Thread 2 is running in first loop
	Thread 2 is running in second loop
This time, `nowait` clause ruins the implicit barrier in first "`for-loop`" construct. So when a thread finishes executing in first "`for-loop`" construct, it won't wait others, and enter next "`for-loop`" construct.
