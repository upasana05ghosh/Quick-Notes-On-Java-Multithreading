# Quick-Notes-On-Java-Multithreading
Quick Notes On Java Multithreading

## Index
1. Basic Concepts
2. Few concepts of Operating System
2. Mutex, Semaphore, Monitor
3. Executor Framework in Java
6. Thread Pool
7. Thread Local

## Basic Concepts
1. **Program**: Program is a set of instruction that resides in secondary memory.
2. **Process**: Process is a program in execution.
3. **Thread**: Thread is a basic unit of execution in a process.

**Threads shares** -> code section, data section, files, signals, etc.  
**Threads have its own**-> register sets, stack, program counter, etc.

**Benefits of Threads**:
1. Higher throughput (no. of process executed / time)
2. Responsive application.
3. Effective utilization of memory.

## Few concepts of Operating System

Concurrency vs Parallelism

**Concurrency** ->  Concurrency is the ability to run multiple processes in overlapping timer periods.    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Ex: Single Core Processor. It can still run your multiple application even when it has a single processor.  
**Parallelism**-> Parallelismthe ability to run multiple processes at the same time.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Ex: Multi-core processor
	
**Process Synchronization**
- If two threads are executing a code that has shared variables, then the output of the program should have to be consistence.
- If your program is not synchronized, then it leads to inconsistency, which should be avoided at all cost.

**Race condition**
 - When the output of the process depends on the order of execution

| task t1 | task t2   | task t3  |
|---------|-----------|----------|
| Read(x) | x = x+ 10 | Write(x) |
|         | Write(x)  |          |
			 
  - here *x* is a global variable
  - *t1, t2 and t3* -> all are being executed at the same time.
  - In this case, the final value of *x* depends on the order of execution of *t1, t2 and t3*.
  
*Critical Section*: Critical Section is the part of the program that consists of shared variables/resources.

Condition for Synchronization: 
1. Mutual Exclusion -> At a time, only a single thread should be executing inside the critical section.
2. Progress -> If a process/thread wants to enter the critical section and critical section is free/available, then it should be 
			   able to enter inside the Critical section.
			   The decision to enter the critical section should be taken by those who are waiting outside.
3. Bounded Wait -> There should be a bound on the number of times a process is entering the critical section.


* If no bouned wait -> then that process might starve
* If no progress ->  then deadlock will occur. 

Checkout Peterson's Critical Section Algorithm for more info.

## Mutex, Semaphore, Monitor
- These are like a tool to achieve synchronization in a program.

Mutex
- Mutual Exclusion
- Allow only a single thread to enter inside the critical section.
- If a thread has acquired the lock, all other threads trying to acquire the lock will block until the lock is released. 
- Example: A one-way road. At a time, only one can vehicle can pass use it. Multiple vehicles can not pass through it at once. 

Semaphore
- Unlike mutex, it allows a certain number of threads to enter the critical section.
- It can be used for signalling among thread.
- Example: Say a 2-way road. At a time, 2 vehicles can pass through it simultaneously.

Mutex vs Semaphore
- Mutex belongs to a thread, but semaphore does not belong to a thread.
- Binary Semaphore =!  Mutex
   - Mutex is locked an unlocked by the same thread.
   - Semaphore, even if it's binary, can be locked/unclocked by different threads.
   
Monitor
 - In simple terms : mutex + conditional variable
 - Conditional variables are a mechanism to wait/suspend a thread until notified by other thread that some condition is true now. check (https://docs.oracle.com/javase/6/docs/api/java/util/concurrent/locks/Condition.html for more details)
 - Why conditional variables. Mutex is not enough?
    - If we don't use conditional variables then, our CPU will spend a lot of time checking some condition is true or not, which will waste a lot of CPU cycles, also known as spin wait. Therefore, it helps to avoid spin-wait to some extent.

Java Monitor
 - In Java, every object act as a monitor. 
 - Every object has wait() and signal() function to notify other threads. 
 - The mutex is hidden but to lock the mutex implicitly, a synchronized keyword is used.
 - To learn more: http://www.csc.villanova.edu/~mdamian/threads/javamonitors.html
 
Inter-thread communication
- wait() - Used to tell the calling thread to go sleep and give up the locks until someone calls notify.
- notify() - Used to wake up a single thread that is in the wait state.
- notifyAll() - Used to wake up all the threads that are in the wait state.
	

## Executor Framework in Java
- If you are working on some large project where there are many threads, then its difficult to manage and maintain them all.
- Executors are used to manage and maintain threads.
- They are based on the producer-consumer pattern.
- The task that is produced is consumed by the threads managed by Executors.

- Three interfaces to manage thread
1. Executor Interface.
2. ExecutorService Interface
3. ScheduledExecutorService

ThreadPool
- consist of multiple worker threads.
- creates a limited number of thread.
- Re-use worker threads whenever they are available instead of creating new threads.

Example of Executors using thread pools

		//creating a thread pool of 5 worker threads using ExecutorService
		ExecutorService executorService = Executors.newFixedThreadPool(5);

		//Calling the task 10 times. Since, we have only 5 worker threads, 
		//so these 5 will be used only, once they are available
        for (int i = 0; i< 10; i++ ){
        executorService.execute(() -> {
            System.out.println("Inside executor" + Thread.currentThread().getName());
        });
        }

		// to free up system resources and to allow graceful application shutdown. 
		//Submitted tasks are still executed, but no new tasks will be accepted.
        executorService.shutdown();
		
		
        while (!executorService.isTerminated()) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("program terminated");
		

Executor Lifecycle
1. Running
2. Shutting Down
3. Terminated

Thread Local
- Allow us to store/fetch data specific to a thread.
- But, be careful, when using them in thread pools as they might mess up the things.


int count = 0;
void sum(int n) {
	count  = count + n;
	return count;
}

This code is not thread-safe. Since the count is a global variable and will not hold the initial value for all the threads.
Instead, we can use thread-local to save the value of the count variable

    ThreadLocal<Integer> count = ThreadLocal.withInitial(() -> 0);

    void sum(int n) {
        counter.set(counter.get() + n);
    }

