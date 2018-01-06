# Taskloop construct

The `taskloop` construct dispatches the iterations of `for-loop` to tasks to run, and its definition is like this:  

	#pragma omp taskloop [clause[[,] clause] ...] new-line
		for-loops

Check following example:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		#pragma omp taskloop
		for (int i = 0; i < 4; i++)
		{
			printf("Iteration %d is executed in thread %d\n", i, omp_get_thread_num());
		}
	
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Iteration 0 is executed in thread 0
	Iteration 1 is executed in thread 0
	Iteration 2 is executed in thread 0
	Iteration 3 is executed in thread 0
