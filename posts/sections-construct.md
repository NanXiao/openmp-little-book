# Sections construct

The `sections` construct defines some sections which run in parallel:  

	#pragma omp sections [clause[ [,] clause] ... ] new-line
	{[
		#pragma omp section new-line]
			structured-block
		[#pragma omp section new-line
			structured-block]
		...
	}
Check the following code:  

	#include <stdio.h>
	#include <omp.h>
	
	int main(void)
	{		
		#pragma omp parallel
		{
			#pragma omp sections
			{
				printf("Thread %d is running\n", omp_get_thread_num());
				#pragma omp section
				printf("Thread %d is running\n", omp_get_thread_num());
				#pragma omp section
				printf("Thread %d is running\n", omp_get_thread_num());
				#pragma omp section
				printf("Thread %d is running\n", omp_get_thread_num());
			}
		}
		
		return 0;
	}
Build and run it on my `24-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is running
	Thread 21 is running
	Thread 22 is running
	Thread 8 is running
We can see `4` sections (for the first section, `#pragma omp section` is optional) are dispatched to different threads to run. Like `for-loop` construct, the following shortcut 

	#pragma omp parallel sections
	{
		......
	}  
if often used instead of:  

	#pragma omp parallel
	{
		#pragma omp sections
		{
			......
		}
	}
Furthermore, some clauses, like `private` and `firstprivate`, can also be used for `sections` construct.