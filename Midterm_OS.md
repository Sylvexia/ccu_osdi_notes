1. Why do we need dual mode?
	1. Dual mode can only call syscall in order to execute privilege command and access to kernel space.
	2. Kernel mode: Can execute user space, privilege instruction and kernel space
	3. User mode: Can only execute user space.
2. What is the difference between context switch and mode change?
	1. Context-switch: switch of different task(process).
	2. Mode change: Switch of different permission mode, like user mode to kernel mode
	3. Overhead:
		1. Context-switch: may trigger a lot of cache miss especially the context-switch between process.
		2. Mode change: may only trigger CPU execute mode, smaller overhead.
3. DMA and cache, coherency problem
	1. From device to CPU, the hardware would make DMA update to cache.
	2. From CPU to device, we write through from CPU to device. 
4. How do DMA improve the CPU performance?
	1. Say you want to copy a large chunk of file on disk, the disk access speed is slower than the bus and CPU, so DMA controller transfer the data to buffer to speed up the performance.
	2. The CPU don't need to transfer the data manually, the CPU configure the DMA to manage the data movement, hence the CPU is free from transferring the data and can do other stuff.
	3. The CPU is interrupted after DMA finished the data transfer.
5. In Linux, there's 3 kinds of memory, buffer, cache, and program memory, state the difference.
	1. Cache Memory: 
		1. Store data in disk or ssd to cache memory so we can speed up the access speed.
	2. Buffer memory:
		1. The data transmission between I/O
		2. It's used to bridge the speed gap between CPU and peripherals.
	3. Program Memory:
		1. Give the memory to program. (e.x.: malloc, mmap to request memory from OS)
		2. The executable should be loaded to primary memory to execute.
6. What is the difference between memory-mapped I/O and port I/O, explain in concise words and in the perspective of assembly.
	1. Memory-mapped I/O
		1. Mapping the register and memory to CPU memory space.
		2. Handle memory as unified layout of memory address space.
		3. The CPU access to data with single bus.
			1. Assembly: 
			```MOV CX, 0xfffffff``` (Cx store to 0xfffffff)
	1. Port-mapped I/O
		1. Transmitting the data to specific port.
		2. Handle memory using the dedicated input and output port.
		3. The memory bus and I/O bus is separated bus.
		4. Assembly: 
			1. ```out 0x255 ax``` (write ax to 0x255)
			2. ```in ax 0x100``` (write to ax from 0x100)
7. What's the difference between interrupt and function call.
	1. Interrupts are invoked by hardware or peripheral. Function call is invoked by program.
	2. The hardware and peripheral generate and send IRQ interrupt request) to CPU. The CPU switch to kernel modem, and disable the interrupt.
	3. CPU uses IVT (interrupt Vector Table) to determine the address of ISR (Interrupt Service Request), ISR save the PC counter, register and the context of the task. These all happens in top half.
	4. ISR invoked bottom half, and execute the kernel thread stuff.
	5. After completing the interrupt task, the CPU restore the state from stack and resume the state.
8. Difference between SMT and CMT
	1. SMT
		1. multiple thread on 1 processor.
		2. single core interleave the same instruction.
		3. focus on utilization of single core.
		4. e.g. Intel hyperthreading, SMT AMD
	2. CMT
		1. multiple processor on 1 chip
		2. each core have its own set of instruction.
		3. focus one overall processing power.
		4. e.g. Oracle SPARC
9. Why vDSO syscall is faster than conventional syscall
	1. vDSO syscall(e.g. ```_vdso_clock_gettime``` ) run in user space, which don't require context switch, which means lower cache missed, which improve the performance.
10. In Linux 2.4 kernel, how to distinguish the I/O bounded task?
	1. Separate the run time to several epoch. When no task can execute, switch to next epoch.
	2. When switch to next epoch, increment all task time slice.
		1. Higher time slice means higher dynamic priority in Linux.
		2. If the task is I/O bound, it wouldn't complete at last epoch. the priority will get higher. (Exactly how it separate the I/O bound)
	3. ```time_slice=time_slice/2 + basetimeslice(nice)```
	4. cons: 
		1. Calculate all goodness takes time.
		2. All CPU use 1 run queue, also when context switch it need to traverse all run queue to find the highest priority task.
11. Please explain in CFS, why does higher priority task have shorter response time, and more CPU use time.
	1. Higher priority task fills back to task queue more quickly. 
	2. Compare to O(1) scheduler, you have to wait a epoch to refill the time slice.
	3. Basically the shorter ```vruntime``` is, the shorter response time is. 
		1. The basic idea: it runs a unit of time. (expected_time/num_task)
		2. Pick a task that has smallest ```vruntime```, if same FIFO.
		3. After execute a task, the ```vruntime``` is increased by 1/```proc_power```, 
12. Peterson Solution:
```
P0: 
flag[0] = true;
turn = 1;
while (flag[1] == true && turn == 1)
{
	// busy wait
}
// critical section
...
// end of critical section
flag[0] = false;
```

```
P1: 
flag[1] = true;
turn = 0;
while (flag[0] == true && turn == 0)
{
	// busy wait
}
// critical section
...
// end of critical section
flag[1] = false;
```

1. Mutual exclusion:
	1. if P0 at critical section, ```flag[1]``` is ```false``` or ```turn == 0```
	2. Which means P1 left critical section, or haven't enter the critical section.
	3. If P0, P1 both want to enter, ```flag[0]==flag[1]==true```, swapping ```flag[i]=true``` and ```turn = j``` will failed! 
2. Progress:
	1. If P0 want to enter CS, P1 don't want to.
		1. ```flag[1]==false```
	2. If P0 want to enter CS, P1 want to enter but p1 only to execute to ```flag[1]==true;```
		1. ```turn==1```
	3. If P0 and P1 both want to enter CS
		1. Same for mutex.
3. Bounded waiting:
	1. You have to prove the execute time is finite, not the run time.
	2. If P0 keep wanting to entering CS, will it enter to CS?
		1. If P0 first, P1 will be next.
		2. ```turn = 1 ``` will guarantee P1 will enter the CS first.