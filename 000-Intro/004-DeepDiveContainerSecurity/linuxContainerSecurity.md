Why?
- Risk Assesement
- This talk is focused on the layer below containers

Segregation (Confidentiality )
- Container to Container
- Process to Process
- Container to the Outside

Access
- Who/When/Where
- Logging
- Start/Stop
- Content
  
Resource Usage

---
Von Neumann Computer (1945)
Processes had IO, ALU = Arithmetic Thing, Registers. Memory. 

Todays World:
:::mermaid
    flowchart TD
        subgraph UserLand
            AppCode-->SystemLibs
            SystemLibs-->Syscall_Interface
        end

        subgraph Kernel
            Syscall_Interface-->MemoryManager
            Syscall_Interface-->NetworkSubsystems
            Syscall_Interface-->Others
            MemoryManager-->CPUScheduler
            NetworkSubsystems-->FileSystems
            Others-->BlockDevices
        end
        
        subgraph Platform
            FileSystems==>MemroyControllers
            FileSystems==>DIMs
            FileSystems==>PCICommunication
            FileSystems==>CPUs
        end

        subgraph Periphery
            MemroyControllers==>Periphery-Block-Net-Char-Video
            DIMs==>Periphery-Block-Net-Char-Video
            PCICommunication==>Periphery-Block-Net-Char-Video
            CPUs==>Periphery-Block-Net-Char-Video
        end
:::

 Apps sit on all of that⬆️

# Namespaces

- **PID-Namespace**: Process Namespace 
- CPU/Memory-Namespace ()
- Network-Namespace
- User-Namespace
- FS/Mount-Namespace
- IPC-Namespace
- UTS Namespace

## PID/Process Namespace
- Each Process has a global and local PID
- Processes of different namespaces can't see each other
- 1st process of the Namespace has a PID1; others are linked to it
- Modifying a Process (Kill;-) is possible in same NS or Child
- Still all on the **Memory Management**: Some nuanced challenges here; One process can eat all the memory if not done propperly. 
- Working in a "Unified Shared Memory" which is a result of an underlyig combined memeory thingy.

## CPU/Memory (cgroups)
- Policy Based Scheduling
- CPU Affinity possible (Manage CPUs/Corresponding Containers)
- CPU usages must be calculated; "Fork Bomb" -- Seems like excessive process splitting can throttle the system
- There is a Timer/Scheduler that checks how much CPU is being consumed by each Container--run it too infrequently, and you'll risk mismanaging your CPU, but run it too frequently and risk burning excessive CPU on measurements. 
- Memory Limitation is difficult--Saying "My CGroup is only alowed to use X memory...what happens? "oom" killer--> Kills "oldest and largest process" in order to preserve memory. OOM killer can result in confusing results. 
- TLDR: Be mindeful of how you manage your memory.
- "Don't use memory blooming" -> Bad for performance
- Dirty/Used/Emtpy Pages is a global topic. 
- Be aware of **Context Switches**: If the CPU Scheduler decides it needs to move a process from one CPU to another. Impacts registers/Performance. 
- Be aware of **Migrations**: If the CPU Scheduler decides to switch from one process to another. Impacts registers/Performance. 

## Network-Namespace
- Puts interfaces into Namespaces
- Routing/Forwarding/Filters/Bridging happens in Kernel
- All Processes in a net-namespace can talk to the corresponding Network-Interface
- TCP/UDP/ICMP stack are still in the kernel

## FS (FileSystem)/Mount Namespace
- Mapping Table for Paths
- See syscall for Open, for example...
## UserNamespace (UID Namespace)
- Gives a container the ability to have their own user numbering
- User stuff

# Linux Kernals
- Linux Containers sit on a shared kernel
- they sit on a shared platform
- they can influence eachother quite easily
- even if process to process isolation is tight, it is just one layer
- networking is alwasy a discussion

# General Container Considerations
- Keep them small and start from scratch (If you use an external image, you don't know what you'll get)
- No Login from outside world
  - Don't Install of openssh in the Container
  - Exec your shell in the Container
- Container Technology != Network Encryptio