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

