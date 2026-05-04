# Introduction to the `fork()` System Call

## What is `fork()`?
In Unix-like operating systems, `fork()` is a system call used to create a new process. The newly created process is called the **child process**, and the process that initiated the `fork()` is called the **parent process**.

When `fork()` is called, the operating system creates an exact duplicate of the calling process. The child process gets a copy of the parent's memory space, variables, file descriptors, and CPU registers. However, they are separate processes; changes made to variables in one process do not affect the other.

## How `fork()` Works
The most unique characteristic of the `fork()` system call is that it is called *once* but returns *twice*: once in the parent process and once in the newly created child process.

You can differentiate between the parent and the child by checking the return value of `fork()`:

*   **Negative Value (`< 0`):** The creation of a child process was unsuccessful (e.g., system reached its maximum number of processes).
*   **Zero (`== 0`):** Returned to the newly created **child process**.
*   **Positive Value (`> 0`):** Returned to the **parent process**. The value returned is the Process ID (PID) of the newly created child.

## Required Headers
In C, you need the following headers to use `fork()`:
```c
#include <sys/types.h>
#include <unistd.h>
```

---

## Examples

### 1. Basic `fork()` Example in C
This example demonstrates the basic branching logic used after a `fork()`.

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    // Calling fork()
    pid_t p = fork();

    if (p < 0) {
        // Fork failed
        fprintf(stderr, "Fork Failed\n");
        return 1;
    } 
    else if (p == 0) {
        // This block is executed by the CHILD process
        printf("Hello from the CHILD process!\n");
    } 
    else {
        // This block is executed by the PARENT process
        // 'p' contains the PID of the child process
        printf("Hello from the PARENT process! My child's PID is %d\n", p);
    }

    // This line will be executed by BOTH the parent and the child
    printf("This is executed by both processes.\n");

    return 0;
}
```

### 2. Memory Isolation Example in C
This example proves that the parent and child have separate memory spaces. Changing a variable in the child does not affect the parent.

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    int shared_var = 10;
    pid_t p = fork();

    if (p == 0) {
        // Child modifies the variable
        shared_var += 20;
        printf("Child Process: shared_var = %d\n", shared_var);
    } else if (p > 0) {
        // Parent modifies the variable differently
        // Adding sleep to ensure child prints first for clarity
        sleep(1); 
        shared_var *= 2;
        printf("Parent Process: shared_var = %d\n", shared_var);
    }

    return 0;
}
```
*Output:*
```
Child Process: shared_var = 30
Parent Process: shared_var = 20
```

### 3. Python Implementation Using `os.fork()`
Python's `os` module provides a wrapper around the POSIX `fork()` system call.

```python
import os
import time

def fork_example():
    try:
        pid = os.fork()
        
        if pid < 0:
            print("Fork failed")
        elif pid == 0:
            print("Hello from the CHILD process!")
        else:
            print(f"Hello from the PARENT process! Child PID: {pid}")
            # Wait for child to finish to avoid mixing output too much
            os.wait() 
            
    except AttributeError:
        print("os.fork() is not available on this platform (e.g., Windows).")

if __name__ == "__main__":
    fork_example()
```
*(Note: `os.fork()` is available on Unix/Linux/macOS but not on standard Windows).*
