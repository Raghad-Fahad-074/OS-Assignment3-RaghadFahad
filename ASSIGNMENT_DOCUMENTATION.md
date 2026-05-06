# Assignment 3 - Complete Documentation

**Student Name**: [Raghad Fahad Alhazani]  
**Student ID**: [445052074]  
**Date Submitted**: [7/5/2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [6/5/2026, 10:25]
**What I implemented**: Initialized the project repository, updated the student ID in the code, and made the initial commit.

**Challenges encountered**: No major challenges; the setup process was straightforward

**How I solved it**: Followed the standard cloning and setup steps using VS Code

**Testing approach**: Compiled and ran the unsynchronized version to observe race conditions, such as inconsistent shared variable values and log outputs.

**Time spent**: 40 minutes

---

### Entry 2 - [6/5/2026, 10:40]
**What I implemented**: Used multiple mutex locks (ReentrantLock) for each shared counter to achieve fine-grained locking

**Challenges encountered**: understanding the difference between coarse-grained locking (single global lock) and fine-grained locking (multiple independent locks).

**How I solved it**: Studied lock granularity concepts and recognized that the counters are independent, so using multiple mutex locks reduces contention and improves concurrency.

**Testing approach**: Ran the program multiple times (10 runs) and verified that all counters produced consistent and correct results

**Time spent**: 1 hour

---

### Entry 3 - [7/5/2026, 12:00]
**What I implemented**:  Applied a mutex lock (ReentrantLock) to protect the shared execution log (ArrayList) from concurrent access

**Challenges encountered**: Initially forgot to release the lock, which could potentially cause a deadlock.

**How I solved it**: Refactored the code to ensure all lock operations are placed inside try-finally blocks so that the lock is always released

**Testing approach**: Increased logging activity and confirmed that no ConcurrentModificationException or inconsistent behavior occurred. 

**Time spent**: 50 minutes

---

### Entry 4 - [7/5/2026, 2:00]
**What I implemented**: Introduced a binary semaphore (1 permit) to control CPU access and enforce mutual exclusion among threads.

**Challenges encountered**: Ensuring that the semaphore is consistently applied in both run() and runToCompletion() methods.

**How I solved it**: Wrapped both methods with acquire() and release() calls inside try-finally blocks to guarantee proper resource management.

**Testing approach**: Temporarily increased the semaphore permits to 2 to observe concurrent execution, then reverted it back to 1 to confirm strict mutual exclusion.

**Time spent**: 1 hour

---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[First race condition – contextSwitchCount++ (and other counters).
Shared resource: integer counters.
Problem: The increment operation (++) is not atomic. Multiple threads may read the same value, increment it, and write it back, resulting in lost updates.
Incorrect behavior: The final counter value may be lower than the actual number of increments.
Second race condition – executionLog.add(message).
Shared resource: ArrayList<String>.
Problem: ArrayList is not thread-safe. Concurrent add() operations can corrupt its internal structure, cause runtime exceptions such as ConcurrentModificationException, or result in lost entries.
Incorrect behavior: The program may crash or some log entries may be missing.]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[ReentrantLock is a mutual exclusion (mutex) mechanism that ensures only one thread can access a critical section at a time. I used it to protect shared resources such as counters and the execution log because these require exclusive access to maintain data consistency.
Semaphore manages a set of permits and allows a limited number of threads to access a resource concurrently. A binary semaphore (1 permit) behaves similarly to a mutex, while a counting semaphore can allow multiple threads. I used Semaphore(1) to control CPU access, ensuring that only one process executes at a time, which simulates a single-core CPU.]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Deadlock occurs when two or more threads are permanently blocked, each waiting for resources held by others.
Prevention techniques used:
Lock ordering (simplified usage): I ensured that no thread holds more than one lock at a time, eliminating the possibility of circular wait.
try-finally blocks: Every lock() or acquire() is paired with a corresponding unlock() or release() inside a finally block. This guarantees that resources are always released, even if an exception occurs.
Additionally, the semaphore is acquired at the beginning of the critical section and released immediately after execution, avoiding nested locking and reducing the risk of deadlock.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[I chose fine-grained locking by using three separate ReentrantLock instances, one for each counter (contextSwitchLock, completedProcessLock, and waitingTimeLock).
Reason: The three counters are independent, meaning updating one does not depend on the others. Using a single coarse-grained lock would cause unnecessary blocking, as threads updating different counters would still have to wait for each other.
Trade-offs: Fine-grained locking increases code complexity and requires careful design, while coarse-grained locking is simpler but reduces concurrency and system throughput.
Since the counters are independent, fine-grained locking provides better concurrency by allowing multiple threads to update different resources simultaneously. This approach follows the principle of protecting each shared resource with its own lock.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**:  `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`

**Why they need protection**:  The read-modify-write operations (increment, addition) are not atomic; 
without locks, updates can be lost.

**Synchronization mechanism used**:  Three separate `ReentrantLock`s (fine-grained)

**Code snippet**:
```java
public static void incrementContextSwitch() {
contextSwitchLock.lock();
try { contextSwitchCount++; } finally { contextSwitchLock.unlock(); }
}
```

**Justification**:  Each counter is independent, so separate locks maximise concurrency.

---

### Critical Section #2: Execution Log

**What resource**: List<String> executionLog

**Why it needs protection**:  ArrayList is not thread‑safe; concurrent add() calls cause
corruption or exceptions.


**Synchronization mechanism used**: ReentrantLock logLock

**Code snippet**:
```java
public static void logExecution(String message) {
logLock.lock();
try { executionLog.add(message); } finally { logLock.unlock(); }
}
```

**Justification**: Exclusive access is required to preserve the logʼs integrity.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: Simulate a single‑core CPU – only one process can execute at a time.

**Number of permits and why**: I used 1 permit (binary semaphore) to simulate a single-core CPU, where only one process can execute at a time. This ensures mutual exclusion, preventing multiple processes from running simultaneously and maintaining correct scheduling behavior

**Where implemented**:  Process.run() and Process.runToCompletion()

**Code snippet**:
```java
SharedResources.cpuSemaphore.acquire();
try {
// ... execution code ...
} finally {
SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**:  Guarantees that even though many threads are ready, only one proceeds into
the CPU at any moment – exactly like a real uniprocessor system.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
 Ran java SchedulerSimulationSync five times.
```

**Results**: 
(Each execution produced consistent and stable results, including identical total counts for context switches and completed processes, as well as consistent waiting time calculations.)

**Why synchronization is necessary**: 
( Without synchronization, race conditions could occur when multiple threads update shared resources such as contextSwitchCount, completedProcessCount, totalWaitingTime, and executionLog. These operations are not atomic and could lead to lost updates or inconsistent values. Therefore, mutex locks are required to ensure thread-safe updates and data consistency.)

**Conclusion**: Synchronization ensures deterministic and reliable results across multiple executions

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: Increased logging activity by executing multiple processes concurrently and repeatedly adding log entries to executionLog.

**Results**: No ConcurrentModificationException or related runtime errors occurred during execution.

**What this proves**: The use of ReentrantLock for the execution log successfully ensures thread-safe access to the shared ArrayList.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: All processes should complete execution exactly once, with accurate updates to context switches and waiting time based on scheduling behavior.

**Actual values**: The final output consistently showed correct totals matching the number of processes and proper accumulation of waiting time and context switches.

**Analysis**: The results confirm that the scheduler logic (Round Robin with synchronization) works correctly and that shared data is properly protected using mutex locks and semaphore control.

---

### Test 4: Different Scenarios
**Scenario tested**: [ Running the system with different numbers of processes and varying time quantum values.]

**Purpose**: To evaluate system behavior under different workload conditions and verify scheduler stability

**Results**: The system maintained correct execution behavior across all scenarios, with processes being scheduled fairly and synchronization remaining stable.

**What I learned**: Changing time quantum and process count affects scheduling behavior and context switch frequency, but proper synchronization ensures correctness is preserved regardless of workload.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[6-8 sentences about key concepts, challenges, insights]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 

**Example 2**: 

---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 

**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 

**What I'm most proud of**: 

---

**End of Documentation**
