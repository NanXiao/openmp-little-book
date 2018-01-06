# Ordered construct

The `ordered` construct is used to guarantee the code run in iteration order of loop. Check the following example:  

	#include <stdio.h>
	#include <omp.h>
			
	int main(void)
	{	
		#pragma omp parallel for
		for (int i = 1; i <= 8; i++)
		{
			printf("Thread %d is executing iteration %d \n", omp_get_thread_num(), i);
		}
			
		return 0;
	}

Build and run it on my `24-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is executing iteration 1
	Thread 6 is executing iteration 7
	Thread 7 is executing iteration 8
	Thread 5 is executing iteration 6
	Thread 4 is executing iteration 5
	Thread 2 is executing iteration 3
	Thread 3 is executing iteration 4
	Thread 1 is executing iteration 2
We can see the execution order of every iteration is random. Modify the `for-loop` code as following:  

	......
	#pragma omp parallel for ordered
	for (int i = 1; i <= 8; i++)
	{
		#pragma omp ordered
		printf("Thread %d is executing iteration %d \n", omp_get_thread_num(), i);
	}
	......

Build and run the program again:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is executing iteration 1
	Thread 1 is executing iteration 2
	Thread 2 is executing iteration 3
	Thread 3 is executing iteration 4
	Thread 4 is executing iteration 5
	Thread 5 is executing iteration 6
	Thread 6 is executing iteration 7
	Thread 7 is executing iteration 8

This time the program outputs in iteration sequence.  

Please notice for the code outside of the `ordered` construct, it still runs in concurrency. Modify the `for-loop`:  

	......
	#pragma omp parallel for ordered
	for (int i = 1; i <= 8; i++)
	{
		printf("Thread %d is executing iteration %d in no-order\n", omp_get_thread_num(), i);
		#pragma omp ordered
		printf("Thread %d is executing iteration %d \n", omp_get_thread_num(), i);
	}
	......
Build and run it again:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is executing iteration 1 in no-order
	Thread 1 is executing iteration 2 in no-order
	Thread 5 is executing iteration 6 in no-order
	Thread 7 is executing iteration 8 in no-order
	Thread 4 is executing iteration 5 in no-order
	Thread 0 is executing iteration 1
	Thread 1 is executing iteration 2
	Thread 3 is executing iteration 4 in no-order
	Thread 6 is executing iteration 7 in no-order
	Thread 2 is executing iteration 3 in no-order
	Thread 2 is executing iteration 3
	Thread 3 is executing iteration 4
	Thread 4 is executing iteration 5
	Thread 5 is executing iteration 6
	Thread 6 is executing iteration 7
	Thread 7 is executing iteration 8


