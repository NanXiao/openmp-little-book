# SIMD construct
`SIMD` construct can leverage some hardware vector processing units to accelerate computation. For example, the following `CPU` supports `AVX` instructions:  

	# lscpu
	......
	avx512f avx512dq rdseed adx smap clflushopt clwb avx512cd avx512bw avx512vl ......

Check the following code:  

	#include <stdio.h>
	#include <omp.h>
	
	#define ARRAY_SIZE	(10240)
	float A[ARRAY_SIZE];
	float B[ARRAY_SIZE];
	float C[ARRAY_SIZE];
		
	int main(void)
	{	
		for (int i = 0; i < ARRAY_SIZE; i++)
		{
			A[i] = i * 2.3;
			B[i] = i + 4.6;
		}
	
		double start = omp_get_wtime();
		for (int loop = 0; loop < 1000000; loop++)
		{
			for (int i = 0; i < ARRAY_SIZE; i++)
			{
				C[i] = A[i] * B[i];
			}
		}
		double end = omp_get_wtime();
		printf("Work consumed %f seconds\n", end - start);
		return 0;
	}
Build and run it:  

	# gcc -O2 -fopenmp parallel.c
	# ./a.out
	Work consumed 4.805690 seconds
It consumes ~`4.8` seconds to finish the following code:  

	for (int loop = 0; loop < 1000000; loop++)
	{
		for (int i = 0; i < ARRAY_SIZE; i++)
		{
			C[i] = A[i] * B[i];
		}
	}
Change the above code as following:  

	for (int loop = 0; loop < 1000000; loop++)
	{
		#pragma omp simd
		for (int i = 0; i < ARRAY_SIZE; i++)
		{
			C[i] = A[i] * B[i];
		}
	}
Build and run the program again:  

	# gcc -O2 -fopenmp parallel.c
	# ./a.out
	Work consumed 1.773028 seconds

This time the execution time is only ~`1.8` seconds.
