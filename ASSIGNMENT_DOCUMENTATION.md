# Assignment 3 - Complete Documentation

**Student Name**: [Manar Omer Alhaj]  
**Student ID**: [443052316]  
**Date Submitted**: [7/5/2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: https://drive.google.com/file/d/18ntt8T5fRWs3pPWM1i6OzBvOmxFxhsgv/view?usp=sharing

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

### Entry 1 - [May 3, 2026, 8:00 PM]
**What I implemented**: Set studentID to 443052316 and added a ReentrantLock for the three counter variables.

**Challenges encountered**:Identifying non-atomic operations in shared variables.

**How I solved it**: Wrapped increments in lock() and finally { unlock() }.

**Testing approach**: 

**Time spent**: 45 mins.

---

### Entry 2 - [May 3, 2026, 10:30 PM]
**What I implemented**: Integrated a binary Semaphore in the Process.run() method.

**Challenges encountered**: Handling the InterruptedException from the .acquire() call.

**How I solved it**: Placed the acquisition inside the existing try-catch block.

**Testing approach**: 

**Time spent**: 1 hour.

---

### Entry 3 - [May 4, 2026, 9:15 AM]
**What I implemented**: Added ReentrantLock to protect the executionLog ArrayList.

**Challenges encountered**: Preventing ConcurrentModificationException during high-frequency logging.

**How I solved it**: Secured the .add() method within the logExecution function.

**Testing approach**: 

**Time spent**: 30 mins.

---

### Entry 4 - [May 4, 2026, 11:00 AM]
**What I implemented**: Final testing across different time quanta and verifying statistic consistency.
**Challenges encountered**: Ensuring all finally blocks were correctly implemented to prevent hangs.
**How I solved it**: Conducted code review to confirm every lock() had a matching unlock().

**Testing approach**: 

**Time spent**: 30 mins.

---

### Entry 5 - [May 7, 2026, 2:00 PM]
**What I implemented**: Finalized the ASSIGNMENT_DOCUMENTATION.md and verified the GitHub repository visibility.

**Challenges encountered**: Summarizing technical decisions concisely

**How I solved it**: Focused on the core differences between locks and semaphores.

**Testing approach**: 

**Time spent**: 45 mins.

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
contextSwitchCount++ is non-atomic; concurrent threads can read the same value and overwrite increments. The executionLog ArrayList also risks corruption or crashes if multiple threads add entries simultaneously.
---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
A ReentrantLock ensures mutual exclusion for shared data, while a Semaphore manages access to limited resources. I used a lock in SharedResources to protect the counters and executionLog from concurrent modification. I used a Semaphore(1) in Process.run() to act as a gatekeeper, ensuring only one process "occupies" the CPU at a time. This combination prevents data corruption while simulating single-core execution.
---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

Deadlock is a state where threads wait indefinitely for locks held by each other, causing the program to freeze. Two prevention techniques include Lock Ordering and using Finally Blocks to ensure resources are always freed. I prevented deadlocks by wrapping every lock and semaphore acquisition in a try-finally pattern. This guarantees that unlock() and release() are called even during interruptions, keeping the simulation running smoothly.
---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

For Task 1, I chose a coarse-grained locking strategy by using a single lock to protect all three counters (contextSwitchCount, completedProcessCount, and totalWaitingTime). The primary reason for this choice was simplicity and safety; using one lock reduces the risk of complex deadlocks and is easier to manage in this simulation. The trade-off is that if one thread is updating the context switch count, another thread must wait to update the waiting time, even though they are different variables. While the counters are independent—meaning fine-grained locking (one lock per counter) would provide better concurrency by allowing simultaneous updates—the coarse-grained approach is sufficient for this workload and ensures absolute data integrity with less overhead. In a high-performance system with thousands of threads, the fine-grained approach would be superior because it minimizes thread contention and maximizes parallelism.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount, totalWaitingTime.

**Why they need protection**: These are shared numeric variables. Without protection, simultaneous updates by multiple threads lead to "lost increments" (race conditions).

**Synchronization mechanism used**: ReentrantLock.

**Code snippet**:
```java
lock.lock();
try { contextSwitchCount++; } 
finally { lock.unlock(); }```

**Justification**: The lock ensures atomic "read-modify-write" cycles, guaranteeing that every process completion and context switch is counted accurately.
---

### Critical Section #2: Execution Log

**What resource**: executionLog (ArrayList).

**Why it needs protection**: ArrayList is not thread-safe. Concurrent additions can cause a ConcurrentModificationException or data loss.
**Synchronization mechanism used**: ReentrantLock.

**Code snippet**:
```java
lock.lock();
try { executionLog.add(message); } 
finally { lock.unlock(); }```

**Justification**: This prevents multiple threads from modifying the internal array structure at the same time, ensuring the log stays intact.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To limit concurrent process execution and simulate a single-core CPU environment.

**Number of permits and why**: 1 permit, because it acts as a binary semaphore to ensure only one process thread "uses" the CPU at a time.

**Where implemented**: In the run() and runToCompletion() methods of the Process class.

**Code snippet**:
```java
SharedResources.cpuSemaphore.acquire();
try { /* Execution Logic */ } 
finally { SharedResources.cpuSemaphore.release(); }```

**Effect on program behavior**: It forces process threads to wait for their turn, resulting in a clean, sequential output without overlapping messages.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
Ran java SchedulerSimulationSync 5 times back-to-back.
# Commands used (run the program at least 5 times)
```

**Results**: 
(Show that running multiple times produces consistent, correct results)

**Why synchronization is necessary**: 
Without it, the total switches might vary (e.g., 33 or 34) because two threads could increment the counter at the exact same millisecond.
**Conclusion**: 
The counters are perfectly synchronized.
---

### Test 2: Exception Testing
**What I tested**: Verifying the absence of ConcurrentModificationException.

**Testing procedure**: Verifying the absence of ConcurrentModificationException.

**Results**: 0 crashes or exceptions across all test runs.

**What this proves**: The ReentrantLock successfully protects the executionLog ArrayList.
---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: Switches: 35 | Completed: 16 | Log Entries: 70.

**Actual values**:  Switches: 35 | Completed: 16 | Log Entries: 70.

**Analysis**: The actual values match the expected logic perfectly, confirming thread safety.
---

### Test 4: Different Scenarios
**Scenario tested**:Changing the Time Quantum to 5000ms.
**Purpose**: To see if the synchronization holds under longer thread "sleep" durations.

**Results**: Simulation remained stable; context switches decreased as expected, but stats remained accurate.

**What I learned**:  Synchronization is independent of the scheduling parameters.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:
I learned that shared resources in a multithreaded environment are extremely fragile without proper protection. Race conditions are often "invisible" until you look at the final statistics and see missing data. I now understand that using try-finally is a critical safety habit to prevent deadlocks. Implementing a binary semaphore taught me how to manage a physical resource (CPU) versus just protecting variables. Thread safety is about both data integrity and orderly execution flow.
---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: Banking apps—preventing two ATMs from withdrawing from the same account balance simultaneously.

**Example 2**: Ticket booking—ensuring the same airplane seat isn't sold to two different customers.

---

### How I would explain synchronization to others:
Imagine a single-seat study room (the CPU). Even if 20 students (threads) arrive at once, only one can enter. Synchronization is the "key" to the door. When one student finishes, they pass the key to the next person. Without this key, everyone would try to enter at once, causing a mess and breaking the furniture (data corruption).
---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Manar-Omer/OS-Assignment3-Manar-Omer

**Number of commits**: 4

**Commit messages**: 
1. set my student ID:443052316
2. ReentrantLock to protect counter variables
3. Added Semaphore to control concurrent CPU access
4. Added ReentrantLock to protect execution log

---

## Summary

**Total time spent on assignment**: 3.5 hours.

**Key takeaways**: 
1. Always release locks in finally
2. Semaphores are for resources. 
3. ArrayLists must be locked.

**Most challenging aspect**: 
Handling the InterruptedException syntax in the run() method.
**What I'm most proud of**: 
 Seeing the orderly, glitch-free output once the semaphore was implemented.
---

**End of Documentation**
