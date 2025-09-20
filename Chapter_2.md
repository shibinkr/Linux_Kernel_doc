# Chapter 2: The Kernel’s Mission — Serving User Processes

The kernel’s role is not defined by the individual tasks it performs—memory management, scheduling, I/O, or security. These are foundational, but they exist only to support user processes.

The kernel’s mission is simple yet absolute: ensure every process runs reliably, securely, and efficiently. If it fails to respond to system calls, allocate memory, or enforce isolation, it has failed its purpose.

## 2.1 A Reactive Design

The kernel is not an autonomous entity; it executes only when triggered. It enters execution in three contexts:

- **System Call** — request from user space.  
- **Interrupt** — signal from hardware.  
- **Kernel Thread** — scheduled maintenance tasks.  

The kernel is therefore reactive, responding only when demanded.

## 2.2 The Lifecycle of a Process

When a process starts (`exec`), multiple kernel subsystems act in sequence:

1. **VFS** resolves the executable path.  
2. **Filesystem driver** loads the binary.  
3. **Memory manager** allocates and maps memory.  
4. **Security modules** enforce permissions.  
5. **Scheduler** queues the process for execution.  

Each step depends on the others, showing how deeply interconnected kernel components are.

## 2.3 The Hidden Complexity of a Simple Read

Even a `read()` call involves multiple layers:

1. **Syscall handler** validates the file descriptor.  
2. **VFS** finds the file object.  
3. Depending on type, the request is routed (file, pipe, socket).  
4. If the buffer page is unmapped, the **memory manager** resolves the fault.  
5. Data is copied, then returned to user space.  

A trivial action in user space translates into a complex, coordinated effort inside the kernel.

## 2.4 System-Wide Coordination

Every operation—file access, networking, inter-process communication—activates multiple kernel components. No subsystem works alone; the entire kernel cooperates to deliver results.

## 2.5 Kernel Threads: Silent Supporters

Kernel threads do not work for themselves. Their role is to maintain conditions for user processes—reclaiming memory, flushing buffers, handling background I/O, or preparing resources. They are supporting actors that ensure user space remains healthy.

## 2.6 A System Built Around Userspace

The Linux kernel is a reactive, cooperative system:

- Activated on demand, never idle.  
- Designed to support and protect processes.  
- Structured to operate harmoniously across subsystems.  

At its core, the kernel is not self-serving. It is the engine that powers, protects, and enables user space.

Overview 
```yaml
          LINUX RUNTIME SYSTEM — KERNEL ORCHESTRATION
          ===========================================

+---------------------------------------------------------------+
|                           USER SPACE                          |
|---------------------------------------------------------------|
| - Processes (apps, shells, servers)                           |
| - Trigger syscalls to request kernel services                 |
| - Cannot access hardware or privileged memory                 |
+---------------------------------------------------------------+
          |                     |                       |
          v                     v                       v
    (1) System Call  (2) Hardware Interrupt  (3) Kernel Thread Wakeup
          |                     |                       |
          v                     V                       V
+---------------------------------------------------------------+
|                        KERNEL ENTRY POINTS                    |
|---------------------------------------------------------------|
| - Arch-specific handlers for SYSCALL, IRQ, faults             |
| - Transition from user to kernel mode                         |
| - Sets up context, stack, and dispatches to subsystem         |
+---------------------------------------------------------------+
                             |
                             v
+---------------------------------------------------------------+
|                      CORE KERNEL SUBSYSTEMS                   |
|---------------------------------------------------------------|
| [Scheduler]   - Selects which task runs on which CPU          |
| [MM / VM]     - Manages memory allocation and address spaces  |
| [VFS]         - Dispatches file/socket/sysfs calls            | 
| [Filesystem]  - Ext4, Btrfs, Tmpfs, etc.                      |
| [Net Stack]   - Handles TCP/IP, UDP, interfaces, sockets      |
| [Block I/O]   - Manages read/write queues, I/O scheduling     |
| [IRQ / Timers]- Handles hardware events, time-based tasks     |
| [Security]    - Enforces isolation (LSM, capabilities, ns)    |
+---------------------------------------------------------------+
                            |
                            v
+---------------------------------------------------------------+
|                        DEVICE INTERFACES                      |
|---------------------------------------------------------------|
| - Drivers for storage, network, USB, display, etc.            |
| - Perform MMIO, DMA, or send commands to hardware             |
| - Notify kernel via IRQ upon completion                       |
+---------------------------------------------------------------+
                            |
                            v
+---------------------------------------------------------------+
|                         HARDWARE LAYER                        |
|---------------------------------------------------------------|
| - Physical CPU, RAM, disks, NICs, buses, MMU, timers          |
| - Kernel communicates via device drivers and bus protocols    |
+---------------------------------------------------------------+

Legend:
- Control begins in user space.
- Kernel activates on demand (via syscalls, interrupts, or threads).
- Subsystems coordinate to fulfill each action.
- Execution ends by returning to userspace or sleeping until needed again.
```
