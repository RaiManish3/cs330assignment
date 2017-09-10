Following System Calls were implemented:

1. Syscall_GetReg ->
-------------------
Register (4) contains the register number which contains the data of interest.
So applying a composition of two "ReadRegister" functions (from machine.cc) gives the required result.

2. Syscall_GetPA ->
------------------

3. Syscall_GetPID ->
------------------
For alloting a unique PID, we use variable "nowPID" which increments at the creation of every new thread.
Also, we maintain a private variable "pid" for Thread Object and a public function "getPID()" to retrieve the PID of the thread. This pid initialises itself with nowPID at the time of creation of thread.

4. Syscall_GetPPID ->
------------------
Together with "pid", we also have a private variable "ppid" and a public function "getPPID()".
Gets initialised at the time of thread formation. When pid>0 ('main' has been created), the ppid for that is made equal to the current thread's PID else when p=0 if declare its ppid to be -1.

5. Syscall_NumInstr ->
------------------
We declare a private variable "instrCount" for each process and initialize it to 0 in its constructor. From documentation, it is clear that we execute assembly instructions of the user program one by one which is primarily triggered by an infinite for loop in Run() function of Machine class. Hence everytime, we execute a new instruction, we increment the "instrCount" of the current thread. To increment, we make use of a public function increaseInstrCount(). Hence when this syscall is called, we just return the current instrCount of currentThread by using another public function retInstrCount() and write it in to register 2.

6. Syscall_Time ->
------------------

7. Syscall_Yield ->
------------------

8. Syscall_Sleep ->
------------------

9. Syscall_Fork ->
------------------

10. Syscall_Exec ->
------------------
During implementing this syscall, we basically take inspiration from two already defined functions- PrintSyscall syscall and LaunchUserProcess function. Initially while copying the executable name passed as a argument to exec syscall to a character array, we proceed similar to PrintString syscall. Then we open the executable and assign a new ProcessAddressSpace to it if the executable isn't NULL which in case we return by incrementing the program counter, else we never increment the PC in exit syscall. After this, we close the executable file and delete the current thread's address space. Now, set the initial register values and load page table registers similar to LaunchUserProcess function. Here, modifications done for fork syscall in ProcessAddressSpace class become useful. We restore the old interrupts thereafter before executing the new instructions.

11. Syscall_Join ->
------------------
When the passed argument (the child PID) is read, first check is done to determine if it is a valid child
of the currently running process. This is done through "validChild()" function which basically iterates over the entire childPID array to look for matching PID. If it does not gets one, we return -1.
Else we must check if the child has already exited or not. This is done by looking at the array "childExitCode".
If this is positive, the child has already exited, we return its exit code. Else we mark this child by setting the waitChild of the parent to the pid of the child and then put parent to sleep. Note that parent cannot be woken up by any interrupt as it is neither in ready or sleep queue. So the child is responsible to wake him up.
Now when this child exits, it makes a check whether its parent was waiting upon him, by checking the waitChild field of the parent.
If so, child moves the parent to the ready queue. Else it simply exits.

12. Syscall_Exit ->
------------------
Exit syscall is always called with a non-negative argument (0 being when called by the main function. Store this argument from register $4 to an integer variable exitCode. Now, to keep track of total number of live threads, we have maintained a global variable threadsCount and initialize it to 0 when we call the Initialize() function at the beginning. Now, each time a thread is made, we increment this count in the thread's constructor. Also, whenever there's a thread to be destroyed checked while scheduling threads, we decrement this count before deleting that threadToBeDestroyed. Now, coming back to exit syscall, if the current threadsCount is 1 and we are exiting from only live thread, then we have to halt the machine from where it never comes back. Else, we check if current threads's parent was waiting for this thread to exit via a join syscall. To achieve this, we access the private pointer to current thread's parent by a public function getParent() and then set the exit status of current thread in an private array of parent's object containig exit codes of all its children via a public function setChildExitStatus. Also, for each process we maintain a private variable waitChild that stores the pid of it's child for which its is waiting upon to join which can be accessed by a public function getWaitChild(). Now check if the current thread's pid and parent's waitChild matches. If yes, set the waitChild to default -1 and then move the parent to ready queue by saving and restoring the old interrupt status. Call FinishThread() finally to set the threadToBeDestoyed and further scheduling.
