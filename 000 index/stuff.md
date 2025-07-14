os basically is a interface to commivcate with the hardware of the system
two main goals are efficiency and usability
batch os
multiprogramming
multitasking
multiprocessing
real time os

program is a passive set of instructions stored in a disk, a process is an active instance of a prgraom in execution

for each process, the os creates a ds
the ds has 4 main sections
stack, heap, data segments, text segments

local variabls, functionds calls, and return 

data segments holds the global variables, and static variabls, 
text segment holds the comp

each process, has the above ds

to manages all the processes, there is another ds, which is pcb(pcb is per process, or all the processesd)

switching between processes is called context switching, the states of a process are also stored in the pcb of the process

the shit that is stored by the pcb
process id
process state (there are some states defined for all the processes, learn about these also)
program counter
cpu registe