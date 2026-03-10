# 01 — First Come First Serve (FCFS)

## What is FCFS?

FCFS (First Come First Serve) is the **simplest CPU scheduling algorithm**. Processes are assigned the CPU in the **exact order they arrive** in the ready queue — just like a queue at a ticket counter.

- **Type**: Non-Preemptive
- **Key property**: Once a process starts executing, it runs to **completion** without interruption.

---

## Key Terms

| Term | Full Form | Formula |
|------|-----------|---------|
| **AT** | Arrival Time | Given |
| **BT** | Burst Time | Given |
| **CT** | Completion Time | Time when process finishes |
| **TAT** | Turnaround Time | CT − AT |
| **WT** | Waiting Time | TAT − BT |
| **RT** | Response Time | Time of first CPU access − AT |

---

## How It Works — Step by Step

### Example

| Process | Arrival Time (AT) | Burst Time (BT) |
|---------|-------------------|-----------------|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

### Step 1 — Sort by Arrival Time
Processes are already sorted: P1 → P2 → P3 → P4

### Step 2 — Build the Gantt Chart

```
| P1  | P2 | P3       | P4     |
0     5    8          16       22
```

- CPU starts at 0, P1 runs from 0→5
- P2 runs from 5→8
- P3 runs from 8→16
- P4 runs from 16→22

### Step 3 — Calculate Metrics

| Process | AT | BT | CT | TAT = CT−AT | WT = TAT−BT |
|---------|----|----|----|-------------|-------------|
| P1 | 0 | 5 | 5 | 5 | 0 |
| P2 | 1 | 3 | 8 | 7 | 4 |
| P3 | 2 | 8 | 16 | 14 | 6 |
| P4 | 3 | 6 | 22 | 19 | 13 |

- **Average TAT** = (5 + 7 + 14 + 19) / 4 = **11.25**
- **Average WT**  = (0 + 4 + 6 + 13) / 4 = **5.75**

> **Note on idle CPU**: If a process arrives after all previous processes are done (AT > current time), the CPU sits **idle** until it arrives.
> Example: If P2 had AT=10 instead of 1, the CPU would be idle from 5→10. CT of P2 = 10 + 3 = 13.

---

## Advantages & Disadvantages

### ✅ Advantages
- Extremely simple to understand and implement
- No starvation — every process **will** eventually get the CPU
- Zero overhead — no need to track priorities or remaining burst times

### ❌ Disadvantages
- **Convoy Effect** — A long process can make all shorter ones wait a very long time
- High average waiting time compared to SJF or Round Robin
- Poor for interactive/time-sharing systems — short responsive tasks suffer

---

## Implementation Logic (Pseudocode)

```
sort processes by (arrival_time, pid)   // tie-break equal AT by PID
current_time = 0

for each process in sorted order:
    if current_time < process.AT:
        current_time = process.AT      // CPU is idle, jump to arrival

    process.CT  = current_time + process.BT
    process.TAT = process.CT - process.AT
    process.WT  = process.TAT - process.BT

    current_time = process.CT
```

> **Tie-breaking rule**: When two processes arrive at the same time, the one with the **smaller PID** runs first (i.e., the process that was created/submitted earlier).

---

## Implementation in C

```c
#include <stdio.h>

typedef struct {
    int pid;
    int at;   // Arrival Time
    int bt;   // Burst Time
    int ct;   // Completion Time
    int tat;  // Turnaround Time
    int wt;   // Waiting Time
} Process;

// Sort by Arrival Time; tie-break by PID
void sortByAT(Process proc[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            int swp = proc[j].at > proc[j+1].at ||
                     (proc[j].at == proc[j+1].at && proc[j].pid > proc[j+1].pid);
            if (swp) {
                Process temp = proc[j];
                proc[j] = proc[j+1];
                proc[j+1] = temp;
            }
        }
    }
}

void fcfs(Process proc[], int n) {
    sortByAT(proc, n);

    int current_time = 0;
    for (int i = 0; i < n; i++) {
        // Handle idle CPU
        if (current_time < proc[i].at)
            current_time = proc[i].at;

        proc[i].ct  = current_time + proc[i].bt;
        proc[i].tat = proc[i].ct - proc[i].at;
        proc[i].wt  = proc[i].tat - proc[i].bt;
        current_time = proc[i].ct;
    }
}

void printResults(Process proc[], int n) {
    printf("\nP\tAT\tBT\tCT\tTAT\tWT\n");
    printf("--------------------------------------------------\n");

    float total_tat = 0, total_wt = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\n",
               proc[i].pid, proc[i].at, proc[i].bt,
               proc[i].ct, proc[i].tat, proc[i].wt);
        total_tat += proc[i].tat;
        total_wt  += proc[i].wt;
    }

    printf("\nAverage TAT = %.2f\n", total_tat / n);
    printf("Average WT  = %.2f\n", total_wt / n);
}

int main() {
    Process proc[] = {
        {1, 0, 5},
        {2, 1, 3},
        {3, 2, 8},
        {4, 3, 6}
    };
    int n = sizeof(proc) / sizeof(proc[0]);

    fcfs(proc, n);
    printResults(proc, n);

    return 0;
}
```

**Compile & Run:**
```bash
gcc fcfs.c -o fcfs && ./fcfs
```

---

## Implementation in C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <iomanip>
using namespace std;

struct Process {
    int pid, at, bt, ct, tat, wt;
};

void fcfs(vector<Process>& proc) {
    // Sort by arrival time; tie-break by PID
    sort(proc.begin(), proc.end(), [](const Process& a, const Process& b) {
        return a.at == b.at ? a.pid < b.pid : a.at < b.at;
    });

    int current_time = 0;
    for (auto& p : proc) {
        if (current_time < p.at)
            current_time = p.at;  // CPU idle

        p.ct  = current_time + p.bt;
        p.tat = p.ct - p.at;
        p.wt  = p.tat - p.bt;
        current_time = p.ct;
    }
}

void printResults(const vector<Process>& proc) {
    cout << "\nP\tAT\tBT\tCT\tTAT\tWT\n";
    cout << string(50, '-') << "\n";

    float total_tat = 0, total_wt = 0;
    for (const auto& p : proc) {
        cout << "P" << p.pid << "\t"
             << p.at << "\t" << p.bt << "\t"
             << p.ct << "\t" << p.tat << "\t" << p.wt << "\n";
        total_tat += p.tat;
        total_wt  += p.wt;
    }

    int n = proc.size();
    cout << fixed << setprecision(2);
    cout << "\nAverage TAT = " << total_tat / n << "\n";
    cout << "Average WT  = " << total_wt  / n << "\n";
}

int main() {
    vector<Process> proc = {
        {1, 0, 5},
        {2, 1, 3},
        {3, 2, 8},
        {4, 3, 6}
    };

    fcfs(proc);
    printResults(proc);

    return 0;
}
```

**Compile & Run:**
```bash
g++ fcfs.cpp -o fcfs && ./fcfs
```

---

## Implementation in Python

```python
def fcfs(processes):
    """
    processes: list of dicts with keys 'pid', 'at' (arrival), 'bt' (burst)
    returns:   same list with 'ct', 'tat', 'wt' filled in
    """
    # Sort by arrival time; tie-break by PID
    processes.sort(key=lambda p: (p['at'], p['pid']))

    current_time = 0
    for p in processes:
        # Handle idle CPU gap
        if current_time < p['at']:
            current_time = p['at']

        p['ct']  = current_time + p['bt']
        p['tat'] = p['ct'] - p['at']
        p['wt']  = p['tat'] - p['bt']
        current_time = p['ct']

    return processes


def print_results(processes):
    print(f"\n{'P':<5} {'AT':<6} {'BT':<6} {'CT':<6} {'TAT':<6} {'WT':<6}")
    print("-" * 35)
    for p in processes:
        print(f"P{p['pid']:<4} {p['at']:<6} {p['bt']:<6} "
              f"{p['ct']:<6} {p['tat']:<6} {p['wt']:<6}")

    n = len(processes)
    avg_tat = sum(p['tat'] for p in processes) / n
    avg_wt  = sum(p['wt']  for p in processes) / n
    print(f"\nAverage TAT = {avg_tat:.2f}")
    print(f"Average WT  = {avg_wt:.2f}")


if __name__ == "__main__":
    processes = [
        {'pid': 1, 'at': 0, 'bt': 5},
        {'pid': 2, 'at': 1, 'bt': 3},
        {'pid': 3, 'at': 2, 'bt': 8},
        {'pid': 4, 'at': 3, 'bt': 6},
    ]

    result = fcfs(processes)
    print_results(result)
```

**Run:**
```bash
python3 fcfs.py
```

---

## Expected Output

```
P     AT     BT     CT     TAT    WT
-----------------------------------
P1    0      5      5      5      0
P2    1      3      8      7      4
P3    2      8      16     14     6
P4    3      6      22     19     13

Average TAT = 11.25
Average WT  = 5.75
```

---

## Summary

| Property | Value |
|----------|-------|
| Type | Non-Preemptive |
| Starvation | ❌ No |
| Convoy Effect | ✅ Yes |
| Complexity | O(n log n) — just sorting |
| Best for | Batch processing systems where simplicity matters |

---

*Next: [02 — Shortest Job First (SJF)](./02_sjf.md)*
