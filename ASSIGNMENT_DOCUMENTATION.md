# Assignment 3 - Complete Documentation

**Student Name**: [Jana alhwail]  
**Student ID**: [445052122]  
**Date Submitted**: [Submission Date]

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

### Entry 1 - [April 30, 2026, 7:30 PM]
**What I implemented**: 
I began by going over the assignment guidelines and the SchedulerSimulationSync.java beginning code file. I replaced the default value of the studentID variable with my real student ID. Additionally, I looked over the shared resources in the SharedResources class, paying particular attention to contextSwitchCount, completedProcessCount, totalWaitingTime, and executionLog.

**Challenges encountered**: 
Understanding why these variables are regarded as shared resources and why they can result in race conditions in a multithreaded program was the primary issue.

**How I solved it**: 
I went over the synchronization ideas from Operating System Concepts, paying particular attention to locks, semaphores, mutual exclusion, race conditions, and critical sections. I found the sections of the code where shared data may be updated by many threads.

**Testing approach**: 
To comprehend the final synchronization statistics, the process execution order, and the output format, I ran the original application.

**Time spent**: 
About 1 hour.

---

### Entry 2 - [May 1, 2026, 4:00 PM]
**What I implemented**: 
I included the necessary Java imports for Semaphore and ReentrantLock. Next, I added logLock to safeguard the executionLog ArrayList, counterLock to safeguard the shared counters, and cpuSemaphore with a single permit to manage CPU access.

**Challenges encountered**: 
Deciding where to declare the locks and semaphore such that all methods in SharedResources and Process could safely access them was the primary challenge.

**How I solved it**:
Within SharedResources, I made the locks and semaphore public static final variables. This allowed them to maintain the synchronization code organized in a single class while sharing synchronization techniques for all threads. 

**Testing approach**: 
After adding the imports and synchronization objects, I saved the code and verified that Visual Studio Code displayed no syntax problems.

**Time spent**: 
About 1.5 hours.

---

### Entry 3 - [May 1, 2026, 8:30 PM]
**What I implemented**: 
I used counterLock to safeguard the shared counter variables. I changed incrementContextSwitch, incrementCompletedProcess, and addWaitingTime so that lock, try, finally, and unlock surround each critical section.

**Challenges encountered**:
Ensuring that every lock is always released, even in the event of an exception during execution, was the primary problem. 

**How I solved it**: 
Try-finally blocks were what I used. The try block updates the shared counter, and the finally block calls counterLock.unlock.

**Testing approach**: 
To ensure that there was only one lock call and one corresponding unlock call, I went over each method. Additionally, I verified that the original counter update algorithm remained unaltered.

**Time spent**: 
About 1 hour.

---

### Entry 4 - [May 2, 2026, 1:00 PM]
**What I implemented**: 
I used logLock to safeguard the executionLog ArrayList. In order to perform executionLog.add(message) inside a protected critical area, I modified the logExecution method.

**Challenges encountered**: 
Understanding that ArrayList is not thread-safe and that multiple threads writing to it simultaneously could result in inconsistent log entries or ConcurrentModificationException was the primary challenge.

**How I solved it**: 
For the execution log, I employed a different ReentrantLock known as logLock. This maintains the design's clarity and distinguishes log protection from counter protection.

**Testing approach**: 
I executed the application and verified that there were no exceptions in the Execution Log Summary at the conclusion.

**Time spent**: 
About 1 hour.

---

### Entry 5 - [May 2, 2026, 2:00 PM]
**What I implemented**: 
The run and runToCompletion methods now have semaphore synchronization. Prior to execution, the process obtains the cpuSemaphore, which it releases in the finally block.

**Challenges encountered**: 
Making sure the semaphore was always released was the primary problem in avoiding deadlock. Managing situations when acquire might be interrupted before the semaphore is obtained presented another difficulty.

**How I solved it**: 
To monitor if the semaphore was properly acquired, I utilized a boolean variable called cpuAcquired. Then, only if cpuAcquired is true did I release the semaphore.

**Testing approach**: 
I ran the program multiple times and verified that all processes completed successfully, the completed process count matched the number of processes, and no deadlock or ConcurrentModificationException occurred.

**Time spent**: 
About 1.5 hours.

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

The first race condition occurs in the shared counter variables such as contextSwitchCount and completedProcessCount. Multiple threads update these variables, and non-atomic operations like incrementing a counter may result in lost updates and incorrect outcomes.

The second race condition occurs in executionLog, which is an ArrayList shared by all threads. Since ArrayList is not thread-safe, concurrent additions may lead to inconsistent data or runtime errors such as ConcurrentModificationException.

To address these problems, I used ReentrantLock to protect these critical sections. The counterLock ensures safe updates of the counter variables, while the logLock protects the executionLog from concurrent modification.

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

ReentrantLock provides mutual exclusion, allowing only one thread to access a critical section at a time. I used ReentrantLock in my code to protect shared resources such as the counter variables and the executionLog.

A Semaphore is used to control access to a limited resource using permits. In my implementation, I used a binary semaphore with a single permit to represent the CPU, allowing only one process to execute at a time.

ReentrantLock was used to protect shared data, while Semaphore was used to control access to the CPU resource.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

Deadlock is a situation where two or more threads are holding a resource and waiting for another resource that will never be released.

One prevention technique is to always release locks in a finally block. In my code, I used try-finally to ensure that each ReentrantLock is always released, even if an exception occurs.

Another technique is to keep critical sections as short as possible and avoid holding resources for a long time. In my implementation, I only locked the necessary operations and kept the protected sections minimal.

Additionally, I used a cpuAcquired flag when working with the semaphore to ensure that the CPU permit is only released if it was successfully acquired, which helps prevent incorrect resource handling.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

To protect all three counter variables, I used a single ReentrantLock called counterLock, which is a coarse-grained locking approach. I chose this design because the counter operations are simple and short. This makes the implementation easier to understand and less prone to synchronization errors. The main advantage of coarse-grained locking is simplicity and easier maintenance. However, it may reduce concurrency because only one thread can update any counter at a time. Fine-grained locking uses a separate lock for each counter, allowing higher concurrency since the counters are independent. For this assignment, I chose coarse-grained locking because it is sufficient and provides a clear and safe solution.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 

**Why they need protection**: 

**Synchronization mechanism used**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Justification**: 

---

### Critical Section #2: Execution Log

**What resource**: 

**Why it needs protection**: 

**Synchronization mechanism used**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Justification**: 

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 

**Number of permits and why**: 

**Where implemented**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Effect on program behavior**: 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
(Show that running multiple times produces consistent, correct results)

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

**Conclusion**: 

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 

**Results**: 

**What this proves**: 

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 

**Actual values**: 

**Analysis**: 

---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 

**Results**: 

**What I learned**: 

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
