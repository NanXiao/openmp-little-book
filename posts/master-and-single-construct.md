# Master and single construct
In `OpenMP`, `Master` construct identifies only master thread (who's thread ID is `0`) executes the region. For example:  

	#include <stdio.h>
	#include <omp.h>
			
	int main(void)
	{	
		#pragma omp parallel
		{
			#pragma omp master
			{
				printf("Thread %d is executing master construct\n", omp_get_thread_num());
			}
			printf("Thread %d is executing non-master construct\n", omp_get_thread_num());
		}
			
		return 0;
	}
Build and run it on my `24-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 23 is executing non-master construct
	Thread 3 is executing non-master construct
	Thread 14 is executing non-master construct
	Thread 11 is executing non-master construct
	Thread 10 is executing non-master construct
	Thread 16 is executing non-master construct
	Thread 9 is executing non-master construct
	Thread 15 is executing non-master construct
	Thread 12 is executing non-master construct
	Thread 6 is executing non-master construct
	Thread 7 is executing non-master construct
	Thread 17 is executing non-master construct
	Thread 13 is executing non-master construct
	Thread 19 is executing non-master construct
	Thread 5 is executing non-master construct
	Thread 21 is executing non-master construct
	Thread 4 is executing non-master construct
	Thread 18 is executing non-master construct
	Thread 22 is executing non-master construct
	Thread 20 is executing non-master construct
	Thread 0 is executing master construct
	Thread 0 is executing non-master construct
	Thread 1 is executing non-master construct
	Thread 8 is executing non-master construct
	Thread 2 is executing non-master construct
Only master thread outputs "`executing master construct`" message.  

There is another `single` construct which guarantees only one thread (no need to be the master thread) runs the region. Modify the above code:  

	#include <stdio.h>
	#include <omp.h>
			
	int main(void)
	{	
		#pragma omp parallel
		{
			#pragma omp single
			{
				printf("Thread %d is executing single construct\n", omp_get_thread_num());
			}
			printf("Thread %d is executing non-single construct\n", omp_get_thread_num());
		}
			
		return 0;
	}

Build and run it again:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Thread 22 is executing single construct
	Thread 14 is executing non-single construct
	Thread 23 is executing non-single construct
	Thread 4 is executing non-single construct
	Thread 16 is executing non-single construct
	Thread 18 is executing non-single construct
	Thread 5 is executing non-single construct
	Thread 9 is executing non-single construct
	Thread 12 is executing non-single construct
	Thread 3 is executing non-single construct
	Thread 15 is executing non-single construct
	Thread 7 is executing non-single construct
	Thread 2 is executing non-single construct
	Thread 10 is executing non-single construct
	Thread 21 is executing non-single construct
	Thread 8 is executing non-single construct
	Thread 13 is executing non-single construct
	Thread 6 is executing non-single construct
	Thread 20 is executing non-single construct
	Thread 1 is executing non-single construct
	Thread 17 is executing non-single construct
	Thread 11 is executing non-single construct
	Thread 0 is executing non-single construct
	Thread 19 is executing non-single construct
	Thread 22 is executing non-single construct
A random selected thread (whose ID is `22`) outputs "`executing single construct`".
