# Private, firstprivate and lastprivate clauses

Check the following program:  

	#include <stdio.h>
	#include <unistd.h>
	#include <omp.h>
		
	int main(void)
	{	
		int var = 100;
		
		printf("Before parallelism, var's address is %p\n", &var);
		
		#pragma omp parallel for
		for (int i = 0; i < 4; i++)
		{
			printf("var's address in thread %d is %p\n", omp_get_thread_num(), &var);
			var = i;
		}
		
		printf("After parallelism, var's address is %p, and value is %d\n", &var, var);
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Before parallelism, var's address is 0x7ffe993d79fc
	var's address in thread 1 is 0x7ffe993d79fc
	var's address in thread 2 is 0x7ffe993d79fc
	var's address in thread 3 is 0x7ffe993d79fc
	var's address in thread 0 is 0x7ffe993d79fc
	After parallelism, var's address is 0x7ffe993d79fc, and value is 0
	# ./a.out
	Before parallelism, var's address is 0x7ffc13c7cd8c
	var's address in thread 3 is 0x7ffc13c7cd8c
	var's address in thread 2 is 0x7ffc13c7cd8c
	var's address in thread 0 is 0x7ffc13c7cd8c
	var's address in thread 1 is 0x7ffc13c7cd8c
	After parallelism, var's address is 0x7ffc13c7cd8c, and value is 1
	# ./a.out
	Before parallelism, var's address is 0x7ffd72be3bbc
	var's address in thread 3 is 0x7ffd72be3bbc
	var's address in thread 2 is 0x7ffd72be3bbc
	var's address in thread 0 is 0x7ffd72be3bbc
	var's address in thread 1 is 0x7ffd72be3bbc
	After parallelism, var's address is 0x7ffd72be3bbc, and value is 1

Every thread prints the same address value of `var` variable, the `var` variable is shared in all threads. Because there is no protection of accessing shared variable, the final value of `var` is undefined.  

Use `private` clause can let every thread get its own copy of shared variable, but the initial value is still unspecified:  

	#include <stdio.h>
	#include <unistd.h>
	#include <omp.h>
		
	int main(void)
	{	
		int var = 100;
		
		printf("Before parallelism, var's address is %p\n", &var);
		
		#pragma omp parallel for private(var)
		for (int i = 0; i < 4; i++)
		{
			printf("var's address in thread %d is %p, value is %d\n", omp_get_thread_num(), &var, var);
			var = i;
		}
		
		printf("After parallelism, var's address is %p, and value is %d\n", &var, var);
		return 0;
	} 
Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Before parallelism, var's address is 0x7ffde0e4c124
	var's address in thread 3 is 0x7f0fe9bf0e20, value is 0
	var's address in thread 1 is 0x7f0feabf2e20, value is 0
	var's address in thread 2 is 0x7f0fea3f1e20, value is 0
	var's address in thread 0 is 0x7ffde0e4c0c0, value is 24
	After parallelism, var's address is 0x7ffde0e4c124, and value is 100

From the printed address of `var` variable in every thread, we can see every thread gets a local copy of `var`, but the initial value is random. The local `var` variable in `main` function is no affected.  

`firstprivate` clause makes a copy from the shared variable:  

	#include <stdio.h>
	#include <unistd.h>
	#include <omp.h>
		
	int main(void)
	{	
		int var = 100;
		
		printf("Before parallelism, var's address is %p\n", &var);
		
		#pragma omp parallel for firstprivate(var)
		for (int i = 0; i < 4; i++)
		{
			printf("var's address in thread %d is %p, value is %d\n", omp_get_thread_num(), &var, var);
			var = i;
		}
		
		printf("After parallelism, var's address is %p, and value is %d\n", &var, var);
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Before parallelism, var's address is 0x7ffe3ae46e00
	var's address in thread 0 is 0x7ffe3ae46da0, value is 100
	var's address in thread 3 is 0x7ff6870ede20, value is 100
	var's address in thread 2 is 0x7ff6878eee20, value is 100
	var's address in thread 1 is 0x7ff6880efe20, value is 100
	After parallelism, var's address is 0x7ffe3ae46e00, and value is 100

The `var` in every thread is initialized as `100`.  

`lastprivate` clause makes the value in last sequential iteration assigned back to the shared variable:  

	#include <stdio.h>
	#include <unistd.h>
	#include <omp.h>
		
	int main(void)
	{	
		int var = 100;
		
		printf("Before parallelism, var's address is %p\n", &var);
		
		#pragma omp parallel for lastprivate(var)
		for (int i = 0; i < 4; i++)
		{
			printf("var's address in thread %d is %p, value is %d\n", omp_get_thread_num(), &var, var);
			var = i;
		}
		
		printf("After parallelism, var's address is %p, and value is %d\n", &var, var);
		return 0;
	}

Build and run it:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Before parallelism, var's address is 0x7ffd53ea3fb0
	var's address in thread 0 is 0x7ffd53ea3f50, value is 24
	var's address in thread 3 is 0x7fd70584fe20, value is 0
	var's address in thread 2 is 0x7fd706050e20, value is 0
	var's address in thread 1 is 0x7fd706851e20, value is 0
	After parallelism, var's address is 0x7ffd53ea3fb0, and value is 3
	# ./a.out
	Before parallelism, var's address is 0x7fff72604380
	var's address in thread 0 is 0x7fff72604320, value is 24
	var's address in thread 2 is 0x7f6f3ac49e20, value is 0
	var's address in thread 1 is 0x7f6f3b44ae20, value is 0
	var's address in thread 3 is 0x7f6f3a448e20, value is 0
	After parallelism, var's address is 0x7fff72604380, and value is 3

Since in following loop:  

	for (int i = 0; i < 4; i++)
	{
		......
	}

`i = 3` is the last sequential iteration, the shared variable `var` is always updated to `3`.