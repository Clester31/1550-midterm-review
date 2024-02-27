# Slide set 1 (Introduction to Operating Systems - System Calls and Kernels)

## An OS is a piece of software that...

### 1. Manages Resources

* Resources managed include:
  
  * The CPU's time (spent executing instructions)
  * The Main Memory (RAM)
  * Input/Output Devices (I/O)
  * Security of the system
 
#### The Von Neumann Architecture (AKA "stored program" concept)

* **In Von Neumann, both the code and data share the same memory**
* The Von Neumann architecture describes a computer that:
  * Fetches instructions from RAM to execute
  * Those instructions can further modify other RAM locations (could contain data or code) 

### 2. Abstracts Details

* We give the user the illusion that they have the following:
  * Exclusive acces to the CPU's time
  * Huge amounts of dedicated RAM to hold its code and data
  * Exclusive access to I/O devices
  * Transparent security
* Virutalization is an abstraction technique that gives the user the illusion that the above is true, when in reality, that is not the case.

#### Virtualization Example: Memory

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/74f5fb6d-5569-458f-83e6-1b9e644bdd0a)

* Example: Assume we are in the VN architecture and we want to run three programs at the same time. They must be resident in memory simultaneously
* We use partitioning in order to devided the memory region into pieces, giving a piece to each program
* This leads to two problems however
  * **Protection Problem** - A program can read/write the code or data of another program onto the system
  * **Relocation Problem** - Programs built for one partition might not work when loaded into a different partition
    * Solution: remember the extents of each partition and ensure the program never reads/writes outside of its boundaries
   
    ![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/fe26ef0a-4a9f-4647-a894-4e30ef0fcec6)

* Our program lives inside of an **address space**
* This illustrates the idea of a virtualization: We give the user the belief that they have all of RAM to themselves, they are only focuesd on their program alone, and not whatever else is running in the background

### The adult in the Room

* Since the OS is a piece of software that has the power to manage resources and abstract details, it has the power to say "no" to any requestS
* The OS is an unbypassable layer between a running application and the resources it wants to use.
  * We can only get certain resources with the permission of the OS 

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/15ae35d0-d139-4b0a-8bb8-d7462088221c)

* Why would we not let the user access a certain resources?
  * Lack of storage space
  * That resource could be used by something else at the moment
 
* So how do we ask the OS for resources? Make a **System Call**

#### System Calls

* Syscalls we've used include print(), read(), exti(), etc.
* A syscall is a **software interrupt**
  * We stop what we are currently doing, but we intent to go back later
  * This makes the OS **event-driven**, as it is not constantly running, but is responding to events, then getting out of the way
* Syscalls are not a j-type instruction (jump)
  * We are not giving it an address to jump to
* It's not an i-type instruction either
  * No loads/stores
* It's an r-type instruction (register based)
* How does a program/cmpiler know what system call we want to run?
  * get the value of $v0
  * $v0 does not hold an address because then we could run any address we want in privileged mode
    * Instead every syscall is given an ordinal
    * $v0 is the index into a table of syscalls
* Why not just use the address instead of an ordinal?
  1. We don't want the user to know the location of the kernel mode isntructions
  2. Syscall address might change, you can't change it in your own code every time
  3. The program can't see it because it's not in the program's address space  
   
##### Interrupt Vector

* **Every event that requires the OS to run code is an interrupt**
* When an interrupt occurs, we want to stop the program and set the address in the OS to run the correct code
* When we call on the interrupt vector: we do it as so:
  ```Interrupt Vector[Interrupt Type]```
* Every system call will go to the same address (usually index 0)
  * This goes to another table called the syscall table, which is indexed by $v0
* Interrupt vector is a hardware table. It is in the state of the CPU
* The sycall table is RAM based and can be modified
* Why have two tables?
  * Hardware space is small - this means the interrupt table can be small and the syscall table can be big     

##### Protection Modes

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/cc57da04-2ff8-4267-854b-9d5473759c0e)

* Most CPU's partiton their instruction sets into two different sets:
  * **Privileged/kernel mode isntructions**
    * These are instructions that we only want the OS to be able to do
    * They mostly just set "secret registers" that exist on the processor 
  * **Protected/user mode instructions**
    * The OS is primarily built out of these instructions
    * Any problem can be solved with protected mode instructions. Everyone has access to them 
* The processor will always know what state it is in
* If we try to execute a privleged mode instruction in protected mode, it will not work
  * However, if we try to execute a user mode instruction in privleged mode, it will work just fine
 
* How can we determine wheteher or not we can use privleged mode instructions?
  * there is one bit in the OS that tells us whether we can have access or not - the **Mode Bit**
    * stored in a register
  * Our process will look at the opcode. If it sees that its a privileged mode instruction, we check the mode bit and see if the mode we are in allows us to run that insturction
* What happens when we try to fetch privileged mode instructions in user mode?
  * We end up fetching from an invalid address that isn' ours - the processor will raise an **exception**
    * SIGSEGV
 
#### Process of a system call

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/5b01eb50-dfa4-45f4-a604-39d7291d315e)

1. Elevate the privilege and find the location of the system call code
2. SYSCALL instruction "traps" the CPU into kernel mode
3. Look type of software up in the **interrupt vector**
4. Set $PC to handler of all system calls
5. Save outgoing process state (context switch)
6. Use sysacll number to lookup adress in **syscall table**, run syscall code
7. Return from interrupt

### Event-Driven

* The OS is not actively running all the time. Syscalls are just a type of event (there are more events than syscalls)
* The OS will respond to interrupts, and then get out of the way

### It is the program that runs first on the computer

* The CPU will start in privileged mode, allowing the code to set up the inital addresses of the interrupt handlers in the interrupt vector
* How do we make suer the OS runs first when we start up the computer?
  * The master boot record is the information in the first sector of a hard disk or removable device
    * it tells us how and where the system's operating system is located in order to be booted into the computer's main storage or RAM
    * Usually a menu of operating systems on a boot loader
   
### Overhead

* The OS is overhead: any resource taken by the OS cannot be used by what we really want our system to run

#### Context Switch

* General Idea: Put things back when the OS is done with whatever it's doing
  * Then we must save the information to the stack to save it between funciton calls 
* The instructions of the OS use the same hardware resources (general purpose registers, for example) as the instructions of the user program we interrupted
* We have to put things back the way they were when we return from the OS
* This save/restore of shared resources is known as a **context switch**
* On entry to the OS, save all shared state and restore OS state. On return from OS, do the inverse.
  * Context switches are overhead, because the OS is taking time away from the applications
  *  
* Where is it safe to store these values?
  * The registers are saved in memory. We use registers because they are on the processor and are close to the ALU and actual work 
* Our measure of time: Reduce the number of context switches to improve system performance

## Monolithic Kernel vs Microkernel (Exokernel)

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/54172c16-7905-4bcd-bace-3bf3a4b2c602)

### Monolithic Kernel

* Mono = One, Lithose = Rock -> One big rock
* All three tasks (main procedure, service routines, Utility routines) are found in the kernel
  * We have a linked list insert for our tasks, a scheduling algorithm, and setting up the processor state
* Every part of a task lives within the kernel's address space
* Work must be loaded into RAM as wella s the data we are working with (Satisfied VN architecture)
* All of the work in the monolithic kernel must be bundled into the OS

### Microkernel / Exokernel

* Simply: Have the OS do less/not do some things the monolithic kernel would do
* All unprivleged work is pushed into user-space
  * Large portions of kernel code have nothing to do with the privilege instructions
  * Scheduling code is also moved into the processor 
* Work will usually be more expensive due to more context switches

### Why use one over the other?

#### 1. Performance Overhead (context switches as a unit of time)

* Monolithic kernel has 2 context switches per schedule
* Microkernel/Exokernel has 4 context switches per schedule
  * This means the microkernel is slower than the monolothic kernel
 
#### 2. Application/Build Safety

* In a monolithic kernel, to change the code in the OS, we must recompile the entire thing after changes are made (remember project 2?)
* In the microkenrel however, the modules exist in seperate process in the address space
  * So we can edit, compile, and swap modules without having to reboot

#### 3. Safety/Bugs

* Microkernels have less bugs. Why?
  * Less code than in a monolithic kernel
  * **Attack Surface** - The more code you have, the harder it is to get right (more rprone to vulenerabilities)
    * NOTE: the kernel is one program, the OS is the kernel + auxiliary software (Linux = kernel, GNU = other stuff)

#### 4. Crashes

* If one module crashes in the monolothic kernel, the entier OS crashes
* In a microkernel, the OS will not crash. The user-process server crashes and can be replaced

#### Overall

* Monolithic kernel has better performance due to less context switches
* Microkernel takes longer because of its modularity, but has a small and well defined interface
   * Also one that can be replaced more easily
* Linux is monolithic, Windows is hybrid micro/monolithic kernel

## Virtual Machines  

* A VM is a program that helps you run another program
* QEMU is a piece of software that can be treated as a seperate system
  * Can run a system on top of it, which is a guest operating system
* QEMU is pretending to be a seperate process since its ARM. x86 is not ARM  

### Application Level

* focus on executing individual programs by decoupling them from the underlying operating system

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/26d99418-927f-4418-9e8f-747c6fd97784)

### Process Level

* A virtual platform created for an individual process and destroyed once the process terminates

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/65773fba-3a74-4dbc-9389-157113464236)

### Hypervisor/VMM

* Can see the hardware and allocate it to different systems

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/feb17506-ed5f-40a0-9136-e7b1e4db7a76)

# Slide Set 2 (Scheduling Life Cycle and Threading)

## Scheduling

### CPU Time

* The first resource we are looking to manage is the time spent running insturctions on the CPU on behalf of a running program
* A running program and its associated data is called a **process**
* A program is the code and (static) data typically stored in an executable file on disk
  * The VN architecture prevents us from running it until it has been copied into RAM
  * You have to decide when it has to get in RAM, and then start running instructions

### Multiprogramming

* **Multiprogramming** (AKA Multiprocessing/multitasking) is the ability to run multiple processes at the same time
* The problem is easy if we have N processes and N processors (CPUs)
  * Just pick a process and assign it to a processor
* Pigeonhole principle says if we have more processes than processors, we must assign two or more processes to the same CPU

### Virtulization: Time Slicing (Pseudoparallelism)

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/900d39ae-aae7-472f-b4af-c346be0da8b8)

* If we slice the CPU's time into tiny pieces, no single piece will allow the process to completely do it's work
* If we have 100 processes and one processor, how do we choose which process to run?
  * This is the role of the scheduler
  * We must have a list of all the processes and at each node of the linked list, there is some sort of variable that keeps track of what state the processor is in (created, ready, running, blocked, exited)
  * We then pick which of the ready processes we will run next
* Once the program terminates, we need some sort of event that switched from the running program into the OS
  * exit() system call does just that - It's a signal to the OS that it needs to switch to a different process
  * exit() has no return. It's an event that allows the OS to take control of the CPU
* If we just run our process sequentially instead of (the illusion of) running multiple at the same time, we don't have a multiprogramming system
  * This is a **batch system**. We start a process then run it to completion
 
### Managing multiple processes at once (Juggling)

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/762d55a4-15f3-41fc-ab2e-1d5e51da4992)

* In the batch system, we only have one process going at a time. When we're finished with it, we go to the next process
  * What we need to di is switch between them and realize that most of the time spent is not running one specific process
    * This means that there is time that we are able to do other work while that process is not running
* We must wait until we get I/O data because we cannot run instructions on dat that is not present in RAM (VN)
* read() is an example of a **blocking system call**
  * when we block, we are essentially waiting for input
  * We can only exit a blocked state once the hardware interrupt inditicates we have an I/O completion
  * When the data transfer is complete, then the state must go from blocked to ready
* **Scheduling is choosing among ready processes**
  * When the process gets its data, we can unblock and continue the process
  * We get that through a hardware interrupt which tells us to change the processors state
* What happens while we are waiting for our data to arrive?
  * Run another porces that is ready in the meantime
  * This illustrates the juggling example

#### I/O via interrupts (The Bus)

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/80e6c9fd-9e4e-4e5b-895a-61d1c2850eb9)

* Hardware exchanges information over a shared newtorks communication known as the **bus**
* Each device listens for addresses/commands. It is then responsible for handilng and doing the request
* The messages that the CPU must handler are delieverd as **hardware interrupts**, which behave as much as software ones do
* The CPU is only hearing what it want's, not actively looking for it

## Threading

* A **thread** is a stream of instructions and it's associated state
* Every process has a thread but some processes can have more than one
  * Every process we run has a thread stream
* Imagine we have 3 threads within one process
  * 3 streams of instructions with independent progress
  * The registers are not the same between the three threads, but they have one address space
    * $PC points to different areas on each thread
* If we have a variable in RAM, threads that are part of the same process can access that data if they are part of one process
* Multi-threading is an ease of communication between threads. They can read and write shared address space locations
* Threads are a lightweight process
  * This is because the amount of associated states that we have to go to from thread to thread are smaller, so its cheaper work
 
![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/9805d685-0217-4e5e-ac1a-5a208e93ca9d)

#### How can processes communicate?

* A process can communicate via files
* A pipe is just a file that acts like a queue: we create a network between two processes
  * All of these methods involve the OS. The OS can say no and end communication between dependent processes which is not good
* The OS is not involved with the communication between threads and address space
  * It can't be stopped or managed via the OS
 
#### Benefits of Threading and Parallelism

* More efficient: Sharing of the same address space
* Performance: the cooperation in the case of a word processor
  * Do not have to open and write files relying on the OS
* If the CPU notices something wrong during running, it sends an interrupt, and the OS terminates the process via a kill signal
  * Signals are messages which are asynchronous that are sent to the process
  * This means something like a webserver could crash completely if one thread goes down
    * If they are each their own process however, if one goes down, the rest stay up    

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/48899ef9-4e8c-450c-b931-7ce856f948e8)

### Cooperative vs Competitive

* Threads are cooperative, since they are able to share information between them within a single process
* Processes are competitive since they are unrelated and are competing for the CPU's time
* Assume that multi-threaded process' threads are cooperative
  * Now assume that every thread has to have a seperate stack.
    * Can we have three stacks that have arbitrary size and one heap?
      * No. they will grow into eachother leading to stack space running out quickly   
* We cannot trust programs to vonluntariy yield, exit, or block soon enough to allow for the illusion of simultaneous action
* Consider the infinite loop program
  * ``` HERE: J HERE```
    * This program makes no events (interrupts) by which the OS can regain control
* We need an event that is not software-originated that we can use to have the OS regain control and make scheduling decisions
  * The only thing on the system that is not SW is hardware
  * Add a hardware timer component to do **preemption**
* Why pick one implementation strategy over the other?
  * There is no sharing -> default to processes unless there is explicit sharing
  * Performance -> use threads
 
![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/4c2d8561-c48e-450a-b822-65b45e259bb3)
 
#### Parallelism in a web server

* The WWW is a client-server architecture where the client (a browser) makes requests of a server via HTTP
* HTTP is stateless - The webserver for each reuqest has no memory of the other requests, even from the same client
  * This is why cookies and session ids exist
  * No need for sharing/communication
* It seems that a webserver is parallel then - all client work can be done at the same time
  * Then we could choose the fastest threads over processes for speed. Or the isolated processes for security

### User Threads vs Kernel Threads

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/19ccc626-a652-4356-b43a-e024410db286)

* **User Threads**
  * User-space library responsible for providing threading support
    * Threading operations are library (function) calls
    * The OS holds the process state
  * Greedy threads (for CPU time) may starve out other threads in the same process
  * A thread doing blocking IO amy make the entire process block
 
* **Kernel Threads**
  * OS provides support for threads natively
    * Threading operations are done via system calls (pthread_create())
  * The OS manages threads as it does processes, with preemption and scheduling
 
#### pthread_create()

* **User Threading**
  * On call we do a JAL into library code
  * We add another thread in the data structure that has the number of threads
  * It's library code that changes a library data structure inside of the user space

* **Kernel Threading**
  * Has to make a syscall to go into the kernel to change the thread state table (Changes a kernel data structure)
  * There are more context switches in a kernel threaded OS
 
#### Infinite Loop Problem

* In the case of an infinite loop, the preemption timer will go off and we enter into the OS becauese of a timer interrupt
  * Call the scheduler afterwards
* User threaded OS:
  * When we got the interrupt timer, we saved all the state and the $PC
  * Then we put the $PC right back where we got the interrupt
    * We're going to pick up right where we left off... in the infinite loop
  * There is no-user space preemption timer. This denies the other two threads CPU time
    * Starvation
* If we have a thread stuck in the running state and we want it to go into the ready state, what do we do?
  * Yield: The process will willingly yield its time
    * Terrible for processes, since they are competitive and want the CPU time
    * Since user threads are within the same process however, there is no competition
* It could be the programmer's problem
  * The purpose of the OS is to protect processes from other processes being greedy. Not from threads being greedy
  * If a thread is greedy, then we must code it to yield
    * yield() wil call into the library and calls the scheduler to pick another process
* In kernel threading, yielding is uneccesary and will cause context switches.
  * Threading library will do nothing if it finds a yield
  * Kernel threading has the preemption timer anyways, why yield in the first place?
 
##### In summary:

* In kernel threading: if a program runs for a long time:
  * Preemption timer goes off
  * We go into the kernel
  * We call the scheduler and it can pick up threads of processes
 
* In user threading: if a program runs for a long time
  * Preemption timer will still go off
  * If a process' thread is blocked, then the entire process will be blocked
    * As opposed to kernel threading where only the thread will be blocked
  * Since that thread is blocked, we can't run the other threads since the process is run as blocked
 
* In kernel threading, we implemented blocking by doing work that was ready
  * If there are other threads to run, rather than loop, yield
 
### Select()

* Problem: if we call read at any moment() there will not be any data and it will not block (i.e. our user is idle)
  * Is there some way we can ask the OS if we were to call read right now, would it block?
    * Select() syscall does just that. It's non blocking and just tells us if there was a key to grab
* When we are in the threading library, call select() to check if the data is ready
  * if data is available, then read()
* In a hybrid threading model, we would want to user kernel threading as the OS would not be able to have the capability for user threads

## I/O bound vs CPU cound processes

* With I/O bound processes
  * We have a process that takes more time in the I/O rather than the CPU time
  * Can do two I/O bound processes in the time of one
  * Batch system does not make sense in this case
* With CPU bound processes
  * Process that the vast majority of execution length would take the CPU
  * Interlace the CPU bound processes
    * If they each take 1 second, and CPU time is .8, then we could get them done in 1.6 by interleaving
      * Where we run instructions of one while the other one waits for I/O
     
## Slide set 2 summary

* A program and it's associated state is a process
* Scheduling is choosing among ready processes
* In a batch system, we run a process to completion, an then move on to the next process
* When a block occurs, we wait for input from the user to complete the data transfer to RAM
  * In the meantime, run another ready process 
* Kernel vs User threading
  * User threading occurs in user space
    * threading library gives user access to threading tools (called with normal functions)
  * Kernel threading occurs in the kernel (duh)
    * OS provides support for threading natively
    * Done through system calls
* Blocking in a thread can block an entire process
  * How to combat this? Use select()
    * Non blocking system calls that checks if info would be read if we called read()
* In a hybrid scenario, we use kernel threading because it requires kernel implementation
  * Requires OS support since the scheduling algorithm must support threads
  * Can still use the threading library, but it will slow down the system
* I/O bound
  * Spends most of its time blocking/waiting on I/O
* CPU bound
  * Spends most of its time running
  * Vast majority is getting the most out of our CPU cycle           

# Slide Set 3 (Scheduling Metrics and Scheduling Algorithms)

## When to schedule?

* Scheduling boils down to choosing whihc of the ready processes is next
* What event tells the kernel when to schedule?
  * A process is created
    * Now there is one more ready process to run
  * A process exits
    * The running process can no longer execute code until the requested data becomes available, pick something else ready
  * A process blocks
    * The running process can no longer execute code until the requested data becomes available, pick something ready
  * A HW IO interrupt
    * Data from a blocked process is now available, consider running something else
  * Clock Interrupts
    * The preemption timer warns us that the running process has used a lot of CPU time. Maybe pick something else
   
![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/e5a4ef62-9af4-4251-b161-d95b66a199cd)

### Characterizing our workload

* **Wall clock time** - How long it seems that a program takes to execute
* **User time**: how long the user executes instructions on behalf of the process
* **System time**: how long the OS was doing instructions on behalf of the process
* In I/O bound and CPU bound processes, they will take the same amount of time (1 unit)
* When we block, we use that time while waiting to run instructions in another process

#### Motivating our Scheduler

* Multiple I/O processes can be scheduled at the same time without increasing the amount of time necessary to run them
* Computational bursts before the process blocks might be so long, that the illusion of animation will break
* If the program doesn't naturally produce an event, we preempt via a preemption timer that runs an interrupt after a certain time
  * We have preemption events that are split up section of the CPU bound time to run CPU bursts
    * In a batch system, this means that our process is slower, but it retains the illusion of doing two things at once
* Every preemption interrupt is overhead as it is needed to maintain the illusion of two things running at the same time
* How are msot processes I/O bound?
  * Although our processes are interractive, our users are much slower compared to the processor
    * Most of the programs we use are I/O bound because we interact with them
* How do we turn a CPU bound process into an I/O bound process?
  * The only instance where something will take half time is if something is entirely CPU bound, since it only has to worry about computations
  * When the CPU gets faster, the bursts of computations keep getting smaller until the program becomes virtually entirely I/O bound
  * Having more transistors translates into faster processes bceuase the CPU components are closer physically, reducing the time it takes for instructions to reach their destination
  * More complicated CPUs allow for different algorithm implementations, leading to better performance
 
#### Three-Level Scheduling

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/7e312d71-9c80-4f20-bd6c-04a1c4f5e8fc)

* What else can we consider scheduling?
  * CPU scheduler is the only one that can be deemed positive (picks what process to run)
  * Negative scheduling is when we deny something the CPU (don't give it RAM)
    * Admission Scheduler
      * Ask it if we should load a process into RAM
      * If it says no (not in RAM or not enough RAM), then the CPU scheduler cannot pick it
      * Or, our workload added into RAM will not be better off with more work (no performance gain)
    * Memory scheduler
      * Evict the parts we aren't using to free up RAM
      * A place where we take a process with partial execution in RAM and temporarily hold it somewhere else (like a disk_    

### IO and CPU bound processes

* Multiple IO bound processes can be scheduled at the same time without increasing the amount of time necessary to run them
* I/O bound processes are very common
  * Interactive programs
* I/O bound processes will use overall have less total CPU usage since most of the time spent is waiting on I/O with shorter CPU bursts in between them
  * Compare than to CPU bound processes, which use more of the CPU as there are less frequent, but longer CPU bursts
    * And also less time doing I/O waits
   
## Scheduling Metrics

* Quantitative Metrics
  * **Throughput** - Number of jobs completed per unit of time
  * **Turnaround Time** - The time between a job being completetd and submitted
    * Includes time its waiting to be scheduled
  * **Average Turnraound Time** - The average turnaround times of a workload
 
* Computer Science Metrics
  * **Asymptotic Runtime** - Big O for the scheduling algorithm/data structure maintenance
  * **Engineering Difficulty** - How hard it is to get a particular algoritnm right
 
### Fairness

* **Comparable processes get comparable services**
  * Comparable does not mean identical/equal 

### Batch Scheduling Algorithm 1 - First Come First Server

* Simply: Jobs are run in the order they arrive (hence, the name)

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/317c046c-38dc-4ff2-90f3-10daf2928fe8)

* Complete Throughput = # of jobs / time = 4 jobs / 16 units of time = **1/4**
* Average Turnaround time:
  * sum of (finish - arrival) / # of jobs
  * A: 4 - 0 = 4
  * B: 7 - 0 = 7
  * C: 13 - 0 = 13
  * D: 16 - 0 = 16
  * 4 + 7 + 13 + 6 = 40/4 = **10**
 
* Logistics
  * If we have a queue in arrival order, we just simply remove the head which is an O(1) operation
  * Overall, the runtime for FCFS is **O(N)**
  * Fairness: FCFS seems reasonably fair, the first in line gets the resource first

### Batch Scheduling Algorithm 2 - Shortest Job First

* Simply: Jobs with the shortest runtime will be run first

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/3258193d-62ca-4678-a44d-e62e9879097a)

* Throughput: # jobs / time = 4 jobs / 16 units of time = **1/4**
* Average Turnaround Time:
  * A: 10 - 0 = 10
  * B: 3 - 0 = 3
  * C: 16 - 0 = 16
  * D: 6 - 0 = 6
  * 10 + 3 + 16 + 6 = **35/4**
 
* Logistics
  * Proof by exchange, this algorithm is optimal and gives you the shortest turnaround time
  * Asymptotic Analysis
    * When a job arrives, we have a list of jobs that are ordered by execution length: O(N)
    * Overall runtime: **O(n^2)**
    * What we want is O(N lg N)
      * How can we do that? HeapSort
      * Insert: lg n
      * Remove: constant
    * Comparable? Fair?
      * Most likely not - prone to starvation as longer jobs will stay in the back if shorter jobs keep arriving
        * Also provably impossible (halting problem)
          * How can we determing the runtime of a job?
      * A slightly faster turnaround time isn't worth it for a slower and impossible alorithm
     
#### More FCFS and SJF practice problems

1. FCFS

![image](https://github.com/Clester31/1550-midterm-review/assets/91839534/669a91c9-6d4c-4e68-ae6a-1bad59da4596)

* Throughput: 4 jobs / 26 secs = **2/13**
* Average Turnaround Time:
  *  



