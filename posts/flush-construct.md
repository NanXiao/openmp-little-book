# Flush construct

`Flush` construct is used to update data in memory so other
threads see the newest value, and guarantee all threads have the same view of the data. Its definition is like this:  

	#pragma omp flush [(list)] new-line

Check following example:  

	#include <stdio.h>
	#include <unistd.h>
	#include <omp.h>
		
	int main(void)
	{	
		int a = 100;
		
		#pragma omp parallel for
		for (int i = 0; i < 2; i++)
		{
			if (0 == omp_get_thread_num())
			{
				a++;
				#pragma omp flush
			}
			else
			{
				sleep(1);
				printf("a is %d\n", a);
			}
		}
	
		return 0;
	}

The master thread uses "`#pragma omp flush`" so the second thread always gets `101` from `a`'s memory. Check the assembly code of the program:  

		......
        movq    -40(%rbp), %rax
        movl    (%rax), %eax
        leal    1(%rax), %edx
        movq    -40(%rbp), %rax
        movl    %edx, (%rax)
        mfence
		......

We can see "`#pragma omp flush`" is converted to `mfence` instruction in `x86` platform.
 