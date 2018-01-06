# Task construct

Check following code:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		#pragma omp parallel num_threads(4)
		{
			printf("Thread %d is running\n", omp_get_thread_num());
		}
	
		return 0;
	}

What "`#pragma omp parallel num_threads(4)`" region actually does is creating `4` threads, and every thread is assigned an implicit task which prints the log.  

Besides implicit task, using `task` construct can create explicit task. Check following code:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		#pragma omp parallel num_threads(4)
		{
			if (1 == omp_get_thread_num())
			{
				#pragma omp task
				printf("Thread %d is running an explicit task\n", omp_get_thread_num());
			}
			printf("Thread %d is running\n", omp_get_thread_num());
		}
	
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is running
	Thread 3 is running
	Thread 1 is running
	Thread 0 is running an explicit task
	Thread 2 is running

We can see thread whose ID is `1` created the task, but it was actually master thread  who executed the task. The explicit task can run immediately or deferred to the future (like the above example).
