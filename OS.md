# Operating Systems — Interview Notes (Infosys L2/L3)

---

## 1. Operating System Basics

**OS** = software layer between hardware and user applications. It manages hardware resources (CPU, memory, disk, I/O) and provides a convenient interface to run programs.

```
 ┌───────────────────────────┐
 │        Applications        │
 ├───────────────────────────┤
 │   Operating System (OS)    │  ← manages resources
 ├───────────────────────────┤
 │         Hardware           │
 └───────────────────────────┘
```

**Functions of OS:** Process management, Memory management, File management, I/O management, Security, Resource allocation, Scheduling.

**Types of OS**

| Type | Key Idea | Example |
|---|---|---|
| Batch | Jobs collected & run in batches, no interaction | Old payroll systems |
| Multiprogramming | Multiple jobs in memory, CPU switches between them when one waits on I/O | Early UNIX |
| Multitasking | Multiple tasks run seemingly at once (time-sliced) | Windows/Linux desktop |
| Multiprocessing | Multiple CPUs work in parallel | Modern servers |
| Time Sharing | CPU time divided into slices among users/processes | Terminal-based UNIX |
| Distributed | Multiple independent computers appear as one system | Google servers |
| Real Time (Hard/Soft) | Strict timing deadlines (Hard = must meet, Soft = best effort) | Hard: Pacemaker, Soft: Video streaming |

---

## 2. Kernel

**Kernel** = the core part of the OS that runs in privileged mode and directly controls hardware (CPU, memory, devices).

| Type | Description | Pros | Cons |
|---|---|---|---|
| Monolithic | Entire OS (drivers, file system, scheduler) runs as one big program in kernel space | Fast (no overhead of message passing) | One bug can crash whole OS; large size |
| Microkernel | Only bare essentials (IPC, scheduling, basic memory) in kernel; rest run as user-space services | More stable, modular, secure | Slower due to inter-process communication |

```
Monolithic:                 Microkernel:
┌───────────────┐           ┌───────────────┐
│ FS │ Drivers   │           │   Core Kernel │
│ Scheduler│ IPC │           └──────┬────────┘
└───────────────┘                  │ IPC
     (all in kernel)     ┌─────────┼─────────┐
                          │ FS      │ Drivers │  (user space services)
                          └─────────┴─────────┘
```

---

## 3. User Mode vs Kernel Mode

| Aspect | User Mode | Kernel Mode |
|---|---|---|
| Privilege | Restricted | Full hardware access |
| Who runs here | Applications | OS core, drivers |
| Access to hardware | Indirect (via system calls) | Direct |
| Crash impact | Only that app crashes | Can crash whole system |

**Mode Switching:** happens when a user program makes a **system call** (e.g., asking to read a file). CPU switches from user mode → kernel mode, executes the privileged operation, then switches back.

```
User Mode  ──system call──▶  Kernel Mode
    ▲                              │
    └───────return───────────────┘
```

---

## 4. System Calls

**System Call** = interface through which a user program requests a service from the OS kernel (since user programs can't directly access hardware).

**Why needed?** To safely and securely let applications use protected resources (files, memory, network) without giving them direct hardware access.

**Common examples**

| Call | Purpose |
|---|---|
| `fork()` | Create a new (child) process |
| `exec()` | Replace current process image with a new program |
| `read()` | Read data from a file/device |
| `write()` | Write data to a file/device |
| `open()` | Open a file |
| `close()` | Close a file |

---

## 5. Process Management

**Program vs Process**

| Program | Process |
|---|---|
| Passive — code stored on disk | Active — program in execution |
| No resources allocated | Has memory, CPU time, resources allocated |

**PCB (Process Control Block):** a data structure the OS keeps for every process, storing:
- Process ID (PID)
- Process state
- Program counter
- CPU registers
- Memory management info
- I/O status info
- Scheduling info

**Process States & Life Cycle**

```
        admit           dispatch
 New ─────────▶ Ready ─────────▶ Running
                  ▲                 │
                  │   preempt/      │ exit
                  │   I/O done      ▼
                  └───────────── Terminated
                                    ▲
                                    │
                              Waiting (I/O wait)
```

Simplified transitions:
```
New → Ready → Running → Terminated
              ↕
           Waiting
```

**Context Switching:** saving the state (PCB) of the currently running process and loading the state of the next process to run, so the CPU can switch between processes.

**Context Switch Overhead:** the CPU does no useful work during a switch — time spent saving/restoring registers, updating PCB, memory maps. Frequent switching increases this overhead.

---

## 6. Threads

**Process vs Thread**

| Process | Thread |
|---|---|
| Heavyweight | Lightweight |
| Has own memory space | Shares memory space with other threads of same process |
| Context switch is costly | Context switch is cheap |
| Independent | Threads of same process can communicate directly |

```
Process
 ┌─────────────────────────────┐
 │  Code | Data | Files (shared)│
 │  ┌────────┐ ┌────────┐       │
 │  │Thread 1│ │Thread 2│  ...  │
 │  │(own stack,        │       │
 │  │ registers, PC)    │       │
 │  └────────┘ └────────┘       │
 └─────────────────────────────┘
```

- **User Threads:** managed by user-level library, kernel unaware of them (fast to create, but if one blocks, whole process may block).
- **Kernel Threads:** managed directly by OS kernel (can run in parallel on multiple cores, scheduled by OS).

**Advantages of Threads:** faster creation/context switch, resource sharing, responsiveness, better CPU utilization on multicore systems.

---

## 7. Multiprogramming vs Multitasking vs Multiprocessing vs Multithreading

| Term | Meaning |
|---|---|
| Multiprogramming | Multiple programs reside in memory; CPU switches to another when one waits for I/O (maximizes CPU use, not necessarily fast response) |
| Multitasking | Multiple tasks run concurrently via time-sharing, giving illusion of parallelism to the user |
| Multiprocessing | Multiple CPUs/cores execute processes truly in parallel |
| Multithreading | Multiple threads within a single process run concurrently |

---

## 8. CPU Scheduling

**Scheduling Algorithms**

| Algorithm | Idea | Preemptive? |
|---|---|---|
| FCFS (First Come First Serve) | Process that arrives first is served first | No |
| SJF (Shortest Job First) | Process with smallest burst time runs first | No |
| SRTF (Shortest Remaining Time First) | Preemptive version of SJF | Yes |
| Round Robin | Each process gets a fixed time quantum in rotation | Yes |
| Priority Scheduling | Highest priority process runs first | Can be either |
| Multilevel Queue | Processes grouped into queues by type/priority, each queue has own scheduling | Depends |

```
Round Robin (time quantum = 2):
Queue: P1 P2 P3
Time:  0  2  4  6  8 ...
       P1 P2 P3 P1 P2 ...
```

**Preemptive vs Non-preemptive**

| Preemptive | Non-preemptive |
|---|---|
| CPU can be taken away from a running process | Process runs till completion or voluntary wait |
| Better responsiveness | Simpler, less overhead |
| e.g. Round Robin, SRTF | e.g. FCFS, SJF |

**Scheduling Criteria**

| Metric | Meaning |
|---|---|
| Turnaround Time | Completion time − Arrival time |
| Waiting Time | Turnaround time − Burst time |
| Response Time | Time from submission to first response |
| Throughput | Number of processes completed per unit time |
| CPU Utilization | % of time CPU is actively working (not idle) |

---

## 9. Synchronization

- **Critical Section:** part of code where shared resources (variables/data) are accessed — only one process/thread should execute it at a time.
- **Race Condition:** occurs when multiple processes/threads access shared data concurrently and the outcome depends on the order of execution — leads to incorrect results.
- **Process Synchronization:** coordinating processes so they don't interfere with each other while accessing shared resources.
- **Producer-Consumer Problem:** a producer generates data and puts it in a shared buffer; a consumer removes data from the buffer. Synchronization is needed so the producer doesn't add to a full buffer and the consumer doesn't remove from an empty one.

```
Producer → [ Shared Buffer ] → Consumer
             (needs locking)
```

---

## 10. Mutex & Semaphore

| Mutex | Semaphore |
|---|---|
| Locking mechanism, binary (locked/unlocked) | Signaling mechanism, can be binary or counting |
| Only the thread that locked it can unlock it | Any process can signal/increment it |
| Used for mutual exclusion only | Used for mutual exclusion AND resource counting |

- **Binary Semaphore:** value is 0 or 1, works like a mutex.
- **Counting Semaphore:** value can range over a set of integers, used to control access to a resource pool with multiple instances (e.g., 5 printers).

```
Semaphore S = 3  (3 resources available)
wait(S): S--     (process takes a resource)
signal(S): S++   (process releases a resource)
```

---

## 11. Deadlock

**Definition:** A situation where two or more processes are blocked forever, each waiting for a resource held by another.

**Necessary Conditions (all four must hold — Coffman conditions):**

| Condition | Meaning |
|---|---|
| Mutual Exclusion | Resource can be held by only one process at a time |
| Hold and Wait | Process holds a resource while waiting for another |
| No Preemption | Resource can't be forcibly taken away |
| Circular Wait | A cycle of processes, each waiting for a resource held by the next |

```
   P1 ──waits for──▶ R2 ──held by──▶ P2
   ▲                                  │
   │                                  ▼
   R1 ◀──held by── P1      P2──waits for──▶ R1
   (circular wait → deadlock)
```

**Handling Deadlocks**

| Strategy | Idea |
|---|---|
| Prevention | Design system so at least one Coffman condition can never hold |
| Avoidance | Grant resources only if system stays in a "safe state" (e.g., Banker's Algorithm — concept only, skip implementation) |
| Detection | Allow deadlock to happen, detect it later (resource allocation graph) |
| Recovery | Kill process(es) or preempt resources to break the deadlock |

**Deadlock vs Starvation**

| Deadlock | Starvation |
|---|---|
| Processes wait forever, none proceed | A process waits indefinitely while others keep getting resources |
| Circular wait involved | No circular wait needed; can be caused by unfair scheduling |

---

## 12. Memory Management

| Logical Address | Physical Address |
|---|---|
| Address generated by CPU (virtual) | Actual address in RAM |
| Used by the program | Used by the memory hardware |
| Translated via MMU | The final destination |

**Memory Allocation**

| Type | Description |
|---|---|
| Fixed Partitioning | Memory divided into fixed-size blocks; each partition holds one process |
| Dynamic Partitioning | Partitions created on-the-fly to exactly fit process size |

**Allocation Algorithms**

| Algorithm | Rule |
|---|---|
| First Fit | Allocate the first free block big enough |
| Best Fit | Allocate the smallest free block that fits (least leftover space) |
| Worst Fit | Allocate the largest free block (leaves biggest leftover) |

```
Free blocks: [10] [4] [20] [8]   Need 6:
First Fit → picks [10]
Best Fit  → picks [8]
Worst Fit → picks [20]
```

---

## 13. Fragmentation

| Internal Fragmentation | External Fragmentation |
|---|---|
| Wasted space *inside* an allocated block (block bigger than needed) | Wasted space *between* allocated blocks (scattered free memory) |
| Happens with fixed-size partitioning | Happens with dynamic partitioning |

```
Internal:                     External:
[Process|wasted] [Process]    [Proc1][free][Proc2][free][Proc3]
     (waste inside block)         (scattered free gaps, none big enough)
```

**Compaction:** rearranging memory contents to combine scattered free blocks into one large block (fixes external fragmentation).

---

## 14. Paging

- **Page:** fixed-size block of a process (logical memory), e.g., 4KB.
- **Frame:** fixed-size block of physical memory, same size as a page.
- **Page Table:** maps each page number to a frame number.
- **Address Translation:** logical address (page number + offset) → physical address (frame number + offset), using the page table.

```
Logical Address: [Page No | Offset]
                       │
                       ▼
                 Page Table
                       │
                       ▼
Physical Address: [Frame No | Offset]
```

Paging removes external fragmentation (since pages/frames are fixed and equal size) but can still cause internal fragmentation (last page may not be full).

---

## 15. Segmentation

**Segmentation:** divides a process into variable-sized logical units (segments) based on program structure — e.g., code segment, data segment, stack segment — rather than fixed-size blocks.

**Paging vs Segmentation**

| Paging | Segmentation |
|---|---|
| Fixed-size blocks | Variable-size blocks |
| Invisible to programmer | Reflects logical program structure |
| No external fragmentation | Can cause external fragmentation |
| Can cause internal fragmentation | No internal fragmentation |

---

## 16. Virtual Memory

**Virtual Memory:** technique that lets a process use more memory than physically available RAM, by using disk space as an extension of RAM.

**Demand Paging:** pages are loaded into memory only when they are actually needed (referenced), not all at once.

**Page Fault:** occurs when a program accesses a page that is not currently in physical memory — OS then fetches it from disk.

```
Process needs Page 5
       │
       ▼
 Is Page 5 in RAM? ── No ──▶ Page Fault ──▶ Load from Disk ──▶ Update Page Table
       │
      Yes
       │
       ▼
   Continue execution
```

---

## 17. Page Replacement

| Algorithm | Idea | Notes |
|---|---|---|
| FIFO | Replace the oldest loaded page | Simple, can suffer Belady's Anomaly |
| LRU (Least Recently Used) | Replace the page not used for the longest time | Performs well, mimics program behavior |
| Optimal | Replace the page that won't be used for the longest time in the future | Theoretical best, not implementable in practice (needs future knowledge) |

**Which performs best?** Optimal (theoretical benchmark) > LRU (close to optimal in practice) > FIFO (weakest, can behave unexpectedly).

---

## 18. Thrashing

**Thrashing:** a state where the CPU spends more time swapping pages in/out of memory than executing actual processes — system throughput collapses.

**Causes:** too many processes competing for too little physical memory; degree of multiprogramming too high; poor page replacement decisions.

**Prevention:** limit degree of multiprogramming, use local (per-process) page replacement, working-set model to allocate frames based on a process's actual active page usage.

```
CPU Utilization
     │        ╭─── peak
     │      ╭─╯
     │    ╭─╯
     │  ╭─╯          ╲___ (thrashing: utilization drops
     │╭─╯                  as more processes are added)
     └───────────────────────────▶ Degree of Multiprogramming
```

---

## 19. Disk Management

| Term | Meaning |
|---|---|
| Seek Time | Time for disk arm to move to the correct track |
| Rotational Latency | Time for the disk to rotate the correct sector under the head |
| Transfer Time | Time to actually transfer the data once positioned |

**Total Disk Access Time ≈ Seek Time + Rotational Latency + Transfer Time**

---

## 20. Disk Scheduling

| Algorithm | Idea |
|---|---|
| FCFS | Serve requests in the order they arrive |
| SSTF (Shortest Seek Time First) | Serve the request closest to current head position |
| SCAN | Head moves in one direction servicing requests, then reverses (like an elevator) |
| C-SCAN | Like SCAN, but after reaching the end, jumps back to the start without servicing on the way back (definition only) |

```
SCAN (elevator):
0 ── 10 ── 20 ── 30 ── ... ── 199 ── (reverse) ── 30 ── 20 ── 10 ── 0
   (services requests while moving in one direction, then turns around)
```

---

## 21. Frequently Asked Comparisons — Quick Reference Table

| Comparison | Key Difference |
|---|---|
| Process vs Thread | Process = independent, own memory; Thread = lightweight, shares memory |
| Program vs Process | Program = passive code on disk; Process = active execution instance |
| Mutex vs Semaphore | Mutex = locking, binary; Semaphore = signaling, can count multiple resources |
| Paging vs Segmentation | Paging = fixed-size, no external frag; Segmentation = variable-size, logical division |
| Internal vs External Fragmentation | Internal = waste inside a block; External = waste between blocks |
| Multiprogramming vs Multitasking | Multiprogramming = maximize CPU use across jobs; Multitasking = user-facing concurrent execution |
| User Mode vs Kernel Mode | User Mode = restricted; Kernel Mode = full hardware access |
| FCFS vs Round Robin | FCFS = order of arrival, non-preemptive; RR = fixed time slice, preemptive, fairer |
| Preemptive vs Non-preemptive | Preemptive = CPU can be taken away; Non-preemptive = runs till completion/block |
| Deadlock vs Starvation | Deadlock = circular wait, no progress; Starvation = indefinite wait, no circular wait needed |

---

## ⭐ Priority Guide for Infosys L2/L3

### Must Prepare (★★★★★)
OS Basics · Process · Thread · Process vs Thread · Context Switching · Scheduling · Mutex · Semaphore · Deadlock · Paging · Virtual Memory · Thrashing · Fragmentation · System Calls · User Mode vs Kernel Mode

### Good to Know (★★★★☆)
Segmentation · Memory Allocation · Disk Scheduling · Types of Kernel

### Can Skip (★★☆☆☆)
Detailed scheduling calculations · Banker's Algorithm implementation · Peterson's Algorithm · Dining Philosophers · Readers-Writers implementation · Belady's Anomaly · TLB internals · NUMA · Huge Pages · Buddy Allocator · Slab Allocator · Demand Segmentation · Copy-on-Write · Swap Cache · Advanced Linux kernel internals

---

### Tip for the interview
Don't recite textbook definitions — explain each concept with a **simple everyday example** (e.g., explain Mutex/Semaphore using a "single washroom key" vs "parking lot with N slots" analogy, or explain paging using a "book divided into equal-sized pages" analogy). Interviewers at this level care more about clarity of understanding than depth of theory.
