
## Mutex

- Mutex = spinlock + sleep + wakeup
	- sleep+wakeup=futex (syscall)
		- futex:

```
void lock(locker)
{
	int count=0;
	//spinlock wait 100 iteration
	while(locker==1){
		count++;
		if(count>1000)
			break;
	}
	// realease CPU
	futex_wait(locker,1,0)
		
}
```
- Adaptive mutex:
	- set spinlock loop iteration
	- if not sucess than LLL_MUTEX_LOCK(mutex)

## sequential lock

- reader-writer
- the cost is on reader, writer has higher priority
- you can only have one writer
- how reader know how to redo?

## Ticket lock

- FIFO spinlock

## rw-spinlock:

- before writer enter CS, took large of ticket, make sure other writer and reader cannot enter CS.
- reader enter CS, one ticket a time, allow multiple reader enter CS

## RCU

- multi-version, let reader have more parallelization
- writer are exclusive to each other

## Memory order:

- relaxed: you can move instruction up and down
- acquire: your instruction cannot move over to the top
- release: your instruction cannot move below it.
- acq_rel(cut): 
	- e.g.: merge: you have done everything before, anything thing after it cannot move to here.
- seq_cst(the strongest): have all acq_rel attribute. And make sure:
	- program order
	- write atomicity

## Deadlock:

1. Mutual Exclusion: Resource cannot share between process.
	1. Solution: Critical section, let CPU do it's own task.
2. Hold & Wait: A process holding resource and waiting to acquire additional resources that are currently held by other processes.
	1. lock all resource (hold)
	2. release all resource when acquiring resource. (wait)
		1. may introduce starvation
3. No Preemption: Resources cannot be forcibly taken away from a process.
	1. rollback (sequential lock)
	2. multiversion (RCU)
4. Circular wait: There must exist a circular chain in resource allocation graph.
	1. Solution: lock the resource sequentially.

resolution:
- roll back
- kill task

## Livelock:
 - a situation in computing where two or more processes or entities are actively responding to each other's actions, but their interactions prevent any of them from making progress.

## Starvation:
- Process(es) are not able to get the resource it need, leading delay of infinite loop.
- example: Process A and Process B  share the same resource, if resource allocation favor A, B would not get the resource it want.

## Priority inversion

- High priority task has to wait low priority
	- solution: 
		- priority inheritance
		- priority ceiling protocol

## DRAM

- usage: 
	- for program execution
	- for cache memory (HDD, SSD)
	- for buffer that communicate between the periperal (e.g.DMA)

## Fragmentation:

- External fragmentation: When memory allocation, create some small memory gaps that is hard to utilize.
- Internal fragmentation: Allocate memory that more than it actually need.

## MPU:

- For protection (control who can access the data)
- Memory address section: 
	- Base and limit to describe, with each attribute to notate, which is for OS permission access.

## MMU

- MMU take the base address and look up for the TLB table to covert logical address to physical address. Offset address remain unchanged.
- Do not solve external fragmentation, TLB has limited entry

## TLB-MISS

- Hardware solution:
	- TLB cannot find -> go to page table( 2nd layer), if find then convert. Find an address to replace at first layer, and re-excute.
- Software solution:
	- Could find the reason why TLB miss and the address.
	- paged page table, hash mapping
	- loading multiple TLB entries.
## Paging

- For MMU to better manage the resource, we allocate 4kb as a page, which is a the base unite of memory, and it resolve the external fragmentation. Allow none contiguous memory.
- We use 2 layer of paging table, 1024* 1024 entries 1M, at most 4GB
- 10+10 bits(2 layer pte) +12bits(offset)


## belady anomaly

- occur in page replacement algorithms where increasing the number of page frames can result in more page faults.

## Why 64-bit can resolve the issue of 2gb limit?

- 2^32 is 4 gigabyte, file offset support->2GB
- PCIE+DRAM way less than 2^64

## MMU and addr decoder

What is the relationship between MMU & decoder?
- MMU: virtual-address (logical address) -> physical address
- Address decoder: physical address to I/O or DRAM?

## Why Virtual memory?

- If TLB miss, but page in DRAM, just write the mapping to tlb
- if not then page fault
- By utillizing MMU, let memory mapping more flexible
	- shared memory
	- map file to process memory space
	- map i/o to process memory space
- Hardware table
- Software table
- OS ISR: (noting reason of cause, IP, address)

## Page Fault

- We haven't load the page to DRAM

## Shared page
- Example:
	- Multiple app share same library
	- mmap
- Allow different process page table entry to point to same data page
	- shared page have faster context switch
	- ASLR speedup?

## Demand paging

- We would load the data from memory if page fault.
	- Pros: Less I/O, memory, faster response, more programs
	- Cons: Introduce large random access, latency
- Page table entry: attribute: right most bit: valid: invalid
- event: instruction that cause page fault:
	- Query process mm_struct
		- if valid: find a empty frame, modify the page table, invalid-> valid
		- if not valid: program error: e.g. pointer is wrong.
- Pure demand paging:
	- When linux execute (execve() ) new executable, wouldn't allocate any page to task.
## Copy on write

- Parent and child process share frames together, when content has different, it would have copy frame
	- Copy parent mm_struct to child
	- set parent and child page table to read-only
	- if modified, generate page fault
		- copy parent and child, and attribute become read-write-able
- for faster generating fork()
- read/write attribute
## Zeroed out page

- Any page that mapped to process should be init to 0
	- e.g. mmap, stack, heap(sbrk, brk)

## Same-page merging

- KVM has multiple page from different virtual machine
- desktop and lockscreen has same page
- no exactly sharing

## Page Replacement

- FIFO (don't quite work)
- Optimal: remove the task that most future didn't use
- LRU: remove the most recent that does not use
- LFU: remove the least frequency 
- MFU: remove the most frequency (does not work)

- Linux:
	- Active to inactive: Second place
	- If new page enter system

## Thrashing:

- If a process don't have enough frames, making page fault keep happening,
- CPU would keep waiting I/O, causing CPU utilization drop.

## Working-Set

- We calculate how many pages get reference in a period of time
- Demand > available frame: thrashing
	- suspend and free that
- Demand << available frame:
	- Increase the degree of multi-programming

## Block storage

- minimum read/write unit: page
- minimum erase unit: block
- logic block address to define a block

## Out-of-place:

creating new copy of data at different location instead of modify the original data.

## I-Node

- A index tree that has multiple layers
- direct:12 data block
- single: 256, double 256^2, triple 256^3 data block
- at most 5 disk acess

## RAID

- Raid 0: multiple disk as a disk, 
	- m disk has m speed up access speed for speed up, not great for ssd
- Raid 1: mirror the same data in different disk
	- read speed up
	- write slow down
- Raid 4: disk 0,1,2 xor to disk 3, as parity disk
- Raid 5: same as Raid 4 but parity distribute to all disks
- Raid 6: same as Raid 5 but has double parity
- Raid 10: do raid 1 and then raid 0
- Raid 50: do raid 5 and then raid 0