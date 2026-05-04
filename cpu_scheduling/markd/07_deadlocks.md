# Deadlocks in Operating Systems

## What is a Deadlock?
A deadlock is a situation in a multiprogramming environment where a set of processes are blocked because each process is holding a resource and waiting for another resource acquired by some other process. 

## Necessary Conditions for Deadlock
A deadlock situation can arise if the following four conditions hold simultaneously in a system (known as Coffman conditions):
1.  **Mutual Exclusion:** At least one resource must be held in a non-shareable mode. Only one process can use the resource at any given instant of time.
2.  **Hold and Wait:** A process is currently holding at least one resource and requesting additional resources which are being held by other processes.
3.  **No Preemption:** A resource can be released only voluntarily by the process holding it, after that process has completed its task.
4.  **Circular Wait:** A set of processes $\{P_0, P_1, ..., P_n\}$ exists such that $P_0$ is waiting for a resource held by $P_1$, $P_1$ is waiting for a resource held by $P_2$, ..., and $P_n$ is waiting for a resource held by $P_0$.

## Resource Allocation Graph (RAG)
Deadlocks can be described more precisely in terms of a directed graph called a system resource-allocation graph.
- **Vertices:** Two types of nodes: Process vertices (Circles) and Resource vertices (Squares/Rectangles).
- **Edges:**
    - Request Edge: Directed edge $P_i \rightarrow R_j$ indicates that process $P_i$ has requested resource $R_j$ and is waiting for it.
    - Assignment Edge: Directed edge $R_j \rightarrow P_i$ indicates that resource $R_j$ has been allocated to process $P_i$.
- **Cycle in RAG:**
    - If a graph contains no cycles, then no deadlock exists.
    - If a graph contains a cycle:
        - If each resource type has exactly one instance, then a deadlock has occurred.
        - If each resource type has several instances, then a deadlock *may* exist.

## Methods for Handling Deadlocks
Generally, there are three ways to handle deadlocks:
1.  **Deadlock Prevention or Avoidance:** Ensure that the system will *never* enter a deadlock state.
2.  **Deadlock Detection and Recovery:** Allow the system to enter a deadlock state, detect it, and recover.
3.  **Ignorance (The Ostrich Algorithm):** Ignore the problem altogether and pretend that deadlocks never occur in the system (used by most operating systems, including UNIX and Windows).

### Deadlock Prevention
Provides a set of methods to ensure that at least one of the necessary conditions cannot hold.
- **Mutual Exclusion:** Not required for sharable resources (e.g., read-only files); must hold for non-sharable resources.
- **Hold and Wait:** Must guarantee that whenever a process requests a resource, it does not hold any other resources. (e.g., require process to request and be allocated all its resources before it begins execution).
- **No Preemption:** If a process that is holding some resources requests another resource that cannot be immediately allocated to it, then all resources currently being held are preempted.
- **Circular Wait:** Impose a total ordering of all resource types, and require that each process requests resources in an increasing order of enumeration.

### Deadlock Avoidance
Requires that the operating system be given additional information in advance concerning which resources a process will request and use during its lifetime. With this knowledge, the OS can decide for each request whether or not the process should wait. The most famous algorithm for deadlock avoidance is the **Banker's Algorithm**.
