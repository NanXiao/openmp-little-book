# Set up compilation environment
I choose [Arch Linux](https://www.archlinux.org/) as the Operating System. For the compiler, both `gcc` and `clang` are OK.  

(1) Install `gcc`:  

	# pacman -S gcc

(2) Install `clang`:

	# pacman -S clang
	......
	Optional dependencies for clang
	    openmp: OpenMP support in clang with -fopenmp
	    python2: for scan-view and git-clang-format
	......

So to use `OpenMP` feature of `clang`, an additional `openmp` package needs to be installed:  

	# pacman -S openmp

Now that the environment is ready, let's write a simple program to get a first experience of `OpenMP`:  

	#include <omp.h>
	#include <stdio.h>
	
	int main(void) {
		#pragma omp parallel
		{
			printf("Thread id is %d\n", omp_get_thread_num());
		}
	    return 0;
	} 

The function of this program just spawns a herd of threads, and every thread prints its ID. My machine's CPU parameters are like following:  

	# lscpu
	Architecture:        x86_64
	CPU op-mode(s):      32-bit, 64-bit
	Byte Order:          Little Endian
	CPU(s):              2
	On-line CPU(s) list: 0,1
	Thread(s) per core:  1
	Core(s) per socket:  2
	Socket(s):           1
	......

Use `gcc` to build and run it:  

	# gcc -fopenmp thread.c
	# ./a.out
	Thread id is 0
	Thread id is 1

Switch to `clang`:  

	# clang -fopenmp thread.c
	# ./a.out
	Thread id is 0
	Thread id is 1

The same output! The program creates `2` threads which is equal to my machine's logical CPU number.  

BTW, use `ldd` to check the linked library of program built with `gcc`:  

	# ldd a.out
        linux-vdso.so.1 (0x00007ffcbbcf7000)
        libgomp.so.1 => /usr/lib/libgomp.so.1 (0x00007f15b75f7000)
        libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007f15b73d9000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007f15b7021000)
        libdl.so.2 => /usr/lib/libdl.so.2 (0x00007f15b6e1d000)
        /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007f15b7a27000)
built with `clang`:  

	# ldd a.out
        linux-vdso.so.1 (0x00007fffb5fb5000)
        libomp.so => /usr/lib/libomp.so (0x00007fcedc447000)
        libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007fcedc229000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007fcedbe71000)
        libdl.so.2 => /usr/lib/libdl.so.2 (0x00007fcedbc6d000)
        /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fcedc910000)
We can see `gcc`'s `OpenMP` library is `libgomp`, while `clang`'s is `libomp`.