# Process Attributes

Every process in a Unix-like operating system has several attributes associated with it. The operating system uses these attributes to manage, schedule, and secure processes.

## Key Process Attributes

### 1. Process ID (PID)
A unique non-negative integer identifier assigned to every running process by the operating system. No two active processes can share the same PID. 
- You can retrieve a process's own PID using the `getpid()` function.

### 2. Parent Process ID (PPID)
Every process (except the initial `init` or `systemd` process, which has PID 1) is created by another process. The creator is the parent. The PPID is the PID of the process that created the current process.
- You can retrieve a process's parent's PID using the `getppid()` function.

### 3. User ID (UID) and Group ID (GID)
These define the ownership and permissions of the process. 
- **Real UID/GID:** Identifies the user who started the process.
- **Effective UID/GID:** Determines the permissions the process has for accessing files and resources.
- Functions: `getuid()`, `getgid()`.

---

## Viewing Process Attributes in the Terminal

You can view process attributes directly from the command line using the `ps` command.
```bash
# View all processes with PID, PPID, User, and Command
ps -ef

# View attributes for a specific process (e.g., process 1234)
ps -p 1234 -o pid,ppid,user,args
```

---

## C Implementations: Retrieving Attributes

To retrieve these attributes in a C program, you need the following headers:
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
```

### Example 1: Basic PID and PPID
This example shows how a parent and child can print their own PIDs and their parent's PIDs.

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    printf("Main process starting. PID: %d\n", getpid());

    pid_t pid = fork();

    if (pid < 0) {
        perror("Fork failed");
        return 1;
    } 
    else if (pid == 0) {
        // Child Process
        printf("[Child]  My PID is: %d\n", getpid());
        printf("[Child]  My Parent's PID (PPID) is: %d\n", getppid());
    } 
    else {
        // Parent Process
        wait(NULL); // Wait for child to finish so terminal output is clean
        printf("[Parent] My PID is: %d\n", getpid());
        printf("[Parent] My Parent's PID (PPID) is: %d (Usually the shell)\n", getppid());
        printf("[Parent] The child I just created has PID: %d\n", pid);
    }

    return 0;
}
```

### Example 2: User and Group IDs
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    printf("Process Information:\n");
    printf("--------------------\n");
    printf("Process ID (PID): %d\n", getpid());
    printf("Parent PID (PPID): %d\n", getppid());
    printf("Real User ID (UID): %d\n", getuid());
    printf("Effective User ID (eUID): %d\n", geteuid());
    printf("Real Group ID (GID): %d\n", getgid());
    printf("Effective Group ID (eGID): %d\n", getegid());

    return 0;
}
```

## Python Implementation: Retrieving Attributes

Python's `os` module provides straightforward functions to access these attributes.

```python
import os

def print_process_attributes():
    print("Process Information:")
    print("-" * 20)
    print(f"Process ID (PID): {os.getpid()}")
    print(f"Parent PID (PPID): {os.getppid()}")
    
    # UID and GID are mostly available on Unix-like systems
    if hasattr(os, 'getuid'):
        print(f"Real User ID (UID): {os.getuid()}")
        print(f"Effective User ID (eUID): {os.geteuid()}")
        print(f"Real Group ID (GID): {os.getgid()}")
        print(f"Effective Group ID (eGID): {os.getegid()}")

if __name__ == "__main__":
    print_process_attributes()
```
