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
