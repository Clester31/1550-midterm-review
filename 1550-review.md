#1550 review

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
 
* How can we determine wheteer or not we can use privleged mode instructions?
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
