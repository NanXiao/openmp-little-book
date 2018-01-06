# Critical construct

Synchronization is the eternal theme in parallel programming. Check the following code:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		int sum = 0;
		
		#pragma omp parallel for
		for (int index = 1; index <= 10; index++)
		{
			sum += index;
		}
		
		printf("Sum is %d\n", sum);
		return 0;
	}

Compile and run it for several times on my `24-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Sum is 46
	# ./a.out
	Sum is 48
	# ./a.out
	Sum is 55

We can see the program didn't output correct result (`55`) every time. The root cause is multiple threads access the shared variable,`sum`, simultaneously, so we need the synchronization mechanism to protect it.  

The `critical` construct is used to produce a critical section:  

	#pragma omp critical [(name) [hint(hint-expression)] ] new-line
		structured-block
Which means there is only `1` thread executing code in section every time. Add "`#pragma omp critical`" in the above code:  

	......
	#pragma omp parallel for
	for (int index = 1; index <= 10; index++)
	{
		#pragma omp critical
		sum += index;
	}
	......

This time the result is always right:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Sum is 55
	# ./a.out
	Sum is 55
	# ./a.out
	Sum is 55

Please note that the critical section makes threads run actually in sequence, and this will downgrade the power of parallelism, so please pay attention to this caveat. 

BTW, the `critical` construct can have optional `name` and `hint`. The `name` is used to identify the `critical` construct, and all `critical` constructs without a name are considered to have the same unspecified name. For `hint`, I will explain it in following chapters.
