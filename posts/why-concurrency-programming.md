# Why concurrency programming?
The CPU has entered the multi-core era for a long time. Take the following server information as an example:  

	$ lscpu
	Architecture:        x86_64
	CPU op-mode(s):      32-bit, 64-bit
	Byte Order:          Little Endian
	CPU(s):              104
	On-line CPU(s) list: 0-103
	Thread(s) per core:  2
	Core(s) per socket:  26
	Socket(s):           2
	NUMA node(s):        2
	......

This box has `2` physical CPUs, and every physical CPU has `26` cores. Since every core has hyper-thread support, there are totally `104` logical CPUs.  

When you run an application, the process will have only one main thread by default. See the following code:  
	
	void task(void)
	{
		// Do heavy work
		return;
	}
	
	int main(void)
	{
		int i = 0;
		
		for (i = 0; i < 10; i++)
		{
			task();
		}
		return 0;
	}

Let's assume the `task()` function will take a long time, and the process will execute task `10` times. If there is only main thread, the actual CPU utilization is like this:  
![image](https://raw.githubusercontent.com/NanXiao/openmp-concurrency-programming/master/images/why-concurrency-programming-htop-1.JPG)  
We can see there is only one CPU is in use. If the program can spawn another `9` threads (plus main thread, there are totally `10` threads), and every thread calls `task()` function, there will be `10` CPUs in use:  
![image](https://raw.githubusercontent.com/NanXiao/openmp-concurrency-programming/master/images/why-concurrency-programming-htop-2.JPG)  
And the whole running time will be reduced significantly.

A caveat which we need pay attention to is if there is only `1` logical CPU, though the process contains multiple threads, they are in fact run in **concurrency**, not in **parallelism**. Because in any time, there is only `1` thread running. In this scenario, the context-switch of threads may be a big overhead, using traditional single-thread model may be a better choice. 