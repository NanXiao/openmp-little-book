# What is OpenMP?
What is `OpenMP`? Let's check the definition from its official [FAQ](http://www.openmp.org/about/openmp-faq/#WhatIs):  

> OpenMP is a specification for a set of compiler directives, library routines, and environment variables that can be used to specify high-level parallelism in Fortran and C/C++ programs.  

The following defintion is from [Wikipedia](https://en.wikipedia.org/wiki/OpenMP):  
> OpenMP (Open Multi-Processing) is an application programming interface (API) that supports multi-platform shared memory multiprocessing programming in C, C++, and Fortran, on most platforms, instruction set architectures and operating systems, including Solaris, AIX, HP-UX, Linux, macOS, and Windows. It consists of a set of compiler directives, library routines, and environment variables that influence run-time behavior.

To summarize, `OpenMP` defines a standard API which supports multi-thread programming on many different Operating Systems and CPU architectures. Now it can be used in 3 programming languages: `C`, `C++` and `Fortran`. The `OpenMP` API consists of compiler directives(`C`/`C++` begin with `#pragma omp`, and `Fortran` begins with `!$omp`), library routines, and environment variables.  

So for applications who uses `OpenMP`, `OpenMP` hides the details of thread management, such as creating, joining, scheduling threads, etc. The applications should concentrate on the code logic, and this will save a lot of energy from developers.