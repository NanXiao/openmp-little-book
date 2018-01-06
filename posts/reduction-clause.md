# Reduction Clause

Check the following code:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		int sum = 100;
		
		#pragma omp parallel for
		for (int i = 1; i <= 4; i++)
		{
			sum += i;
		}
		
		printf("sum is %d\n", sum);
		return 0;
	}

Because `sum` is a shared variable in threads, so we need to use synchronization to protect accessing it:  

	#pragma omp parallel for
	for (int i = 1; i <= 4; i++)
	{
		#pragma omp critical
		sum += i;
	}

But this will cause losing the advantage of using parallelism. The other method is using `reduction` clause:

    reduction(reduction-identifier : list)	
Modify the above code to understand `reduction` clause better:  

	#include <stdio.h>
	#include <omp.h>
		
	int main(void)
	{	
		int sum = 100;
		
		printf("Before parallelism, sum's address is %p\n", &sum);
		
		#pragma omp parallel for reduction(+ : sum)
		for (int i = 1; i <= 4; i++)
		{
			printf("sum's address in thread %d is %p, value is %d\n", omp_get_thread_num(), &sum, sum);
			sum += i;
		}
		
		printf("After parallelism, sum's address is %p, and value is %d\n", &sum, sum);
		return 0;
	}  

In above `reduction(+ : sum)` clause, `+` is `reduction-identifier` and `sum` is `list`. Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Before parallelism, sum's address is 0x7ffcc880baf0
	sum's address in thread 3 is 0x7f6baea5ee20, value is 0
	sum's address in thread 2 is 0x7f6baf25fe20, value is 0
	sum's address in thread 0 is 0x7ffcc880ba90, value is 0
	sum's address in thread 1 is 0x7f6bafa60e20, value is 0
	After parallelism, sum's address is 0x7ffcc880baf0, and value is 110

From the output, we can see every thread gets a local copy of `sum` (different addresses), but it is initialized to `0` (this is according to the `+` operation), not an unspecified value. After finishing this parallel region, the local `sum` of every thread are reduced to one value, and combined to the original shared `sum` defined in `main` function (the value is `110`).  

Besides `+`, `OpenMP` also supports other operators, such as `-`, `*`, and so on. Based on the `reduction-identifier`, the initial values of `list` are also different. E.g., for `*`, the initial value is `1`, not `0`.    
