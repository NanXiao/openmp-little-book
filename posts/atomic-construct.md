# Atomic construct

Apart from using `critical` construct to synchronize accessing the same variable, `atomic` is another choice. Check the following code:  

	#include <stdio.h>
	#include <omp.h>
			
	int main(void)
	{	
		int sum = 0;
	
		#pragma omp parallel for
		for (int index = 1; index <= 10; index++)
		{
			#pragma omp atomic
			sum += index;
		}
			
		printf("Sum is %d\n", sum);
		return 0;
	} 
Build and run it on my `24-core` machine:  

	# gcc -fopenmp parallel.c
	# ./a.out
	Sum is 55
	# ./a.out
	Sum is 55
	# ./a.out
	Sum is 55
	# ./a.out
	Sum is 55
The `atomic` ensures updating `sum` variable is an atomic operation, so the program always calculate the correct result. Besides `+`, `atomic` construct also supports other operators, such as: `-`, `*`, `/`, etc. Similarly, `-=`, `*=`, `/=` are covered too. 

