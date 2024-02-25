# Slide set 1 (Introduction to Operating Systems)

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

# Slide Set 2 (Scheduling Part 1)

## Scheduling

### CPU Time

* The first resource we are looking to manage is the time spent running insturctions on the CPU on behalf of a running program
* A running program and its associated data is called a **process**
* A program is the code and (static) data typically stored in an executable file on disk
  * The VN architecture prevents us from running it until it has been copied into RAM
  * You have to decide when it has to get in RAM, and then start running instructions

### Multiprogramming

* **Multiprogramming (AKA Multiprocessing/multitasking) is the ability to run multiple processes at the same time
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

### 
