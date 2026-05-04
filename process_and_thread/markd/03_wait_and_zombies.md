# Waiting, Zombies, and Orphans

When using `fork()`, the parent and child processes execute concurrently. Often, the parent needs to wait for the child to finish its task before continuing. This is where the `wait()` system call comes in.

## The `wait()` System Call
The `wait()` system call suspends the execution of the calling process (usually the parent) until one of its child processes terminates.

**Required Headers:**
```c
#include <sys/wait.h>
```

If a parent has multiple children, `wait()` will return when *any* of the children terminate. To wait for a specific child, you can use `waitpid(pid, &status, options)`.

## Zombie Processes
A **Zombie process** (or defunct process) is a process that has completed execution (via the `exit()` system call) but still has an entry in the process table. 

This occurs because the parent process needs to read the child's exit status. When a child dies, it sends a `SIGCHLD` signal to the parent. If the parent does not call `wait()` to collect this exit status, the child remains a zombie.
- **Why is it bad?** While they don't consume memory or CPU, they consume a PID. If too many zombies accumulate, the system might run out of available PIDs, preventing new processes from starting.

## Orphan Processes
An **Orphan process** is a running process whose parent has terminated or finished before the child.
- **What happens to them?** When a process is orphaned, it is immediately "adopted" by the `init` process (or `systemd`, PID 1) in Linux. The `init` process periodically calls `wait()` to collect the exit statuses of any orphaned children that have terminated, preventing them from becoming zombies.

---

## Implementations and Examples

### 1. Using `wait()` to Synchronize Parent and Child
This C example shows a parent waiting for a child to finish before printing its final message.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        printf("[Child] Working on a task...\n");
        sleep(2); // Simulate some work
        printf("[Child] Task complete! Exiting.\n");
        exit(0); // Exit successfully
    } 
    else if (pid > 0) {
        printf("[Parent] Waiting for child to complete...\n");
        
        int status;
        // wait() blocks until the child finishes
        pid_t child_pid = wait(&status); 
        
        if (WIFEXITED(status)) {
            printf("[Parent] Child (PID: %d) exited with status %d\n", child_pid, WEXITSTATUS(status));
        }
        printf("[Parent] All done.\n");
    }

    return 0;
}
```

### 2. Creating a Zombie Process (Example)
This code intentionally creates a zombie process. You could compile and run this, then use another terminal to run `ps -l` or `top` to see the child process marked with a 'Z' or '<defunct>'.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        // Parent process
        printf("Parent (PID %d) is sleeping for 30 seconds.\n", getpid());
        printf("Check 'ps -l' in another terminal to see the zombie child!\n");
        sleep(30); 
        // Parent exits without calling wait()
    } 
    else if (pid == 0) {
        // Child process
        printf("Child (PID %d) is exiting immediately.\n", getpid());
        exit(0); 
    }

    return 0;
}
```

### 3. Creating an Orphan Process (Example)
This code intentionally creates an orphan. The parent finishes and exits while the child is still sleeping.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        // Parent process
        printf("Parent (PID %d) is exiting immediately.\n", getpid());
        exit(0); 
    } 
    else if (pid == 0) {
        // Child process
        printf("Child (PID %d) is sleeping. My parent is currently %d.\n", getpid(), getppid());
        sleep(2); // Parent will die during this sleep
        
        // After sleep, the parent has died. PPID will now be 1 (init/systemd) or a subreaper.
        printf("Child wakes up! My new parent is %d.\n", getppid());
    }

    return 0;
}
```
