# 05 — Priority Scheduling (Non-Preemptive)

## What is Priority Scheduling?

Each process is assigned a **priority number**. The CPU is given to the process with the **highest priority** from the ready queue. Tie-break order: **Priority → Arrival Time → PID**.

- **Type**: Non-Preemptive  
- **Convention here**: **Lower number = Higher priority** (priority 1 is most urgent)
- **Key property**: Critical processes always run first

---

## Key Terms

| Term | Full Form | Formula |
|------|-----------|---------|
| **AT** | Arrival Time | Given |
| **BT** | Burst Time | Given |
| **PR** | Priority | Given (lower = higher priority) |
| **CT** | Completion Time | When process finishes |
| **TAT** | Turnaround Time | CT − AT |
| **WT** | Waiting Time | TAT − BT |

---

## How It Works — Step by Step

### Example

| Process | AT | BT | Priority |
|---------|----|----|----------|
| P1 | 0 | 4 | 2 |
| P2 | 1 | 3 | 1 |
| P3 | 2 | 1 | 4 |
| P4 | 3 | 5 | 3 |
| P5 | 4 | 2 | 2 |

### Decisions

- **t=0**: Only P1 arrived → P1 runs (non-preemptive, no interruption)
- **t=4**: P2(PR=1), P3(PR=4), P4(PR=3) available → **P2 wins**
- **t=7**: P3(PR=4), P4(PR=3), P5(PR=2) available → **P5 wins**
- **t=9**: P3(PR=4), P4(PR=3) → **P4 wins**
- **t=14**: Only P3 → **P3 runs**

### Gantt Chart

```
| P1   | P2  | P5 | P4      | P3 |
0      4     7    9         14   15
```

### Metrics

| Process | AT | BT | PR | CT | TAT | WT |
|---------|----|----|----|----|-----|----|
| P1 | 0 | 4 | 2 | 4  | 4  | 0  |
| P2 | 1 | 3 | 1 | 7  | 6  | 3  |
| P3 | 2 | 1 | 4 | 15 | 13 | 12 |
| P4 | 3 | 5 | 3 | 14 | 11 | 6  |
| P5 | 4 | 2 | 2 | 9  | 5  | 3  |

- **Average TAT** = (4+6+13+11+5)/5 = **7.8**
- **Average WT**  = (0+3+12+6+3)/5 = **4.8**

---

## Starvation & Aging

**Starvation**: Low-priority processes may never execute if high-priority ones keep arriving.

**Aging fix**: Gradually increase priority of waiting processes over time. Every N units of waiting, bump priority by 1. Eventually every process becomes the most urgent.

---

## Advantages & Disadvantages

### ✅ Advantages
- Critical tasks guaranteed to run first
- Flexible and expressive

### ❌ Disadvantages
- **Starvation** (fixed by aging)
- **Priority inversion** (fixed by priority inheritance)
- Priorities must be assigned correctly

---

## Implementation Logic (Pseudocode)

```
while completed < n:
    candidates = arrived, unfinished processes
    if empty → current_time++; continue

    // Tie-break: first by earliest arrival, then by smallest PID
    selected = min(candidates, key=(priority, arrival_time, pid))
    selected.ct  = current_time + selected.bt
    selected.tat = selected.ct - selected.at
    selected.wt  = selected.tat - selected.bt
    current_time = selected.ct
```

---

## Implementation in C

```c
#include <stdio.h>
#include <limits.h>

typedef struct {
    int pid, at, bt, pr, ct, tat, wt;
} Process;

void priorityScheduling(Process proc[], int n) {
    int done[n];
    for (int i = 0; i < n; i++) done[i] = 0;

    int completed = 0, current_time = 0;

    while (completed < n) {
        int idx = -1, min_pr = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (!done[i] && proc[i].at <= current_time) {
                if (proc[i].pr < min_pr ||
                   (proc[i].pr == min_pr && proc[i].at < proc[idx].at) ||
                   (proc[i].pr == min_pr && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_pr = proc[i].pr;
                    idx = i;
                }
            }
        }

        if (idx == -1) { current_time++; continue; }

        proc[idx].ct  = current_time + proc[idx].bt;
        proc[idx].tat = proc[idx].ct  - proc[idx].at;
        proc[idx].wt  = proc[idx].tat - proc[idx].bt;
        current_time  = proc[idx].ct;
        done[idx] = 1;
        completed++;
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
        {1, 0, 4, 2},
        {2, 1, 3, 1},
        {3, 2, 1, 4},
        {4, 3, 5, 3},
        {5, 4, 2, 2}
    };
    int n = sizeof(proc) / sizeof(proc[0]);
    priorityScheduling(proc, n);
    printResults(proc, n);
    return 0;
}
```

**Compile & Run:**
```bash
gcc priority.c -o priority && ./priority
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

void priorityScheduling(vector<Process>& proc) {
    int n = proc.size();
    vector<bool> done(n, false);
    int completed = 0, current_time = 0;

    while (completed < n) {
        int idx = -1, min_pr = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (!done[i] && proc[i].at <= current_time) {
                if (proc[i].pr < min_pr ||
                   (proc[i].pr == min_pr && proc[i].at < proc[idx].at) ||
                   (proc[i].pr == min_pr && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_pr = proc[i].pr;
                    idx = i;
                }
            }
        }

        if (idx == -1) { current_time++; continue; }

        proc[idx].ct  = current_time + proc[idx].bt;
        proc[idx].tat = proc[idx].ct  - proc[idx].at;
        proc[idx].wt  = proc[idx].tat - proc[idx].bt;
        current_time  = proc[idx].ct;
        done[idx] = true;
        completed++;
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
        {1, 0, 4, 2},
        {2, 1, 3, 1},
        {3, 2, 1, 4},
        {4, 3, 5, 3},
        {5, 4, 2, 2}
    };
    priorityScheduling(proc);
    printResults(proc);
    return 0;
}
```

**Compile & Run:**
```bash
g++ priority.cpp -o priority && ./priority
```

---

## Implementation in Python

```python
def priority_scheduling(processes):
    """Non-Preemptive Priority Scheduling. Lower PR = higher priority."""
    n = len(processes)
    done = [False] * n
    completed = 0
    current_time = 0

    while completed < n:
        candidates = [
            (i, p) for i, p in enumerate(processes)
            if not done[i] and p['at'] <= current_time
        ]
        if not candidates:
            current_time += 1
            continue

        # Tie-break: earliest arrival, then smallest PID
        idx, selected = min(candidates, key=lambda x: (x[1]['pr'], x[1]['at'], x[1]['pid']))
        selected['ct']  = current_time + selected['bt']
        selected['tat'] = selected['ct'] - selected['at']
        selected['wt']  = selected['tat'] - selected['bt']
        current_time    = selected['ct']
        done[idx] = True
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
        {'pid': 1, 'at': 0, 'bt': 4, 'pr': 2},
        {'pid': 2, 'at': 1, 'bt': 3, 'pr': 1},
        {'pid': 3, 'at': 2, 'bt': 1, 'pr': 4},
        {'pid': 4, 'at': 3, 'bt': 5, 'pr': 3},
        {'pid': 5, 'at': 4, 'bt': 2, 'pr': 2},
    ]
    result = priority_scheduling(processes)
    print_results(result)
```

**Run:**
```bash
python3 priority.py
```

---

## Expected Output

```
P     AT     BT     PR     CT     TAT    WT
-----------------------------------------
P1    0      4      2      4      4      0
P2    1      3      1      7      6      3
P3    2      1      4      15     13     12
P4    3      5      3      14     11     6
P5    4      2      2      9      5      3

Average TAT = 7.80
Average WT  = 4.80
```

---

## Summary

| Property | Value |
|----------|-------|
| Type | Non-Preemptive |
| Starvation | ✅ Yes (use aging to fix) |
| Key Concept | Aging prevents indefinite blocking |
| Best for | Systems with clearly defined process priorities |

---

*Previous: [04 — Round Robin](./04_round_robin.md) | Next: [06 — Preemptive Priority](./06_preemptive_priority.md)*
