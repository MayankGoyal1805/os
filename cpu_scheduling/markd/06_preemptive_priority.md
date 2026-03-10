# 06 — Preemptive Priority Scheduling

## What is Preemptive Priority Scheduling?

This is the **preemptive version** of Priority Scheduling. Whenever a new process arrives, if its priority is **higher** (lower number) than the currently running process, the CPU is immediately **taken away** and given to the newcomer.

- **Type**: Preemptive  
- **Convention**: **Lower number = Higher priority**
- **Key property**: The highest-priority process always holds the CPU at every moment

---

## Key Terms

| Term | Formula |
|------|---------|
| **TAT** | CT − AT |
| **WT** | TAT − BT |
| **RT** (Response Time) | First CPU access time − AT |

---

## How It Works — Step by Step

### Example

| Process | AT | BT | Priority |
|---------|----|----|----------|
| P1 | 0 | 5 | 3 |
| P2 | 1 | 3 | 1 |
| P3 | 2 | 4 | 2 |
| P4 | 3 | 1 | 3 |

### Trace

- **t=0**: P1 starts (only option, PR=3)
- **t=1**: P2 arrives with PR=1 → **preempt P1**, P2 runs
- **t=2**: P3 arrives with PR=2, P2 has PR=1 → no preemption, P2 continues
- **t=3**: P4 arrives with PR=3, P2 has PR=1 → no preemption, P2 continues
- **t=4**: P2 finishes. Candidates: P1(PR=3,rem=4), P3(PR=2), P4(PR=3). **P3 wins**
- **t=8**: P3 finishes. P1(PR=3,rem=4), P4(PR=3). P4 arrived later → **P1 wins** (tie-break by AT)
- **t=12**: P1 finishes → **P4 runs**
- **t=13**: P4 finishes

### Gantt Chart

```
| P1 | P2  | P3    | P1    | P4 |
0    1     4       8       12  13
```

### Metrics

| Process | AT | BT | PR | CT | TAT | WT |
|---------|----|----|----|----|-----|----|
| P1 | 0 | 5 | 3 | 12 | 12 | 7 |
| P2 | 1 | 3 | 1 | 4  | 3  | 0 |
| P3 | 2 | 4 | 2 | 8  | 6  | 2 |
| P4 | 3 | 1 | 3 | 13 | 10 | 9 |

- **Average TAT** = (12+3+6+10)/4 = **7.75**
- **Average WT**  = (7+0+2+9)/4 = **4.5**

---

## Advantages & Disadvantages

### ✅ Advantages
- Highest-priority process always gets immediate CPU access
- Excellent response time for high-priority tasks
- Used in real-time systems (e.g., interrupt handlers)

### ❌ Disadvantages
- **Starvation** — low-priority processes may never run (use aging)
- **Priority inversion** — low-priority process holds a lock that a high-priority one needs

---

## Implementation Logic (Pseudocode)

```
// Simulate unit by unit (like SRTF but pick by priority, not remaining time)
remaining[i] = proc[i].bt for all i
current_time = 0

while completed < n:
    candidates = [i : proc[i].at <= current_time and remaining[i] > 0]

    if empty → current_time++; continue

    // Tie-break: earliest arrival, then smallest PID
    idx = min(candidates, key=(priority, arrival_time, pid))
    remaining[idx]--
    current_time++

    if remaining[idx] == 0:
        proc[idx].ct  = current_time
        proc[idx].tat = proc[idx].ct - proc[idx].at
        proc[idx].wt  = proc[idx].tat - proc[idx].bt
        completed++
```

---

## Implementation in C

```c
#include <stdio.h>
#include <limits.h>

typedef struct {
    int pid, at, bt, pr, ct, tat, wt;
} Process;

void preemptivePriority(Process proc[], int n) {
    int remaining[n];
    for (int i = 0; i < n; i++) remaining[i] = proc[i].bt;

    int completed = 0, current_time = 0;

    while (completed < n) {
        int idx = -1, min_pr = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (proc[i].at <= current_time && remaining[i] > 0) {
                if (proc[i].pr < min_pr ||
                   (proc[i].pr == min_pr && proc[i].at < proc[idx].at) ||
                   (proc[i].pr == min_pr && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_pr = proc[i].pr;
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

void printResults(Process proc[], int n) {
    printf("\nP\tAT\tBT\tPR\tCT\tTAT\tWT\n");
    printf("-----------------------------------------------------\n");
    float total_tat = 0, total_wt = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\t%d\n",
               proc[i].pid, proc[i].at, proc[i].bt, proc[i].pr,
               proc[i].ct, proc[i].tat, proc[i].wt);
        total_tat += proc[i].tat;
        total_wt  += proc[i].wt;
    }
    printf("\nAverage TAT = %.2f\n", total_tat / n);
    printf("Average WT  = %.2f\n",  total_wt  / n);
}

int main() {
    Process proc[] = {
        {1, 0, 5, 3},
        {2, 1, 3, 1},
        {3, 2, 4, 2},
        {4, 3, 1, 3}
    };
    int n = sizeof(proc) / sizeof(proc[0]);
    preemptivePriority(proc, n);
    printResults(proc, n);
    return 0;
}
```

**Compile & Run:**
```bash
gcc preemptive_priority.c -o pp && ./pp
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
    int pid, at, bt, pr, ct, tat, wt;
};

void preemptivePriority(vector<Process>& proc) {
    int n = proc.size();
    vector<int> remaining(n);
    for (int i = 0; i < n; i++) remaining[i] = proc[i].bt;

    int completed = 0, current_time = 0;

    while (completed < n) {
        int idx = -1, min_pr = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (proc[i].at <= current_time && remaining[i] > 0) {
                if (proc[i].pr < min_pr ||
                   (proc[i].pr == min_pr && proc[i].at < proc[idx].at) ||
                   (proc[i].pr == min_pr && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_pr = proc[i].pr;
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
    cout << "\nP\tAT\tBT\tPR\tCT\tTAT\tWT\n";
    cout << string(55, '-') << "\n";
    float total_tat = 0, total_wt = 0;
    for (const auto& p : proc) {
        cout << "P" << p.pid << "\t"
             << p.at << "\t" << p.bt << "\t" << p.pr << "\t"
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
        {1, 0, 5, 3},
        {2, 1, 3, 1},
        {3, 2, 4, 2},
        {4, 3, 1, 3}
    };
    preemptivePriority(proc);
    printResults(proc);
    return 0;
}
```

**Compile & Run:**
```bash
g++ preemptive_priority.cpp -o pp && ./pp
```

---

## Implementation in Python

```python
def preemptive_priority(processes):
    """
    Preemptive Priority Scheduling.
    Lower priority number = higher priority.
    Simulates unit by unit.
    """
    n = len(processes)
    remaining = [p['bt'] for p in processes]
    completed = 0
    current_time = 0

    while completed < n:
        candidates = [
            i for i in range(n)
            if processes[i]['at'] <= current_time and remaining[i] > 0
        ]

        if not candidates:
            current_time += 1
            continue

        # Pick highest priority (lowest number); tie-break by arrival time, then PID
        idx = min(candidates,
                  key=lambda i: (processes[i]['pr'], processes[i]['at'], processes[i]['pid']))

        remaining[idx] -= 1
        current_time += 1

        if remaining[idx] == 0:
            p = processes[idx]
            p['ct']  = current_time
            p['tat'] = p['ct'] - p['at']
            p['wt']  = p['tat'] - p['bt']
            completed += 1

    return processes


def print_results(processes):
    print(f"\n{'P':<5} {'AT':<6} {'BT':<6} {'PR':<6} {'CT':<6} {'TAT':<6} {'WT':<6}")
    print("-" * 41)
    for p in processes:
        print(f"P{p['pid']:<4} {p['at']:<6} {p['bt']:<6} {p['pr']:<6} "
              f"{p['ct']:<6} {p['tat']:<6} {p['wt']:<6}")
    n = len(processes)
    avg_tat = sum(p['tat'] for p in processes) / n
    avg_wt  = sum(p['wt']  for p in processes) / n
    print(f"\nAverage TAT = {avg_tat:.2f}")
    print(f"Average WT  = {avg_wt:.2f}")


if __name__ == "__main__":
    processes = [
        {'pid': 1, 'at': 0, 'bt': 5, 'pr': 3},
        {'pid': 2, 'at': 1, 'bt': 3, 'pr': 1},
        {'pid': 3, 'at': 2, 'bt': 4, 'pr': 2},
        {'pid': 4, 'at': 3, 'bt': 1, 'pr': 3},
    ]
    result = preemptive_priority(processes)
    print_results(result)
```

**Run:**
```bash
python3 preemptive_priority.py
```

---

## Expected Output

```
P     AT     BT     PR     CT     TAT    WT
-----------------------------------------
P1    0      5      3      12     12     7
P2    1      3      1      4      3      0
P3    2      4      2      8      6      2
P4    3      1      3      13     10     9

Average TAT = 7.75
Average WT  = 4.50
```

---

## Full Algorithm Comparison

| Algorithm | Type | Starvation | Avg WT (example) | Best For |
|-----------|------|-----------|-------------------|---------|
| FCFS | Non-Preemptive | ❌ | 5.75 | Batch, simple systems |
| SJF | Non-Preemptive | ✅ | 4.00 | Batch with known burst times |
| SRTF | Preemptive | ✅ | 3.00 | Optimal throughput |
| Round Robin | Preemptive | ❌ | 4.00 | Time-sharing / interactive |
| Priority (NP) | Non-Preemptive | ✅ | varies | Priority-driven batch |
| Priority (P) | Preemptive | ✅ | varies | Real-time, critical tasks |

---

## Summary

| Property | Value |
|----------|-------|
| Type | Preemptive |
| Starvation | ✅ Yes (aging solves it) |
| Response Time | ✅ Excellent for high-priority tasks |
| Implementation | Unit-by-unit simulation (like SRTF) |
| Best for | Real-time systems, OS interrupt handling |

---

*Previous: [05 — Priority Scheduling (NP)](./05_priority.md)*
