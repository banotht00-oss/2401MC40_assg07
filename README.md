# Operating Systems Assignment 7

## Overview

This assignment implements and tests four major Operating System concepts in the xv6 kernel:

1. User Threads
2. Semaphores and Producer-Consumer Synchronization
3. Deadlock Detection and Avoidance
4. Priority Scheduling and Aging

The implementations were compiled and tested in QEMU.

## Q1 - User Threads

Implemented user-level thread creation and management using the `clone()` and `join()` system calls.

### Features
- `clone(fcn, arg, stack)` creates a schedulable thread with a unique PID, kernel stack, trapframe, file references, and parent relationship.
- Threads begin execution at the specified function with the provided stack.
- A userspace return trampoline calls `exit(0)` when the thread function returns.
- `join()` waits for and reaps clone-created children, while `wait()` continues to handle regular process children.
- Orphaned threads are adopted by init.
- Shared physical pages are maintained using reference counts, preventing premature deallocation.
- Each thread has an isolated trapframe and trampoline mapping while sharing ordinary user memory.

### Testing

The `threadtest` program creates three worker threads, each performing 100,000 unsynchronized counter increments.

Example output:

```text
threadtest: counter=102121 expected=300000
```

The counter may vary between runs because concurrent read-modify-write operations can cause lost updates.

## Q2 - Semaphores and Producer-Consumer

Implemented kernel semaphores to provide synchronization between concurrent processes and threads.

### Features
- Ten semaphore slots are maintained by the kernel.
- Each semaphore contains a value, initialization state, and spinlock.
- `sem_wait()` blocks when the semaphore value is zero and decrements it when available.
- `sem_signal()` increments the value and wakes waiting processes.
- Invalid IDs, negative initial values, and use before initialization return errors.
- `sem_down()` and `sem_up()` remain available as compatibility aliases.

### Producer-Consumer Problem

The `synctest` program demonstrates:
- Mutual exclusion for shared-counter increments.
- A five-slot circular buffer.
- The `empty`, `full`, and `mutex` semaphores.
- Synchronization between producer and consumer processes.

Example output:

```text
synctest: counter=300000 expected=300000
producer: 1
consumer: 1
...
producer: 20
consumer: 20
synctest: producer-consumer complete
```

The semaphores prevent the producer from writing to a full buffer and the consumer from reading from an empty buffer.

## Q3 - Deadlock Detection and Avoidance

Implemented deadlock detection and the Banker's algorithm using resource-allocation data structures.

### Features
- Fixed-size allocation, request, and available-resource arrays.
- Deadlock detection by repeatedly identifying processes whose outstanding requests can be satisfied.
- Resource release simulation after process completion.
- Identification of unfinished processes as deadlocked.
- Banker's safety algorithm using `Need = Max - Allocation`.
- Rejection of resource requests that result in an unsafe state.
- A pipe-based deadlock demonstration involving two blocked child processes.
- Cleanup of blocked processes by killing and reaping them after detection.

### Test Cases

| Scenario | Expected Result |
|---|---|
| Two processes hold resources and request each other's resources | P0 and P1 are deadlocked |
| Available resources allow processes to complete sequentially | No deadlock |
| Granting a request makes a safe Banker state unsafe | Request denied |

## Q4 - Priority Scheduling and Aging

Implemented priority-based process scheduling with aging to reduce starvation.

### Features
- Default process priority: `10`
- Highest priority: `1`
- Lowest priority: `20`
- The scheduler selects the runnable process with the smallest priority number.
- A scheduling cursor provides round-robin ordering among equal-priority processes.
- Every 30 ticks spent waiting in the runnable state, aging improves the process priority by one.
- Priority cannot improve beyond `1`.
- `set_priority(pid, p)` changes a process priority.
- `get_priority(pid)` retrieves a process priority.

### Testing

The `priotest` program creates CPU-bound children with different initial priorities and observes their execution and priority changes.

Example output:

```text
aging: lowest-priority child is now priority=20 after 30 ticks
aging: lowest-priority child is now priority=19 after 60 ticks
aging: lowest-priority child is now priority=18 after 90 ticks
child pid=17 finished priority=1 completion_tick=100 runtime=100
child pid=18 finished priority=5 completion_tick=101 runtime=100
child pid=19 finished priority=10 completion_tick=102 runtime=100
child pid=20 finished priority=17 completion_tick=103 runtime=100
priotest: all children completed; aging promotes waiting processes
```

The exact completion ticks and observed values may differ between executions due to QEMU configuration and scheduling interleavings. The test demonstrates priority aging and the completion of lower-priority processes.

## Validation and Testing

- Kernel compilation completed successfully.
- All four user programs compiled and ran in QEMU.
- `threadtest` demonstrated shared-memory behavior and lost updates.
- `synctest` verified semaphore synchronization and producer-consumer operation.
- `deadlock.c` tested deadlock detection and avoidance scenarios.
- `priotest` demonstrated priority scheduling and aging.
- `usertests -q` completed its functional test cases, including lazy-allocation tests.

**Known issue:** The final free-page accounting reported one fewer free page than the initial count. This discrepancy remains documented and should not be interpreted as a completely clean regression-test result.

## Learning Outcomes

- Understanding thread creation, shared memory, and thread lifecycle management.
- Implementing synchronization primitives in the kernel.
- Applying semaphores to solve race conditions and producer-consumer synchronization.
- Understanding deadlock detection and safe-state resource allocation.
- Implementing priority scheduling and aging to reduce starvation.
- Testing kernel-level changes using QEMU.

## Submission Evidence

Terminal screenshots are not embedded in this README. Capture the actual QEMU terminal outputs of `threadtest`, `synctest`, `deadlock.c`, and `priotest` for submission evidence.
