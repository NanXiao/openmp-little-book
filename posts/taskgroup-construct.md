# Taskgroup construct

When a running task meets `taskgroup` construct:  

	#pragma omp taskgroup new-line
		structured-block

It will pause, and won't execute again until all child and descendant tasks in `taskgroup` finish. Check the following example:  

	#include <stdio.h>
	#include <omp.h>
	#include <unistd.h>
		
	int main(void)
	{	
		#pragma omp task
		{
			printf("Thread %d is in the beginning of main task\n", omp_get_thread_num());
			#pragma omp taskgroup
			{
				#pragma omp task
				{
					sleep(1);
					printf("Thread %d is in running sub-task 1\n", omp_get_thread_num());
				}
				#pragma omp task
				{
					sleep(1);
					printf("Thread %d is in running sub-task 2\n", omp_get_thread_num());
				}
			}
			printf("Thread %d is in the ending of main task\n", omp_get_thread_num());
		}
	
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 0 is in the beginning of main task
	Thread 0 is in running sub-task 1
	Thread 0 is in running sub-task 2
	Thread 0 is in the ending of main task

We can see even though the sub-tasks in `taskgroup` will sleep (call `sleep`), the main task won't be scheduled to run. The main task will wait all tasks in `taskgroup` end.

