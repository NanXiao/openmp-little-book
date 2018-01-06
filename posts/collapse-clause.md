# Collapse clause

The `collapse` clause is used to convert a prefect nested loop into a single loop then parallelize it. Check the following example:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		#pragma omp parallel for
		for (int i = 0; i < 4; i++)
		{
			for (int j = 0; j < 5; j++)
			{
				printf("Thread number is %d\n", omp_get_thread_num());
			}
		}
		
		return 0;
	}  
Compile and run it on my `24-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread number is 0
	Thread number is 0
	Thread number is 0
	Thread number is 0
	Thread number is 0
	Thread number is 3
	Thread number is 3
	Thread number is 3
	Thread number is 3
	Thread number is 3
	Thread number is 1
	Thread number is 1
	Thread number is 1
	Thread number is 1
	Thread number is 1
	Thread number is 2
	Thread number is 2
	Thread number is 2
	Thread number is 2
	Thread number is 2
Every iteration of outer loop will be dispatched to `1` every thread to run:  

	#pragma omp parallel for
	for (int i = 0; i < 4; i++)
	{
		......
	}

Every thread will execute the inner `for-loop` sequentially:  

	for (int j = 0; j < 5; j++)
	{
		......
	}
So there are only `4` threads in active state actually.  

Modify the directive from  

	#pragma omp parallel for
to  

	#pragma omp parallel for collapse(2)
Then build and run it again:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread number is 0
	Thread number is 2
	Thread number is 18
	Thread number is 16
	Thread number is 6
	Thread number is 8
	Thread number is 7
	Thread number is 10
	Thread number is 14
	Thread number is 12
	Thread number is 13
	Thread number is 17
	Thread number is 15
	Thread number is 9
	Thread number is 11
	Thread number is 19
	Thread number is 4
	Thread number is 3
	Thread number is 5
	Thread number is 1
This time we can see `20` threads are utilized. The integer after `collapse` (i.e., `2` in this example) identifies how many loops to be parallelized, and counted from outer side to inner side. Please be aware that `collapse(1)` and no `collapse` take the same effect for loop parallelism.