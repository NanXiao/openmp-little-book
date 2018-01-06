# The basic parallel construct
Let's see the classic `OpenMP` "Hello World!" program:  

	#include <stdio.h>
	
	int main(void)
	{
		#pragma omp parallel
		{
			printf("Hello World!\n");
		}
		
		return 0;
	}
Check the kernel part:  

	#pragma omp parallel
	{
		printf("Hello World!\n");
	}
The whole part is called **construct**. The first line, "`#pragma omp parallel`" is **compiler directive**, and `parallel` is the **directive name**. The rest part, "`{ ... }`", is called **structured block**. The **directive** applies **structured block**, which means "`{ printf("Hello World!\n"); }`" will run in parallelism (In this simple example, the '`{`' and '`}`' are not necessary, but I think it is a good habit to keep them.).  

Compile and run the program on one `2-core` machine:  

	# gcc -fopenmp hello.c
	# ./a.out
	Hello World!
	Hello World!

The "`Hello World!`" is printed twice. Run it on another `24-core` machine: 

	# gcc -fopenmp hello.c
	# ./a.out
	Hello World!
	Hello World!
	......
	Hello World!
This time, "`Hello World!`" is outputted `24` times. It seems the thread number is always equal to CPU cores, but we **should not** assume it. The algorithm of generating how many threads is a little complicated, and the spec has a detailed description of it.  

We can use `num_threads` **clause** to set the requested number of threads. Change the directive as following:  

	#pragma omp parallel num_threads(4)

compile again and run the program on both the `2-core`  and `24-core` machines, and the "`Hello World!`" is outputted `4` times. Please notice that the Operating System will try best to allocate `num_threads` threads, but due to the limitation on the resources, it is not guaranteed. For example:  

	#pragma omp parallel num_threads(300000)

The above directive will cause program crash on my `2-core` machine. 

In fact, the general format `OpenMP` directive for `C/C++` is like this:   

	#pragma omp directive-name [clause[ [,] clause] ... ] new-line 

"`#pragma omp`" identifies that this is used for `OpenMP`; every directive must have one `directive-name` (e.g., `parallel`.), and `0` or more clauses (e.g., `num_threads`.) follow it. At the end of directive, there should be a new-line character, e.g, on `Unix`, it is `\n`.
