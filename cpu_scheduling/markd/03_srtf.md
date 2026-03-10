# 03 — Shortest Remaining Time First (SRTF)

## What is SRTF?

SRTF is the **preemptive version of SJF**. At every moment a new process arrives, the CPU checks if the newcomer's burst time is **less than the remaining burst time** of the currently running process. If so, the current process is **preempted** and the CPU is handed to the shorter one.

- **Type**: Preemptive  
- **Also known as**: Preemptive SJF  
- **Key property**: Theoretically optimal — gives the **minimum possible average waiting time** for any scheduling algorithm (assuming burst times are known).

---

## Key Terms

| Term | Full Form | Formula |
|------|-----------|---------|
| **AT** | Arrival Time | Given |
| **BT** | Burst Time | Given |
| **RT** | Remaining Time | Decreases as process runs |
| **CT** | Completion Time | When process finishes |
| **TAT** | Turnaround Time | CT − AT |
| **WT** | Waiting Time | TAT − BT |

---

## How It Works — Step by Step

### Example

| Process | Arrival Time (AT) | Burst Time (BT) |
|---------|-------------------|-----------------|
| P1 | 0 | 7 |
| P2 | 2 | 4 |
| P3 | 4 | 1 |
| P4 | 5 | 4 |

### Trace through time

| Time | Event | Running | Remaining BTs |
|------|-------|---------|---------------|
| 0 | P1 arrives | P1 | P1=7 |
| 2 | P2 arrives (BT=4 < P1 remaining=5) → **preempt P1** | P2 | P1=5, P2=4 |
| 4 | P3 arrives (BT=1 < P2 remaining=2) → **preempt P2** | P3 | P1=5, P2=2, P3=1 |
| 5 | P3 finishes. P4 arrives (BT=4). P2 remaining=2 → **P2 wins** | P2 | P1=5, P2=2, P4=4 |
| 7 | P2 finishes. Choose min(P1=5, P4=4) → **P4 wins** | P4 | P1=5, P4=4 |
| 11 | P4 finishes → **P1 runs** | P1 | P1=5 |
| 16 | P1 finishes | — | — |

### Gantt Chart

```
| P1 | P2 | P3 | P2 | P4  | P1      |
0    2    4    5    7     11        16
```

### Calculate Metrics

| Process | AT | BT | CT | TAT = CT−AT | WT = TAT−BT |
|---------|----|----|----|-------------|-------------|
| P1 | 0 | 7 | 16 | 16 | 9 |
| P2 | 2 | 4 | 7  | 5  | 1 |
| P3 | 4 | 1 | 5  | 1  | 0 |
| P4 | 5 | 4 | 11 | 6  | 2 |

- **Average TAT** = (16 + 5 + 1 + 6) / 4 = **7.0**
- **Average WT**  = (9 + 1 + 0 + 2) / 4 = **3.0**

> Even lower than non-preemptive SJF (which gave Avg WT = 4.0)!

---

## Advantages & Disadvantages

### ✅ Advantages
- **Globally optimal** — minimizes average waiting time
- Short processes are always prioritised, giving great response time

### ❌ Disadvantages
- **Starvation** — long processes can starve indefinitely if short ones keep arriving
- **Context switch overhead** — frequent preemptions are costly in real systems
- Burst times must be known/predicted in advance

---

## Implementation Logic (Pseudocode)

```
// Simulate time unit by unit
remaining[i] = proc[i].bt for all i
current_time = 0
completed = 0

while completed < n:
    // Among arrived, unfinished processes, pick min remaining time
    // Tie-break: earliest arrival time, then smallest PID
    idx = argmin over {i : at[i] <= current_time and remaining[i] > 0} of (remaining[i], at[i], pid[i])

    if no such idx:
        current_time++   // CPU idle
        continue

    remaining[idx]--
    current_time++

    if remaining[idx] == 0:
        completed++
        proc[idx].ct  = current_time
        proc[idx].tat = proc[idx].ct - proc[idx].at
        proc[idx].wt  = proc[idx].tat - proc[idx].bt
```

---

## Implementation in C

```c
#include <stdio.h>
#include <limits.h>

typedef struct {
    int pid, at, bt, ct, tat, wt;
} Process;

void srtf(Process proc[], int n) {
    int remaining[n];
    for (int i = 0; i < n; i++) remaining[i] = proc[i].bt;

    int completed = 0, current_time = 0;

    while (completed < n) {
        int idx = -1, min_rem = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (proc[i].at <= current_time && remaining[i] > 0) {
                if (remaining[i] < min_rem ||
                   (remaining[i] == min_rem && proc[i].at < proc[idx].at) ||
                   (remaining[i] == min_rem && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_rem = remaining[i];
                    idx = i;
                }
            }
        }

        if (idx == -1) {
            current_time++;
            continue;
        }

        remaining[idx]--;
        current_time++;

        if (remaining[idx] == 0) {
            completed++;
            proc[idx].ct  = current_time;
            proc[idx].tat = proc[idx].ct  - proc[idx].at;
            proc[idx].wt  = proc[idx].tat - proc[idx].bt;
        }
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
    printf("Average WT  = %.2f\n",  total_wt  / n);
}

int main() {
    Process proc[] = {
        {1, 0, 7},
        {2, 2, 4},
        {3, 4, 1},
        {4, 5, 4}
    };
    int n = sizeof(proc) / sizeof(proc[0]);

    srtf(proc, n);
    printResults(proc, n);

    return 0;
}
```

**Compile & Run:**
```bash
gcc srtf.c -o srtf && ./srtf
```

---

## Implementation in C++

```cpp
#include <iostream>
#include <vector>
#include <climits>
#include <iomanip>
using namespace std;

struct Process {
    int pid, at, bt, ct, tat, wt;
};

void srtf(vector<Process>& proc) {
    int n = proc.size();
    vector<int> remaining(n);
    for (int i = 0; i < n; i++) remaining[i] = proc[i].bt;

    int completed = 0, current_time = 0;

    while (completed < n) {
        int idx = -1, min_rem = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (proc[i].at <= current_time && remaining[i] > 0) {
                if (remaining[i] < min_rem ||
                   (remaining[i] == min_rem && proc[i].at < proc[idx].at) ||
                   (remaining[i] == min_rem && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_rem = remaining[i];
                    idx = i;
                }
            }
        }

        if (idx == -1) { current_time++; continue; }

        remaining[idx]--;
        current_time++;

        if (remaining[idx] == 0) {
            completed++;
            proc[idx].ct  = current_time;
            proc[idx].tat = proc[idx].ct  - proc[idx].at;
            proc[idx].wt  = proc[idx].tat - proc[idx].bt;
        }
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
        {1, 0, 7},
        {2, 2, 4},
        {3, 4, 1},
        {4, 5, 4}
    };

    srtf(proc);
    printResults(proc);

    return 0;
}
```

**Compile & Run:**
```bash
g++ srtf.cpp -o srtf && ./srtf
```

---

## Implementation in Python

```python
def srtf(processes):
    """
    Shortest Remaining Time First — Preemptive SJF
    Simulates one time unit at a time.
    """
    n = len(processes)
    remaining = [p['bt'] for p in processes]
    completed = 0
    current_time = 0

    while completed < n:
        # Find arrived, unfinished process with minimum remaining time
        candidates = [
            i for i in range(n)
            if processes[i]['at'] <= current_time and remaining[i] > 0
        ]

        if not candidates:
            current_time += 1  # CPU idle
            continue

        # Tie-break: earliest arrival, then smallest PID
        idx = min(candidates, key=lambda i: (remaining[i], processes[i]['at'], processes[i]['pid']))

        remaining[idx] -= 1
        current_time += 1

        if remaining[idx] == 0:
            completed += 1
            p = processes[idx]
            p['ct']  = current_time
            p['tat'] = p['ct'] - p['at']
            p['wt']  = p['tat'] - p['bt']

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
        {'pid': 1, 'at': 0, 'bt': 7},
        {'pid': 2, 'at': 2, 'bt': 4},
        {'pid': 3, 'at': 4, 'bt': 1},
        {'pid': 4, 'at': 5, 'bt': 4},
    ]

    result = srtf(processes)
    print_results(result)
```

**Run:**
```bash
python3 srtf.py
```

---

## Expected Output

```
P     AT     BT     CT     TAT    WT
-----------------------------------
P1    0      7      16     16     9
P2    2      4      7      5      1
P3    4      1      5      1      0
P4    5      4      11     6      2

Average TAT = 7.00
Average WT  = 3.00
```

---

## Comparison: FCFS vs SJF vs SRTF

| Metric | FCFS | SJF | SRTF |
|--------|------|-----|------|
| Avg WT | 5.75 | 4.00 | 3.00 |
| Avg TAT | 11.25 | 8.00 | 7.00 |
| Preemptive? | ❌ | ❌ | ✅ |
| Starvation? | ❌ | ✅ | ✅ |

---

## Summary

| Property | Value |
|----------|-------|
| Type | Preemptive |
| Starvation | ✅ Yes |
| Globally Optimal WT | ✅ Yes |
| Context Switches | High (unit-by-unit simulation) |
| Best for | Theoretical analysis; real-world uses predicted burst times |

---

*Previous: [02 — SJF](./02_sjf.md) | Next: [04 — Round Robin](./04_round_robin.md)*
