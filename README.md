最初实现SM3时，使用的是string类，并且自定义了很多函数用于进制转换，位运算等操作，在转换的过程中势必会浪费很多不必要的时间，因而使用unsigned int类型重新实现SM3。

与使用多线程优化SM4不同，SM4是对称加密算法，我们可以将彼此之间没有数据依赖的部件分配到多个线程并行计算。而SM3本质上是求哈希值，因而整体优化思路是测量计算1000次所需的时间，将任务分配到多个线程并行处理。

定义以下两个函数(以四线程为例)：
```
void Thread4(unsigned char* message, uint len, unsigned char* Hash)
{
	for (int i = 0; i < 250; i++)
		SM3(message, len, Hash);
}

void thread4(unsigned char* message, uint len, unsigned char* Hash)
{
	thread* threads = new thread[thread_num];
	int i = 0;
	for (i = 0; i < thread_num; i++)
		threads[i] = thread(Thread4, ref(message), len, Hash);
	for (i = 0; i < thread_num; i++)
		threads[i].join();
}
```

主函数中调用：
```
thread4(message, len, digest);
```

经过多次单线程、四线程与八线程时间对比，使用四线程对SM3进行优化，速度再次提升了3倍左右。测试还发现，使用8个线程并没有进一步提升速度，个人认为原因在于与计算相比，创建新线程会产生更大的开销，因而影响了速度的提升。
