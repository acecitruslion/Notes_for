# Operating Systems (OS) — Interview Notes

Format: short definition + example. Keep answers to 1-2 lines, expand only if asked.

---

## ⭐⭐⭐ Must Know

### 1. Process vs Program
- **Program** – passive set of instructions stored on disk (e.g., a `.exe` file).
- **Process** – active instance of a program in execution, has its own memory, PCB (Process Control Block), state.

### 2. Process States

- <img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/1b44b592-ec50-4d7a-809f-5a21b13a967f" />

- **Ready** – waiting for CPU.
- **Running** – currently executing.
- **Waiting** – blocked on I/O or an event.



### 3. Process vs Thread
| Process | Thread |
|---|---|
| Independent, has own memory space | Lightweight, shares memory with other threads of same process |
| Heavyweight (more overhead) | Lightweight (less overhead) |
| Communication needs IPC | Can communicate directly via shared memory |
| Example: opening two separate apps | Example: a browser's tabs/rendering + downloading within one app |

### 4. Multithreading
Multiple threads run within a single process, sharing memory but executing independently — improves performance/responsiveness (e.g., one thread handles UI, another handles a background download).

### 5. Context Switching
Saving the state (registers, PC, etc.) of a currently running process/thread and loading the state of another so the CPU can switch between them. Pure overhead — no useful work is done during the switch itself.

### 6. CPU Scheduling
Decides which process in the **Ready queue** gets the CPU next.
- **FCFS (First Come First Serve)** – executes in arrival order; simple but can cause long wait (convoy effect).
- **SJF (Shortest Job First)** – picks the process with smallest burst time; minimizes average wait time but needs to know burst time in advance, can starve long jobs.
- **Round Robin** – each process gets a fixed time slice (quantum), then moves to back of queue; fair, good for time-sharing systems.
- **Priority Scheduling** – process with highest priority runs first; can cause starvation of low-priority processes (fixed via **aging**).

### 7. Deadlock
A situation where two or more processes are stuck waiting for each other's resources forever, and none can proceed.

### 8. Deadlock — 4 Necessary Conditions (Coffman Conditions)
All 4 must hold simultaneously for deadlock:
1. **Mutual Exclusion** – resource held by only one process at a time.
2. **Hold and Wait** – process holds a resource while waiting for another.
3. **No Preemption** – resource can't be forcibly taken away.
4. **Circular Wait** – a cycle of processes each waiting on the next.

### 9. Deadlock Handling
- **Prevention** – design the system so at least one of the 4 conditions can never hold (e.g., request all resources at once, impose resource ordering).
- **Avoidance** – allow requests only if the resulting state is still "safe" (e.g., **Banker's Algorithm**).
- **Detection & Recovery** – let deadlock happen, detect it (wait-for graph / resource allocation graph), then recover by killing/rolling back a process (choosing a "victim").
- **Ignore (Ostrich Algorithm)** – pretend deadlock doesn't happen; just reboot/restart if it does. Sounds silly, but this is what most general-purpose OSes (Windows, Linux) actually do, since deadlocks are rare in practice and prevention/avoidance overhead isn't worth it for every process.

### 10. Semaphore
An integer variable used for process synchronization, accessed only via **wait()** (P) and **signal()** (V) operations.
- **Binary Semaphore** – value 0 or 1, works like a mutex.
- **Counting Semaphore** – value can range over multiple resources, e.g., controlling access to a pool of N identical resources.

### 11. Mutex
A locking mechanism that allows **only one thread** to access a critical section/resource at a time. Ownership-based — only the thread that locked it can unlock it.
- **Semaphore vs Mutex** – Mutex is for mutual exclusion (locking) only, owned by the locking thread; Semaphore can allow multiple threads (counting) and has no ownership — any thread can signal it.

### 12. Process Synchronization
The mechanism of coordinating the execution of multiple processes/threads that share resources, so they don't interfere with each other and produce correct, predictable results.
- **Why needed:** without it, concurrent access to shared data causes race conditions and inconsistent results.
- **Tools used:** locks, mutex, semaphores, monitors.
- **Two broad problem types:** processes competing for a shared resource (mutual exclusion) and processes needing to coordinate execution order (e.g., producer must produce before consumer consumes).

### 13. Critical Section
The part of the code where a process accesses shared resources (shared variable, file, etc.) — must be executed by only one process at a time to avoid conflicts.

**Critical Section Problem — a solution must satisfy 3 conditions:**
1. **Mutual Exclusion** – only one process can execute in the critical section at a time.
2. **Progress** – if no process is in the critical section, one of the processes waiting to enter must be allowed in without indefinite delay (no unnecessary blocking).
3. **Bounded Waiting** – there must be a limit on how many times other processes can enter the critical section before a waiting process gets its turn (prevents starvation).

**Structure of a process using a critical section:**
```
Entry Section     → request permission to enter (lock/wait)
Critical Section  → access shared resource
Exit Section      → release permission (unlock/signal)
Remainder Section → rest of the code
```

**Solutions:** Peterson's Algorithm (software, 2 processes, uses turn + flag variables — mostly theoretical/academic), and practical ones: locks, mutex, semaphores.

### 14. Race Condition
When two or more processes/threads access and modify shared data **concurrently**, and the final result depends on the timing/order of execution — leads to incorrect/unpredictable results. Fixed using synchronization (locks, semaphores).

**Example:** Two threads incrementing the same bank balance variable at the same time can lose one of the updates if not synchronized.

### 15. Classic Synchronization Problems
- **Producer-Consumer** – producer adds items to a shared buffer, consumer removes them; need synchronization so producer doesn't add to a full buffer and consumer doesn't remove from an empty one. Solved using semaphores (`full`, `empty`, `mutex`).
- **Reader-Writer** – multiple readers can read shared data simultaneously, but a writer needs **exclusive access** (no other reader/writer at the same time). Goal: allow concurrency for reads while protecting writes.
- **Dining Philosophers** – 5 philosophers sit around a table with 5 forks (one between each pair); each needs **both** adjacent forks to eat. Illustrates deadlock/starvation risk if everyone picks up their left fork at once and waits forever for the right one.
  - **Fix idea:** allow at most 4 philosophers to sit at once, or make one philosopher pick up forks in the opposite order (breaks circular wait).
- **Sleeping Barber** – a barber sleeps when no customers; a customer wakes him if he's asleep, or waits in a limited-seat waiting room if he's busy, or leaves if the waiting room is full. Illustrates synchronization between a limited-capacity resource (barber) and multiple arriving clients using semaphores.

---

## ⭐⭐ Important

### 16. Types of Memory Management
- **Contiguous Allocation** – each process gets one single continuous block of memory.
  - **Fixed (Static) Partitioning** – memory divided into fixed-size partitions upfront; simple, but causes internal fragmentation.
  - **Variable (Dynamic) Partitioning** – partitions sized exactly to each process's need; avoids internal fragmentation, but causes external fragmentation over time.
- **Non-Contiguous Allocation** – a process's memory can be scattered across different locations.
  - **Paging** – fixed-size blocks (pages/frames).
  - **Segmentation** – variable-size logical blocks (segments).
- **Swapping** – temporarily moving an entire process out of RAM to disk and back, to free up memory for other processes.
- **Overlays** – (older technique) load only the part of a program needed at a given time into memory, swap in other parts as needed — used when a program is larger than available memory.

### 17. Paging (in detail)
Memory management scheme that divides process memory into fixed-size **pages** (logical) and physical memory into same-size **frames** — avoids external fragmentation, allows non-contiguous allocation.
- **Page Table** – per-process table that maps each page number → frame number in physical memory.
- **Logical Address** = `Page Number + Offset`. The Page Number is looked up in the page table to get the Frame Number; Offset stays the same → gives the physical address.
- **TLB (Translation Lookaside Buffer)** – a small, fast cache of recent page table lookups, sitting inside the CPU/MMU — avoids hitting the (slower) page table in memory on every access. TLB hit = fast; TLB miss = fall back to the page table.
- **Multi-level Paging** – the page table itself is paged (a "page table of page tables") — used because a single-level page table for a large address space would itself be too big to fit in memory.
- **Page size trade-off** – smaller pages = less internal fragmentation but a bigger page table; larger pages = smaller page table but more wasted space per page (more internal fragmentation).

### 18. Fragmentation (with examples)
- **Internal Fragmentation** – allocated block is larger than what's needed, wasting space *inside* the block.
  - Example: process needs 18 KB, allocated a fixed 20 KB page/partition → 2 KB wasted inside the block. Common in **paging / fixed partitioning**.
- **External Fragmentation** – free memory is scattered in small chunks between allocated blocks; total free space may be enough, but not contiguous enough for a new request.
  - Example: 10 KB free between process A and B, another 10 KB free elsewhere — a new 15 KB process can't fit into either even though 20 KB is free in total. Common in **segmentation / variable partitioning**.
  - **Fix:** Compaction — shifting processes together to consolidate free memory into one block (costly, pauses processes).

### 19. Virtual Memory
Technique that lets a program use more memory than physically available RAM, by using disk space (swap) as an extension — gives each process the illusion of a large, contiguous address space.
- **How it works:** only the actively-used pages of a process are kept in RAM; the rest stay on disk (in the swap space) until needed.
- **Page Fault** – occurs when a process accesses a page that isn't currently in RAM; OS pauses the process, loads the required page from disk into a free frame (or evicts one using a page replacement algorithm), then resumes.
- **Benefit:** lets more processes run than physical RAM would normally allow, and lets programs be larger than RAM.
- **Cost:** page faults are slow (disk I/O) — too many of them leads to thrashing (below).

### 20. Thrashing
When the CPU spends more time swapping pages in/out of memory (handling page faults) than doing actual useful work — happens when too many processes compete for too little RAM (degree of multiprogramming too high).

**CPU utilization vs Degree of Multiprogramming:**
```
CPU
Utilization
   │
   │            ____
   │          /       \
   │        /            \        ← thrashing region
   │      /                 \___
   │    /                        \___
   │  /                               \___
   │/_______________________________________\___
   └───────────────────────────────────────────── Degree of
                    ↑                              Multiprogramming
              optimal point
      (adding more processes beyond
       this point crashes performance)
```
- Initially, adding more processes improves CPU utilization (more work available).
- Beyond an optimal point, adding more processes forces excessive page swapping → CPU utilization **drops sharply** — this is thrashing.
- **Fix:** reduce degree of multiprogramming (suspend some processes), use a **working set model**, or add more RAM.

### 21. Synchronization
Coordinating access of multiple processes/threads to shared resources to avoid race conditions — done via locks, mutexes, semaphores, monitors.

### 22. Starvation vs Deadlock
- **Deadlock** – processes stuck forever, no one can proceed.
- **Starvation** – a process keeps waiting indefinitely because other (higher priority) processes keep getting the resource first, but the system as a whole is still progressing. Fixed via **aging** (gradually increasing waiting process's priority).

### 23. IPC (Inter-Process Communication)
Mechanisms that let processes communicate/share data:
- **Shared Memory** – processes access a common memory region directly; fast, but needs synchronization.
- **Message Passing** – processes exchange data via messages (send/receive), no shared memory; slower but simpler/safer, works across machines too.

### 24. Banker's Algorithm
A deadlock **avoidance** algorithm — before granting a resource request, it checks if the system would remain in a "safe state" (i.e., there's still some order in which all processes can finish). If not safe, the request is denied/delayed.

### 25. Cache vs RAM vs Virtual Memory
| Cache | RAM | Virtual Memory |
|---|---|---|
| Fastest, smallest, closest to CPU | Main memory, faster than disk | Uses disk space to extend RAM |
| Stores frequently used data | Stores running processes/data | Used when RAM is full |
| Very expensive per byte | Moderately expensive | Cheap (uses disk) |

---

## ⭐ Good to Know

### 26. Demand Paging
Pages are loaded into memory **only when needed** (on a page fault), not all upfront — saves memory, but a page fault causes some delay.

### 27. Segmentation
Memory management scheme that divides a process into variable-sized **logical segments** (code, data, stack) instead of fixed-size pages — matches how a program is logically structured, but can cause external fragmentation.

### 28. Page Replacement Algorithms
- **FIFO** – replace the oldest loaded page.
- **LRU (Least Recently Used)** – replace the page not used for the longest time.
- **Optimal** – replace the page that won't be used for the longest time in the future (theoretical best, used as a benchmark).

**Worked example** — Reference string: `7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2` with **3 frames**:

- **FIFO** – replace whichever page has been in memory longest, regardless of use.
  → Total page faults = **9** (evicts pages purely by arrival order, so it sometimes kicks out a page that's about to be reused — e.g., page `0` gets evicted then immediately needed again).
- **LRU** – replace the page not *used* for the longest time.
  → Total page faults = **10** for this string.
- **Optimal** – replace the page that won't be needed for the longest time in the future (look ahead in the string).
  → Total page faults = **7** (fewest possible — this is why it's the benchmark; not implementable in practice since it needs future knowledge).

*(Exact fault count can vary slightly by textbook convention on whether the first 3 unique pages count as "compulsory" faults — the key point to say in an interview is: **Optimal ≤ LRU ≤ FIFO** in number of page faults, and FIFO can suffer from Belady's Anomaly while LRU/Optimal don't.)*

### 29. Memory Allocation Strategies
- **First Fit** – allocate the first free block big enough.
- **Best Fit** – allocate the smallest free block that's big enough (less wasted space, but slower/more fragmentation over time).
- **Worst Fit** – allocate the largest free block available (leaves a larger leftover chunk).

### 30. System Call vs Library Call
- **System Call** – request to the OS kernel for a service (e.g., file I/O, process creation) — involves a mode switch to kernel mode.
- **Library Call** – a regular function call to a library, executes in user mode, may or may not eventually call a system call internally.

### 31. Kernel Mode vs User Mode
- **Kernel Mode** – privileged mode, full access to hardware/memory, where the OS core runs.
- **User Mode** – restricted mode where normal applications run; must go through system calls to access hardware/kernel services. This separation protects the system from crashing/misbehaving apps.

### 32. Monolithic vs Microkernel
| Monolithic Kernel | Microkernel |
|---|---|
| Entire OS (drivers, file system, etc.) runs in kernel space | Only core services (IPC, scheduling, basic memory mgmt) run in kernel; rest run in user space |
| Faster (less communication overhead) | More modular, more stable/secure (a driver crash doesn't crash the OS) |
| Example: Linux | Example: QNX, Minix |

### 33. Belady's Anomaly
A counter-intuitive case (seen with FIFO page replacement) where **increasing** the number of page frames leads to **more** page faults, not fewer — one reason FIFO is considered a weaker algorithm compared to LRU/Optimal (which don't suffer from this).

### 34. Disk Management & Disk Scheduling
The OS manages how data is read from/written to disk, and **disk scheduling** decides the order in which pending disk I/O requests (read/write at various track/cylinder positions) are served — goal: minimize seek time (time for the disk head to move).

**Example setup:** Disk has 200 cylinders (0–199), head currently at cylinder **53**, pending requests (cylinder queue): `98, 183, 37, 122, 14, 124, 65, 67`.

- **FCFS (First Come First Serve)** – serve requests in arrival order.
  → Head moves: 53→98→183→37→122→14→124→65→67. Simple but lots of back-and-forth seeking = high total seek time (here, **640** cylinders moved).
- **SSTF (Shortest Seek Time First)** – always serve the request **closest** to current head position next.
  → Reduces seek time a lot vs FCFS, but can cause **starvation** of far-away requests if near requests keep arriving.
- **SCAN ("Elevator" algorithm)** – head moves in one direction serving all requests along the way, reaches the end, then reverses direction — like an elevator.
  → No starvation, more uniform wait time than SSTF.
- **C-SCAN (Circular SCAN)** – like SCAN, but after reaching one end, head jumps straight back to the **beginning** without serving requests on the return trip, then scans forward again — gives more uniform wait time than plain SCAN (avoids the long wait for requests near the start after a leftward pass).
- **LOOK** – like SCAN, but head only goes as far as the **last request** in that direction (not all the way to the disk's end) before reversing — avoids wasted movement to empty end regions.
- **C-LOOK** – like C-SCAN, but similarly stops at the last request instead of going all the way to the disk end before jumping back.

**One-liner to compare:** "FCFS is simple but inefficient; SSTF is efficient but can starve requests; SCAN/LOOK sweep like an elevator to balance fairness and efficiency; the C- variants avoid re-serving the same region twice in a row for more consistent wait times."

---

### 35. Multiprogramming vs Multitasking vs Multiprocessing vs Multithreading
| Term | Meaning | Example |
|---|---|---|
| **Multiprogramming** | Multiple programs reside in memory at once; CPU switches to another program whenever the current one waits on I/O — goal is to keep CPU busy, not fast response. | Early single-CPU systems running several batch jobs |
| **Multitasking** | Multiple tasks run seemingly at the same time via fast time-sharing/switching, giving the *user* the illusion of parallelism. | Browsing while music plays on your laptop |
| **Multiprocessing** | Multiple CPUs/cores execute multiple processes **truly in parallel** (not just switching). | Modern multi-core servers/laptops |
| **Multithreading** | Multiple threads *within a single process* run concurrently, sharing that process's memory. | A browser tab rendering the page while another thread downloads a file |

**One-liner:** "Multiprogramming = keep CPU busy across jobs; Multitasking = user-facing concurrency via switching; Multiprocessing = real parallelism with multiple CPUs; Multithreading = concurrency within one process."

### 36. Zombie Process vs Orphan Process
- **Zombie Process** – a process that has **finished execution** but its entry still remains in the process table because the parent hasn't yet read its exit status (via `wait()`). It's "dead" but not fully cleaned up — takes up a PID slot but no other resources.
  - **Fix:** parent calls `wait()`/`waitpid()` to read the exit status, which removes the zombie entry.
- **Orphan Process** – a process whose **parent has terminated** while it's still running. It gets adopted by the `init`/root process (PID 1), which will `wait()` on it when it finishes — so orphans don't become zombies.

**One-liner:** "Zombie = child is dead, parent hasn't collected the exit status yet. Orphan = parent is dead, child is still alive and gets adopted by init."

### 37. Aging
A technique to prevent **starvation** in priority-based scheduling — the longer a process waits in the ready queue, the more its priority is gradually increased, until it's guaranteed to eventually get the CPU.

**Example:** A low-priority process waiting too long has its priority bumped up every fixed interval (e.g., +1 every 10 sec) until it becomes high-priority enough to run.

---

## 38. Quick Recap — Comparisons Table

| Comparison | Key Difference |
|---|---|
| Process vs Thread | Process = independent, own memory; Thread = lightweight, shares memory |
| Program vs Process | Program = passive code on disk; Process = active execution instance |
| Mutex vs Semaphore | Mutex = locking, binary, ownership-based; Semaphore = signaling, can count, no ownership |
| Deadlock vs Starvation | Deadlock = circular wait, no progress at all; Starvation = indefinite wait, system still progresses |
| Paging vs Segmentation | Paging = fixed-size, no external frag; Segmentation = variable-size, logical division, no internal frag |
| Internal vs External Fragmentation | Internal = waste inside a block; External = waste scattered between blocks |
| Kernel Mode vs User Mode | Kernel = full hardware access; User = restricted, goes through system calls |
| System Call vs Library Call | System Call = kernel-mode OS service request; Library Call = user-mode function call |
| Monolithic vs Microkernel | Monolithic = everything in kernel (fast); Microkernel = minimal kernel, rest in user space (stable) |
| FIFO vs LRU vs Optimal | Optimal ≤ LRU ≤ FIFO in page faults; only FIFO suffers Belady's Anomaly |
| Prevention vs Avoidance vs Detection vs Ignore | Prevention = design out a Coffman condition; Avoidance = check safe state (Banker's); Detection = find + recover after the fact; Ignore = don't bother (Ostrich) |
| Shared Memory vs Message Passing | Shared Memory = fast, direct, needs sync; Message Passing = slower, safer, works across machines |
| Multiprogramming vs Multitasking | Multiprogramming = maximize CPU use across jobs; Multitasking = user-facing concurrent execution |
| Multitasking vs Multiprocessing | Multitasking = illusion of parallelism via switching (1 CPU); Multiprocessing = real parallelism (multiple CPUs) |
| Zombie vs Orphan Process | Zombie = child done, parent hasn't collected exit status; Orphan = parent gone, child adopted by init |
| SCAN vs LOOK | SCAN goes all the way to the disk end before reversing; LOOK stops at the last request and reverses early |

---

### Quick Interview Tip
For scheduling/synchronization problems (Producer-Consumer, Dining Philosophers, etc.), just explain the **setup + the problem it demonstrates + how it's solved (semaphores/locks)** — no need to write full code unless asked.
