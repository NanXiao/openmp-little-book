# Parallel loop construct

Parallel loop construct is a very common idiom in `OpenMP` programming. The following is the syntax of **loop construct**:  

	#pragma omp for [clause[ [,] clause] ... ] new-line
		for-loops

What **loop construct** does is distributing every iteration into threads in team, and makes iterations run concurrently. Check the following example:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{		
		#pragma omp for
		for (int index = 0; index < 10; index++)
		{
			printf("Thread id is %d\n", omp_get_thread_num());
		}
			
		return 0;
	}

Build and run it on my `24-core` machine:  

	$ gcc -fopenmp parallel.c
	$ ./a.out
	Thread id is 0
	Thread id is 0
	Thread id is 0
	Thread id is 0
	Thread id is 0
	Thread id is 0
	Thread id is 0
	Thread id is 0
	Thread id is 0
	Thread id is 0
Since there is only main thread, the loop executed in sequential in fact. Add "`#pragma omp parallel`" to spawn other threads:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		#pragma omp parallel
		{
			#pragma omp for
			for (int index = 0; index < 10; index++)
			{
				printf("Thread id is %d\n", omp_get_thread_num());
			}
		}
			
		return 0;
	}
Build and run it again:  

	$ gcc -fopenmp parallel.c
	$ ./a.out
	Thread id is 1
	Thread id is 0
	Thread id is 9
	Thread id is 6
	Thread id is 3
	Thread id is 7
	Thread id is 2
	Thread id is 5
	Thread id is 4
	Thread id is 8

This time every iteration is dispatched to different thread to run. Actually, we always use the following format:  

	#pragma omp parallel for
	for (int index = 0; index < 10; index++)
	{
		......
	}
which is the shortcut of:  

	#pragma omp parallel
	{
		#pragma omp for
		{
			......
		}
	}
	
P.S. Please do not mix up the following code:

	#pragma omp parallel
	{
		for (...)
		{
			......
		}
	}
Which means executing `for-loops` in every thread.