================================================================================================================
                                                  Exokernels
================================================================================================================
1. What are the four different kinds of program discontinuities that may be experienced by a process of the library OS running on the CPU?
  * Interrupts
  * Exceptions
  *	System calls
  * Page Faults
	
2. How does Exokernel handle these program discontinuities and ensure that the right library OS gets involved in handling these program discontinuities.  (Use figures to get your points across clearly)
  * Exokernel maintains Processor Environments (PE) data structures on behalf of each library OS
  * PE contains the handler entry point for each different type of program discontinuities. 
  * When an event results in a trap, exokernel checks the corresponding PE data structure, and retrieves the entry point address in the library OS. 
  * Using the entry point, exokernel upcalls into the library OS and the library OS handles the event.

3. What is the role performed by the software TLB?	
  * Guaranteed mapping of virtual memory to physical memory is stored
  * On TLB miss, exokernel checks if mapping exists in S-TLB and if yes, the restores into H-TLB.
  * Reduces startup memory penalty for a LOS during context switch.
	
4. What is the role performed by the Processor Environment?
  * ExoKernel maintains a PE datastructure for each library OS
  * Each PE has an entry point address for handling program discontinuities (address context, interrupt context, exception context, protected entry context).
  * When a discontinity happens, Exokernel checks the corresponding PE and retrieves the entry point
  * ExoKernel upcalls the library OS and LOS handles the event

5. Enumerate the steps involved in context switching from one library OS to another.
  * Current state of outgoing LOS is saved using its PE and dumped into STLB for that LOS
  * Entries of the outgoing LOS in the Hardware TLB are emptied
  * Entries of the incoming LOS are loaded into Hardware TLB using the PE of chose one.
  * Volatile processor state is also loaded using the PE of incoming LOS

6. Choose the correct two choices pertaining to Exokernel from the following 
	A. Once acquired through secure binding, a library OS can hold on to resources as long as it wants.
	B. Processor environment (PE) data structure in Exokernel contains, among other things, information pertaining to entry points in the library OS for upcalls from the Exokernel.
	C. Upon a TLB miss, Exokernel always calls the faulting library OS for servicing the miss.
	D. The mechanism in Exokernel for downloading code into the kernel is to avoid border crossing for protection domains that are critical for achieving high performance.

	  * B and D

7. Consider a user process executing in a library OS on top of the Aegis Exokernel. The user process incurs a page fault (i.e., a TLB miss). Explain the action that follows in each of the following situations. You should 
identify the mechanisms and data structures available in Exokernel that enable it to accomplish the action.
	a) The faulting virtual address is a guaranteed mapping for the library OS.
	  * Using the PE, exokernel looks up STLB to find the mapping
	  * Exokernel installs the mapping and resumes processing.

	b) The faulting virtual address is not a guaranteed mapping for the library OS but a valid mapping exists for the virtual page in the page table of the faulting process.
	  * Using PE, exokernel identifies the starting address of the exception context
	  * exokernel upcalls LOS
	  * LOS looks up the page table for the process and retrieves the mapping
	  * LOS calls the TLB update primitive available to put the mapping into TLB by presenting the capability
	  * Exokernel resumes processing

	c) This is the first access to the faulting address by the user process.
	  * Using PE, exokernel identifies the starting address of the page fault
	  * LOS calls its disk driver to initiate I/O from disk to a Physical Page frame
	  * Device driver presents its capability for the hard disk controller and page frame number to exokernel to perform DMA
	  * Once DMA is complete, exokernel uses the PE interrupt context to upcall LOS
	  * LOS calls the TLB update primitive available to put the mapping into TLB by presenting the capability
	  * Exokernel resumes processing

8. Explain the role of the “processor environments”in the Aegis Exokernel.  How is it used to achieve the functionalities provided by a library OS?
  * Processor Environments are the datastructures for each LOS in Exokernel
  * PE contain the starting address space for various programming discontiuites (like system call, exception, page fault, interrupts)
  * When a program discontinuity happens then the PE for the respective LOS is contacted and the starting address space for that disconinity is obtained.
  * Then a upcall is made to the LOS to handle the event

9. This question is with reference to Exokernel. A user process makes a system call. Enumerate the sequence of steps (with a figure and concise bullets) by which this system call gets serviced by the library OS that this process belongs to.
  * Processor Environments are the datastructures for each LOS in Exokernel
  * PE contain the starting address space for various programming discontiuites (like system call, exception, page fault, interrupts). In this call for Protected Entry context
  * When a program discontinuity happens then the PE for the respective LOS is contacted and the starting address space for that disconinity is obtained.
  * Then a upcall is made to the LOS to handle the event

10. What is the purpose of the “software TLB” in the Exokernel approach to providing extensibility?
  * All library OSes live above the kernel. Thus each library OS is in its own hardware address space.
  * When a library OS is scheduled to run on the processor by Exokernel, the (hardware) TLB has to be flushed.
  * Thus the newly scheduled library OS will have an inordinate amount of TLB misses.
  * STLB manages the translation for virtual address to physical memory location.
  * Whenever a page fault happens in HTLB, then the STLB for the respective LOS is checked if there is an entry
  * If there is an entry the mapping is returned from there, otherwise an upcall is made to LOS using LE

11. “SPIN’s approach of extending logical protection domains and Exokernel’s approach of downloading code into the kernel are one and the same.”
  * False
  * Functionally, they accomplish the same thing, namely extend the kernel with additional functionality. (+2)
  * However, in Exokernel once the code is downloaded into the kernel there is no protection against malicious or erroneous behavior of the downloaded code. (+1)
  * In SPIN, since the extension via logical protection domains is achieved under the protection of the strong semantics of Modula-3, the other subsystems that live within the same hardware address space are protected from malicious or erroneous behavior of any given logical protection domain.

12. A library OS implements a paged virtual memory on top of Exokernel. An application running in this OS encounters a page fault. List the steps from the time the page fault occurs to the resumption of this application. Make any reasonable assumptions to complete this problem (stating the assumptions). (Concise bullets please)
  * Fielding by exokernel
  * exokernel notifies Library OS by calling the entry point in the PE data structure for this library OS 
  * Library OS allocates page frame running page replacement algorithm if necessary to find a free page frame from the set of page frames it already has 
  * Library OS calls exokernel through the API to install translation (faulting page, allocated page frame) in TLB, presenting capability, an encrypted cypher

13. How do you implement secure bindings
  * Hardware Methods like TLB entry
  * Software caching - Shadow TLB in software for each LOS helps in avoiding penalty during context switches.
  * Downloading Code in kernel - Done to avoid border crossing. Functionally equivalent to SPIN

14. Revocation of resource allocated to LOS
  * Revoke call to LOS through reposession vector.
  * LOS takes corrective action
  * LOS can seed exokernel for auto-save

15. How does secure bindings work in Exokernel
  * Decopule authorization of hardware from its use.
  * After binding call, it creates an encrypted key and gives it to LOS
  * LOS uses the encrypted key for any future access to the resource

================================================================================================================
                                              µ-Kernels
================================================================================================================

1. All the subsystems in the L3 microkernel live in user space. Every such subsystem has its own address space. The address space mechanism “grant” allows an address space to give a physical memory page frame to a peer subsystem. Can this compromise the “integrity” of a subsystem? Explain why or why not.
  *  No. 
  * All such mechanisms use the IPC of L3 which is one of the foundations for independence and integrity in L3
  * A subsystem that receives the memory page frame (via the grant mechanism) from a peer has the ability to “accept” or “reject” the page frame if it does not “trust” its peer.

2. Enumerate the myths about a microkernel-based approach that are debunked by Liedtke in the L3 microkernel.
  * Border crossing between User and Kernel switches
  * Address space switch
  * Thread switching and IPC
  * Memory effect due to loss of locality

3. How does Liedtke debunks the myths
	-- Border crossing between User and Kernel switches
	  * Proof by construction. µ-Kernel takes 123 CPU cycles in comparison to non µ-Kernel 
	-- Address space switch
	  * Flushing of TLB depends on implementation
	  * For small protection domains, use of address spaced TLB is HW supports it and architectural features like segment registers if HW does not supports it.
	  * For large protection domains its similar to other kernels

	-- Thread switching and IPC
	  * Shown by construction that its not true and is comparable to SPIN and Exokernel

	-- Memory effect due to loss of locality
	  * For small protection domains use architectural features like segment features.

4. Reasons for Mach's expensive border crossing
  * Code bloat because of focus on portability. This resulted in bigger memory footprint
  * Less locality and hence more cache misses.

5. Thesis of L3 for OS structing
  * µKernels should have minimum abstractions
  * µKernels are processor specific implementation and hence not portable.
  * Helps in building processor independent abstractions at a higher layer.

================================================================================================================
                                            Virtualization
================================================================================================================
1. What is the role performed by I/O rings in the Xen architecture?
  * Serves as a communication data structure between GOS and Xen
  * GOS places necessary descriptors corresponding to a HV call in I/O ring for Xen to pick up.
  * Xen places responses to HV call from GOS in the associated I/O ring
  * I/O rings unique for each GOS


2. A process “foo” executing in XenoLinux on top of Xen makes a blocking system call: fd = fopen(<file-name>);
Show all the interactions between XenoLinux and Xen for this call. Use pictures to make your description easier to follow.
  * XenoLinux fields the fopen call (fastpath)
  * XenoLinux puts all the necessary descriptors required for the call in the I/O ring corresponding to GOS
  * Xen performs the request
  * Xen puts the response in the I/O ring corresponding to GOS
  * XenoLinux picks up the response and makes the blocked request runnable again.

3. Explain the concept and use of fast path handlers in Xen.
  * Allows processes in GOS to access entry points in GOS without going through Xen
  * Implemented by Xen keeping a table for each GOS of traps and entry points
  * System calls are implemented using fast call handlers
  * Registered once by GOS at startup.
  * Upon System call, transfer control to entrypoint via table lookup

4. Distinguish between VPN, PPN, and MPN as discussed in VMWare.
  * VPN is the virtual address space for a memory location in a process inside a GOS.
  * PPN is the physical address space corresponding to a VPN
  * MPN is the Machine page number which actually maps to the real memory location. PPN to MPN mapping can be either managed by HV in Fully Virtualized environment and by GOS/HV in PV enviroment.

5. What does the shadow page table of ESX server hold and why?
  * Shadow table maps PPN to MPN
  * One SPT for each GOS
  * Removes on level of indirection
  * PPN to MPN mapping can be either managed by HV in Fully Virtualized environment and by GOS/HV in PV enviroment.

6. VM1 is not using its allocated memory; VM2 needs more memory; ESX server would like to get some memory from VM1 and give it to VM2. How is this accomplished with ballooning? Explain with pictures.
  * Ballooning is installed in GOS as a device driver
  * When memory is required from GOS then HV makes a direct call to balloon driver (BD) for memory.
  * This results in inflating the balloon and BD asking for memory from GOS
  * GOS procures the memory and gives to BD which inturn gives it to HV
  * Similarly deflation is used to return the memory back to GOS

7. Explain how content-based physical page sharing works across VMs in VMWare’s ESX server
  * Scan a candidate PPN and generate a content hash
  * If the hash matches an entry in the hash table, then perform full comparison of the scanned page and the matched page
  * If the two pages are identical then mark both PPNs to point to the same MPN thus freeing up a machine page; mark both the PTEs in the respective shadow page tables (PPN->MPN entries) as “copy on write”
  * If there is no match to the scanned page, then create a new “hint frame” entry in the hash table
  * The scanning is done in the background during idle processor cycles


8. Explain how I/O rings may be used to implement the networking subsystem of a guest OS in Xen (use pictures and concise bullets)
  * Receive and Transmit IO Rings
  * IO rings are specific to a GOS
  * Transmit IO ring just has descriptors enqueued in it
  * Shared buffers in GOS are used for actually storing the data and hence no copying.

9. What is the shadow page table (SPT), and who maintains it?
  * SPT maps PPN TO MPN
  * PPN to MPN mapping can be either managed by HV in Fully Virtualized environment and by GOS/HV in PV enviroment.

10. How many SPTs are there?   Why?
  * 1 for each GOS
  * Each guest OS has its own illusion of a contiguous physical memory, which has to supported by the hypervisor by having a distinct SPT for each virtual machine.

11.  Each process created by a guest OS has its own page table. Let’s call this physical page table (PPT). What is the relationship between PPT, SPT, and HPT?
  * Each process in a guest OS has its own PPT. The PPT contains the VPN to PPN mapping assigned by the guest OS.
  * Hypervisor allocates the real machine pages to back the physical pages of each guest OS. SPT contains the mapping between PPN and the MPN.
  * HPT is what the processor uses to translate the VA of the currently running process to the real physical address.

12.  How is page fault by a process handled with this set up? (Concise bullets please)
  * Hypervisor catches the page fault.
  * Passes the faulting VA to the currently executing guest OS.
  * Guest OS services the page fault.
  * Hypervisor traps the PT/TLB updates of the guest OS to directly enter the VPN to MPN mapping into the HPT.

13.  A process in a guest OS makes a request to read a file from the disk. Using figures, explain the steps taken by the guest OS and the Xen hypervisor using I/O rings to fulfill the guest OS's request.

  * The file system of the guest OS translates the file name to a disk block address and passes it to the disk subsystem of the guest OS.
  * The disk subsystem shares an I/O ring data structure with Xen.
  * Guest OS (i.e., the disk subsystem) uses an available slot in the I/O ring to enqueue the disk read request. It embeds a physical page frame pointer in this descriptor.
  * Hypervisor performs the disk I/O using DMA directly transferring the disk block into this physical page frame. 
  * Hypervisor enqueues a response using an available slot in the I/O ring data structure.


14. Give two or three concise bullet points explaining why virtualization is becoming a big thing in today’s computing landscape.
  * Resource consolidation: Sharing hardware resources across a set of users would bring down the overall cost of ownership and maintenance.
  * Utility computing: Companies can provide computing services with complete performance isolation and bill the users separately. The users have access to higher amount of resources while sharing their cost and do not have to maintain them.
  * Provides extensibility at the granularity of an operating system.

15. Define virtual pages, physical pages, and machine pages.
  * Virtual pages: These are the pages in the process address space of the user level processes in guest OS. Each user process sees a contiguous address space of virtual memory pages that represents its memory footprint.
  * Physical pages: These are the page frames that the guest OS thinks are backing the virtual pages of its user level processes.
  * Machine pages: Actual machine memory page frames in the DRAM that back the virtual pages.

16. In a fully virtualized setting, who keeps the mapping between physical pages and machine pages?
  * Hypervisor

17. In a para-virtualized setting, who keeps the mapping between physical pages and machine pages?
  * GOS or HV but GOS usually

18. Address translation in virtualized setting requires two levels of indirection: virtual -> physical -> machine
How is the performance impact of these two levels of indirection mitigated in full virtualization?
  * Shadow page table is the real hardware page table
  * PT/TLB updates of the guest OS trapped (privileged instruction)
  * Shadow page table (i.e., the real hardware page table) updated by the hypervisor
  * Translation installed into the TLB by the hypervisor (if allowed by the architecture, if not
	installed in the hardware page table
  * Subsequent accesses to this virtual page happens at hardware speeds without causing a trap


19. How is the performance impact of these two levels of indirection mitigated in para virtualization?
  * In para virtualization, the guest OS'es are aware of the underlying hypervisor. So, the guest OS's can maintain the efficient mapping of physical to machine pages. Allocating and managing the hardware Page table can be done by guest OS. For example, the guest OSes in Xen handle the page faults and install the new mappings into the hardware page table (which is a privileged operation) via hypercalls. The hypervisor provides hypercalls for operations like creating, switching and updating page tables to support these privileged operations by the VMs

20. Explain with appropriate examples, what is the opportunity for sharing memory across virtual machines.
  * Multiple instances of same OS, say Linux, can be running as multiple VMs on the same hardware. if both the instances are running the same application like Firefox, the two Firefox instances most likely have the same binary code. So, the content of the code pages for these processes will be the same. Sharing of these memory pages across virtual machines, without affecting the protection, can thus benefit the whole system.


21. Explain, memory sharing across virtual machines can be accomplished unbeknownst to the VMs.
  * Content based sharing can be used to achieve VM oblivious memory sharing. To implement this, the hypervisor maintains the content hashes for the different memory pages in a hashtable. The hashtable entries contain the following information which acts as a hint for a possible content match:
		• Hash value for the page
		• VM that the page belongs to
		• PPN and MPN
	To check whether a candidate PPN can be shared:
		• Generate a content hash and search the hashtable for a match
		• If there is a match, then do full comparison of the candidate page and the matched page
		• If the two pages are identical then change the page table entries such that both the PPNs point to the same MPN and free the old machine page
		• Mark both the page table entries in the respective shadow page tables as “copy on write”
		• Else, then add the content hash to the hashtable as a new hint.

22. What is the distinction between “physical page” and “machine page” in a virtualized setting?
  * Physical page is the illusory view of the physical memory from the Guest OS MMU. Physical pages are deemed contiguous from the Guest OS point of view.
  * Machine page is the view of the physical memory from the Hypervisor. This refers to the REAL hardware physical memory. The actual “physical memory” given to a specific guest OS maps to some portion of the real machine memory.

23. A new process starts up in a para-virtualized library OS executing on top of the Xen hypervisor.  List the interaction between the library OS and Xen to establish a distinct protection domain for the process to execute in.
  * Library OS (Linux) allocates memory from its own reserve for a new page table.
  * Library OS registers this memory with the Hypervisor (Xen) as a page table by using a Hypercall.
  * Library OS makes updates to this page table (virtual page, physical page mapping)
	through hypervisor via batches of updates in a single hypercall. 
  * Library OS changes the active page table via the hypervisor thus in effect “scheduling”
	the new process to run on the processor.


24. The library OS is “up called” by Xen to deal with a page fault incurred by its currently running process. How does the library OS make the process runnable again?
  * When a page-fault occurs, the hypervisor catches it, and asynchronously invokes the corresponding registered handler. Following are the steps:
	Inside the hypervisor:
		- Xen detects the address which caused the page-fault. For example, the faulting virtual
		address may be in a specific architectural register 
		- This register value is copied into a suitable space in the shared data-structure between the hypervisor and library OS. Then the hypervisor does the up-call, which activates the registered page-fault handler.  Inside the library OS page-fault handler:
		- The library OS page fault handler allocates a physical page frame (from its pool of free, i.e., unused pre-allocated physical memory kept in a free-list). 
		- If necessary the library OS may run page replacement algorithm to free up some page frames if its pool of free memory falls below a threshold. 
		- If necessary the contents of the faulting page will be paged in from the disk. 
		- Note that paging in the faulting page from the disk would necessitate additional interactions with the hypervisor to schedule an I/O request using appropriate I/O rings.
		- Once the faulting page I/O is complete, the library OS will establish the mapping in the page table for the process by making a hypervisor call. 


25. When the library OS schedules a process to run, how does the CPU learn where the page table is for doing address translation on every memory access?

  * To schedule a process the library OS will do the following:
		- library OS has a distinct PT data structure for each process. 
		- dispatching this process to run on the processor involves setting the PTBR (a privileged operation)
		- The library OS will try to execute this privileged operation 
		- This will result in a trap into the hypervisor 
		- The hypervisor will “emulate” this action by setting the PTBR 
	Henceforth, the CPU will implicitly use the memory area pointed to by the PTBR as the page table

26. When the library OS services a page fault and updates the page table for a given process, how is this mapping conveyed to the CPU so that it can do the address translation correctly for future access to this page by this process?
  * Page fault service involves finding a free physical page frame to map the faulting virtual page to this allocated physical page frame. 
  * To establish this mapping, the library OS has to update the page table or the TLB depending on the specifics of the processor architecture assumed by the fully virtualized library OS. Both of these are privileged operations which will result in a trap when the library OS tries to execute either of them.
  * Hypervisor will catch the trap and “emulate” the intended PT/TLB update by the library OS’s into the library OS’s illusion of PT/TLB. 
  * More importantly, the hypervisor has a mapping of what machine page (MPN) this physical page of the guest OS refers to in its shadow page table. 
  * Hypervisor will establish a direct mapping between the virtual page and the corresponding machine page by entering the mapping into the hardware page table (the shadow page table) or the TLB depending on the specifics of the processor architecture. 


27. Explain how the page table for a newly created process is set up by a library operating system executing on top of Xen.
  * Allocates and initializes physical page frame and registers it with Xen to serve as page table for process
  * creates VPN to PPN mapping for the process
  * Communicates the mapping via hypercalls to Xen
  * Xen populates PT with these mappings
  * Allocates page frames for the process to back these VPN to PPN mapping

============================================================================================================
                                        Shared Memory Model
============================================================================================================

1. What are the different type of models in Shared Memory Model
  * Entire memory space is accessible by any CPU
  * Dance Hall Architecture - Here CPUs are on one side and memory on the other. 
  * Symmeteric Multi processor - CPUs are on one side and memory on the other and a private cache is associated with each processor.
  * Distributed Shared Memory - CPUs are on one side and they have a private cache associated with them and a single shared memory shared amongst multiple CPUs

2. Cache Coherency Problem
  * Memory location cached by different processors and they need to coordinate with each other in terms of dirty cache. This problem arises because each processor has its private cache

3. Memory Consistency Model (MCM)
  * Its a contract between hardware and software
  * Consistency expectation from programmer's POV

4. Sequential Consistency Model
  * Type of a MCM
  * Maintain program order (textually)
  * Arbitrary interleaving between different processes

5. What are the different type of Hardware Cache Coherent models
  * Write Invalidate Model - If there is any change to memory location that are cached locally then invalidate 
  * Write Update - Done by propogating through system which updates the cache location.

6. What are scalability issues with synchronization?
  * Latency - Time spent by thread in acquiring lock
  * Waiting time - Time spent in waiting to get a lock
  * Contention - How many other threads are contending for the same lock

7. Array Based Queueing lock
  * Pros - Fairness, private spin variable for each process, not noisy, 1 atomic operation per critical section
  * Cons - High space complexity  (static - based on number of processors)

8. Link List based queueing lock
  * Pros - Fairness, private spin variable for each process, not noisy, 1 atomic operation per critical section, low space complexity depends on number of requestors (dynamic)
  * Cons - LinkedList maintenance and hence might be slower

9. What are the cons with centralized barrier algorithm ?
  * 2 spin loops

10. What are the cons with sense reversing algorithm ?
  * Hot spot on one variable
  * not scalable

11. What are the pros and cons of Tree Barrier algorithm ?
  * Pros - Limited sharing so scalable
  * Cons
    - Spin location is not statically determined. Depends on thread arrival pattern.
    - Contention increases with number of child
    - Non cache coherent processor will lead to spin location on a remote location

12. What is NUMA Architecture ?
  * Access to local location is faster than remote localtion

13. What are the properties of a MCS Tree Barrier (4-ary Arrival)
  * Wakeup tree is binary
  * Arrival tree is 4-ary
  * Statically allocated spin location
  * contention for shared location is low

14. What are the properties of a Tournament Barrier algorithm ?
  * Spin location is statically assigned
  * No need of fetch-and-free operation, just need atomic read and write operation
  * Can take advantage of IPC
  * Works even if the processor is not a shared multi-processor
  * Amount of communication is log N
  * Cant exploit spatial locality even in cluster machine

15. What are the properties of a Disseminaton algorithm ?
  * Spin location is statically determined
  * No hierarchy
  * Works for cluster and NCC
  * No waiting for anybody else.
  * Number of rounds ceiling of log N
  * Communication complexity of N(log N)
  * Communication per round N


============================================================================================================
                                                RPC
============================================================================================================
1. What are the different steps in a RPC call
  * Caller's call results in a kernel traps
  * Kernel validates the call
  * Kernel copies the argument in kernel's address space. This is done by serialization of the data structure into a RPC message
  * Kernel locates the server procedure that needs to be executed
  * Kernel copies the argument in server's address space
  * Kernel schedules the server to execute the procedure
  * Server procedure starts the execute
  * On completion of the procedure, it results in a kernel trap for returning the results
  * Kernel copies the response in server's address space into kernel buffer
  * Then kernel copies the response in Kernel buffer
  * Then kernel copies the response into client's address space
  * Kernel schedules the client to pick up the response and unblock the client
  * Everything happens at run time.

2. How to make RPC calls cheap ?
  * Client stub copies the arguments from client stack (passed by value) on A-stack. 
  * Client's procedural call results in a trap in the kernel and presents the BO
  * BO is the capability to validate if the client is authorized to make a call to server
  * Once BO is validated by the kernel, kernel can see the Procedural Descriptor associated with the BO
  * At this point kernel can borrow the client thread and it can doctor the client thread to run on server address space.
  * Kernel allocates Execution stack which can be used by server procedure can use.
  * Now kernel trasnfer controls to server 
  * Service stub takes the argument from A-stack and copies to E-stack.
  * Now execution of the server process starts.
  * After completion of execution, the results are copied to A-stack
  * Server does a return trap to kernel
  * Kernel re-doctors the thread to run in client address space
  * Client thread re-executes and client stub copies the result into client stack
  * May result in loss of locality


3. Explain RPC on a SMP 
  * Exploit multiple processors
  * Pre-load server domain on a processor and dont let anything else run on it then it can keep the caches warm for whatever domain runs.
  * A-stack is provided in the shared memory and no kernel intervention is required.
  * Kernel mediation is required only when the client call needs to be authorized

============================================================================================================
                                            Scheduling
============================================================================================================

1. First Come First Serve Scheduling policy
  * More weightage on fairness
  * less weightage on affinity

2. Fixed Processor Scheduling policy
  * More weightage on affinity
  * less weightage on fairness

3. Last Processor Scheduling policy
  * Schedule a thread based on where was it last run

4. Minimum Intervening Scheduling policy
  * Keep thread-processor affinity
  * Affinity is calculate by counting number of intervening threads
  * Higher the affinity number lesser the chance of scheduling as the cache might have been dirtied

5. Minimum intervening plus queuing Scheduling policy
  * Use affinity index and queue for processor
  * If the queue is longer then dont schedule

6. What are the various implementation issues with scheduling
  * Global queues become infeasible when huge number of processors. Global queues can be avoided by keeping  affinity based local queues.
  * Queues can be also organized by its priority.

7. For heavily loaded processor then fixed processor is better

8. Procrastination
  * A processor might insert an idle loop before picking up a job
  * Done because it does not has any thread in the run queue which is not in cache, waiting and picking something which has its content in the cache will help in performance

9. 

============================================================================================================
                                            Papers
============================================================================================================
1. What is Balance Set Scheduling
  * Its a way to improve the performance of virtual memory
  * Used for L2 caches
  * Divide all the runnable threads into sub group or sets such that combined working set of each group fits into cache
  * Reuse pattern of memory locations in the working set is more important than the size of the working set
  * Estimate cache miss ratios for all possible groups of threads using the Berg-Hagersten-based model
  * Select cache miss ratio threshold
  * Schedule those thread groups whose estimated cache miss ratio is below the threshold.

2. What is operating system ? 
  OS is a software which helps in secure access to hardware devices, provides well defined APIs for accessing OS services and helps in reducing contention for competing H/W requests.

3. What is Abstraction ?
  Abstraction is a way of providing a well defined interface which hides the actual implementation of a subsystem

4. What is a program ?
  It is a static image of itself loaded into the memory. It is the memory footprint created when you start it.

5. What is a process ?
  A program in execution is called as a process. Process is a program with the state of all the threads executing in it.

6. What is a thread ?
  A thread is a lightweight entity within a process that can be scheduled for execution. All threads of a process share its virtual address space and system resources. 

7. What are the important characterstics of an OS structure ?
  * Protection
  * Flexibility
  * Performance
  * Scalability
  * Agility
  * Responsiveness

8. Monolithic Operating System
  * Clear distinct seperation between OS and applications so that user is protected from OS and vice-versa.
  * Applications run in their own AS and so does OS.
  * OS provides services for accessing hardware and other services
  * Results in loss of performance because of seperate AS but improved security.
  * No extensibility

9. DOS Operating System
  * OS and applications are in the same address space and hence a system services call is like a procedural call.
  * Provides higher performance but results in poor protection
  * Provides extensibility

10. µKernal Based operating system
  * Built for extensibility
  * Poor in performance because of code bloat
  * Secure
  * Application and OS in its own AS
  * Operating system services implemented on top of the kernel
  * Inter Process communication for interaction between application and services
  * Bad performance because of border crossing and loss of locality

11. What is the underlying premise for SPIN and ExoKernel based approach
  * Monoloithic structure of an OS does not gives extensibility
  * µKernal based design compromises on performance because of frequent border crossing and loss of locality.
  * Performance like DOS, protection like Monolithic and extensibility like µKernel

12. What were the features of Hydra OS
  * Provided kernel mechanisms for resource allocation
  * Resource managers built as coarse grained objects to reduce border crossing overhead
  * Due to coarse grained objects, less oppurtunities for customization and extensibility.

13. What were the key features of Mach based OS
  * µKernel based with very limited mechanisms
  * All the services implemented as user process
  * Focus on portability and extensibility
  * Bad performance

14. What are the key features of SPIN
  * Co-locate a minimum kernel with its extension in the same hardware AS to avoid border crossing.
  * Compiler based modularity. Uses the characterstics of a strongly typed language (Modula 3) to enforce modularity.
  * Logical protection domain provided by the data abstractions provided by programming language. Not reliant on hardware address space.
  * Dynamic call binding which helps in providing extensibility by providing different implementation to the same interface defination.

15. What are the features of Modula 3 or how does it allows for logical protection domains ?
  * Strongly typed language with built in safety and encapsulation mechanisms.
  * Automatic Memory management
  * Supports data abstraction called as objects with well defined entry points
  * Supports the notion of threads that execute in the context of the object, and it allows raising exceptions
  * Fine grained protection via capabilities
  * Capabilities as language supported pointers

16. What are the different protection domains provided by SPIN
  * Create  - Used for initializing the objects and exporting the name
  * Resolve - Used for dynamic binding of source and target domain
  * Merge   - Used for creating an aggregated domain

17. How does SPIN helps in Memory Management ?
  * SPIN wants to allow extensions to manage physical memory allocated to them in whatever fashion they choose to
  * Provides interface functions that are defined as core services memory management system
  * Does not enforces on implementation
  * Once implemented, it creates a logical protection domain which becomes part of SPIN extension
  * Functions executed when the hardware event occurs

18. How does SPIN supports the concept of threads ?
  * Provides an abstraction called Strand
  * Actual extension/OS will have to map its implementation of threads to Strand
  * Strands are used by Spin's global scheduler

19. How does exokernel facilitates hardware resource access
  * LOS asks for access to a resource
  * Exokernel validates the requests and creates a secure binding and gives an encrypted key back to LOS
  * LOS uses this key to access the resource, Exokernel verifies the key and allows access.

20. How does exokernel implements secure binding ?
  * Hardware Mechanisms using the encrypted key.
  * Software caching. caching the hardware TLB in software cache for each LOS and avoiding the context switch penalty. 
  * Downloading code into the kernel - Done to avoid border crossing. It is functionally equivalent to SPIN extensions.

21. Who compromises security more - Exokernel or SPIN ?
  * Exokernel because there is no validation once the code has been downloaded into the kernel
  * SPIN uses Modula 3 language which provides compile time checking, run time verification and language enforced

22. How are page faults handled in ExoKernel
  * Page fault is fielded by ExoKernel
  * Exokernel knows which LOS owns the currently running process
  * Exokernel upcalls the LOS page fault handler and asks for the page frame
  * LOS figures out the mapping and gives the mapping and encryption key to ExoKernel
  * Exokernel verifies the key and if its good then installs the mapping on Hardware TLB

23. Explain memory management in ExoKernel using software TLB
  * Software TLB is sort of a snapshot of the hardware TLB for each of the LOS
  * STLB is a datastructure for each LOS on exokernel
  * In case of a context switch, HTLB will dump subset of mappings for that LOS in corresponding STLB and load the mappings of the incoming LOS from its STLB into HTLB

24. Explain how does Exokernel manages CPU scheduling 
  * Exokernel maintains linear vector of time slices
  * Every time slice has a begin and an end and is provided to LOS representing how much time it can use on CPU
  * When the time quantum ends the LOS has to give up the CPU and the new one takes over

25. Compare the revocation policy in case of SPIN and Exokernels
  * SPIN has a concept of strands which maps the library's notion of thread and it uses that for revocation
  * Exokernel upcalls the LOS when it wants to take away resources and provides a reposession vector
  * LOS uses the repossession vector to clean up the resources
  * Exokernel also provides a way to seed auto-saving options to LOS

26. What are the different type of programming discontinuties ?
  * Page fault
  * Interrupt
  * System calls
  * Exceptions

27. How does Exokernel deals with Programming Discontinuities ?
  * Uses PE datastructure
  * Invokes the specific handler whenever a disconintuity happens 
  * Upcalls the LOS

28. What data structure does Exokernel uses for dealing with programming discontinuities ?
  * Processor Environment (PE) datastructure maintained by Exokernel for each LOS
  * PE contains the entry points in the LOS for dealing with the different kinds of program discontinuities
  * Page faults are handled by 
  * Interrupts are handled by Interrupt Handlers
  * System calls by Protected Entry Context
  * Exceptions are handled by exception handlers 

29. What is the implicit cost of border crossing 
  * Address translations contained in TLB because we have to switch the AS
  * Loss of locality 

30. What are strikes against µKernel
  * Border crossing cost between kernel and user switch
  * AS switch between services
  * Thread switching context and IPC
  * Memory effects due to loss of locality

31. How does L3 debunks the myths against µKernels
  * Border crossing cost depends upon implementation and a carefully implemented L3 µKernel just takes 123 cycles as compared 107 cycles
  * Address space switches can be handled by tagging the TLB with AS. Even if AS tagged TLB is not available, use the hardware capabilities like Segment registers. Shared hardware AS for protection domains.In case of large protection domains if the whole HW AS is covered then TLB flush is the only option.
  * Thread switching - Costs involved in thread switching, is saving all the volatile state of the processor, the T1 thread that is running on the processor. It's modified the registers of the CPU. All of that has to be stashed away in the thread context block before we can schedule the second thread to run on the processor. Proves it by construction that it is comparable
  * Memory effects - the memory effects can be mitigated by carefully structuring the protection domains, in the hardware address space. In case of small protectiond domains, don't put each of these in its own hardware address space. Pack them together in the same hardware address space and then force protection, for these processes from one another, through segment registers.

32. What are the reasons for Mach's expensive border crossing ?
  * Code bloat
  * loss of locality

33. What is Liedtke's thesis of L3 for OS structuring
  * µKernel should have minimum abstractions
  * µKernel are processor specific and hence not portab;e
  * Right set of µkernel abstractions and processor specific implementation.

34. Why is virtualization important ?
  * Access to more resources at a minimum cost
  * Cost benefits in terms of owning, upgrading and maintaining it.
  * Provide resources with complete performance isolation and bill each individual user separately.

35. What is a Virtual Machine Manager or a hypervisor ?
  * Operating system of operating systems
  * Manages the individual OS

36. What are the different type of hypervisors
  * Native/Bare Metal Hypervisors - Run directly on bare metal machines. Guest OS (GOS) run directly on hypervisor resulting in better performance. e.g. Xen
  * Hosted Hypervisors - Run as an application process on top of host hardware. e.g. VMWare

37. What are the different type of virtualization techniques ?
  * Full Virtualization - Leave the GOS unmodified and run it on Hypervisor.GOS is run as an application process.
  * Para Virtualization - Modify the source code of GOS. Helps with problematic instructions (those who fail silently) and can also lead to optimizations. e.g Xen

38. What is trap and emulate strategy ?
  * GOS thinks its running in the privilege mode on bare metal
  * GOS tries to execute privileged instruction which results in a trap
  * Hypervisor will then emaulate the instruction, execute it and give the result back to GOS

39. What is binary editing strategy ?
  * GOS thinks its running in the privileged mode on the bare metal
  * It runs some privileged instruction but fails silently without resulting in a trap
  * Hypervisor already knows about these instructions and looks out for them and takes the appropriate action

40. What is a shadow page table and how is it used for effecient mapping in case of Full Virtualization ?
  * In moder OS each process run its own protection domain and a seperate AS. OS maintains a page table for each of the process for translating VPN to a PPN.
  * In case of virtualization GOS thinks that PPN is the real memory since its sitting on top of the HV.
  * However MPN is the real memory and only Hypervisor knows about it.
  * Whenever an address translation has to happen, VPN to PPN happens in a page table allocated to that process by the OS and the PPN to MPN translation happens in a different page table called as Shadow Page Table which is maintained by the hypervisor.
  * However this process is ineffecient cause it results in 2 levels of indirection
  * To make it effecient, whenever the GOS tries to update the PT owned by the process it results in a trap because its a privileged instruction
  * This trap is handled by HV and it identifies that this needs PPN points to a MPN and updates the SPT
  * It further installs this translation into the TLB or Hardware Page Table.
  * We are not going through the guest operating system to do the translation, but so long as the translation has already been installed in the TLB and the hardware page table.

41. How is memory management done in a ParaVirtualized environment ?
  * Since the GOS is modified it knows its running on the hypervisor.
  * GOS maintains the mapping of continous Physical memory to discontinous hardware page (machine memory)
  * Hypervisors like Xen provide hypercalls which help in telling Xen about changes in Hardware Page Tables
  * Each GOS gets physical memory when it starts working on a hypervisor and that is used by it to host a page table on behalf of the new process. This also results in a create page table hypercall.
  * When the newly created process needs to run on the processor, GOS sends a hypercall to switch to the page table.
  * Similalry in case of updating the page table in case of page faults, it results in an update page table hypercall to Xen.

42. How can you dynamically increase memory in a virtualized environment ?
  * Uses the technique called Ballooning.
  * Special device driver in each GOS.
  * In case of memory requirements by a GOS, HV contacts the GOS and ask the balloon device driver to inflate.
  * Inflating of balloon causes the GOS to give more memory to Balloon by paging out unnecessary processes and freeing up memory. This is further handed to HV so that it can feed the starving GOS
  * Deflation of balloon causes the GOS to obtain more memory which means it can page in more processes

43. How can we share memory across VMs or what is VM oblivious page sharing ?
  * Content based sharing
  * Used in VMWare ESX Server
  * A hashtable in HV stores the MPN and hash of its content
  * When a VM tries to create a PPN to MPN entry then check the hashtable if there is a match on the hash
  * If there is a match then its a hint that content might be same. 
  * Check the contents completely and see if they match.
  * In case of a successful match update the PPN to MPN entry to point to this location and icnrease the reference count in the hash table by 1
  * In case on of the process wants to change the content then make a copy of it and use the freed up space.

44. What are the different memory allocation policies ?
  * Pure Share based approach - Pay less, get less.Might lead to hoarding and wastage.
  * Working set policy - Based on working set - shrink and expand.
  * Dynamic Idle adjusted shares approach - Tax the guys who are hoarders. Tax might be 50% chances of taking away the allocated resources.
  * Reclaim most idle memory - Allowing for sudden working set increases.
