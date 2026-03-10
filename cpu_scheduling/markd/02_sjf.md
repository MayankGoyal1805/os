# 02 — Shortest Job First (SJF)

## What is SJF?

SJF (Shortest Job First) schedules the process with the **smallest burst time** from the ready queue. Among all currently available processes, the one that will finish the quickest gets the CPU next.

- **Type**: Non-Preemptive  
- **Also known as**: SJN (Shortest Job Next), Shortest Process Next (SPN)  
- **Key property**: Gives **minimum average waiting time** among all non-preemptive algorithms (provably optimal for a static set of processes).

> ⚠️ The preemptive version of SJF is called **SRTF (Shortest Remaining Time First)** and is covered in the next file.

---

## Key Terms

| Term | Full Form | Formula |
|------|-----------|---------|
| **AT** | Arrival Time | Given |
| **BT** | Burst Time | Given |
| **CT** | Completion Time | When the process finishes |
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

### Step 1 — At each decision point, pick the shortest BT from arrived processes

- **t=0**: Only P1 has arrived → P1 runs (no choice)
- **t=7**: P1 finishes. P2, P3, P4 have all arrived. BTs are 4, 1, 4 → **P3 (BT=1) wins**
- **t=8**: P3 finishes. P2 and P4 remain. Both have BT=4 → **P2 wins** (arrived earlier, tie-break by AT)
- **t=12**: P2 finishes → **P4 runs**
- **t=16**: P4 finishes

### Step 2 — Gantt Chart

```
| P1      | P3 | P2  | P4  |
0         7    8     12    16
```

### Step 3 — Calculate Metrics

| Process | AT | BT | CT | TAT = CT−AT | WT = TAT−BT |
|---------|----|----|----|-------------|-------------|
| P1 | 0 | 7 | 7  | 7  | 0 |
| P2 | 2 | 4 | 12 | 10 | 6 |
| P3 | 4 | 1 | 8  | 4  | 3 |
| P4 | 5 | 4 | 16 | 11 | 7 |

- **Average TAT** = (7 + 10 + 4 + 11) / 4 = **8.0**
- **Average WT**  = (0 + 6 + 3 + 7) / 4 = **4.0**

> Compare this to FCFS on the same set — SJF gives noticeably lower average WT.

---

## Advantages & Disadvantages

### ✅ Advantages
- **Optimal average waiting time** for a given set of processes (non-preemptive case)
- Short processes don't get stuck behind long ones (unlike FCFS)

### ❌ Disadvantages
- **Starvation** — a process with a very long burst time may never execute if short processes keep arriving
- **Burst time must be known in advance** — in real systems, this is usually estimated (e.g., using exponential averaging)
- Not suitable for interactive systems

---

## Implementation Logic (Pseudocode)

```
sort processes by arrival time (to know who has arrived at each moment)
completed = 0
current_time = 0
visited[] = false

while completed < n:
    // Find all processes that have arrived and not yet completed
    candidates = [p for p in processes if p.at <= current_time and not visited[p]]

    if candidates is empty:
        current_time++       // CPU is idle, advance time
        continue

    // Pick the one with the smallest burst time
    // On tie: first by earliest arrival, then by smallest PID
    selected = min(candidates, key = (bt, at, pid))

    selected.ct  = current_time + selected.bt
    selected.tat = selected.ct - selected.at
    selected.wt  = selected.tat - selected.bt
    current_time = selected.ct
    visited[selected] = true
    completed++
```

---

## Implementation in C

```c
#include <stdio.h>
#include <limits.h>

typedef struct {
    int pid, at, bt, ct, tat, wt;
} Process;

void sjf(Process proc[], int n) {
    int completed = 0, current_time = 0;
    int done[n];
    for (int i = 0; i < n; i++) done[i] = 0;

    while (completed < n) {
        int idx = -1, min_bt = INT_MAX;

        // Find shortest burst among arrived, unfinished processes
        for (int i = 0; i < n; i++) {
            if (!done[i] && proc[i].at <= current_time) {
                if (proc[i].bt < min_bt ||
                   (proc[i].bt == min_bt && proc[i].at < proc[idx].at) ||
                   (proc[i].bt == min_bt && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_bt = proc[i].bt;
                    idx = i;
                }
            }
        }

        if (idx == -1) {
            current_time++;  // CPU idle
            continue;
        }

        proc[idx].ct  = current_time + proc[idx].bt;
        proc[idx].tat = proc[idx].ct  - proc[idx].at;
        proc[idx].wt  = proc[idx].tat - proc[idx].bt;
        current_time  = proc[idx].ct;
        done[idx] = 1;
        completed++;
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

    sjf(proc, n);
    printResults(proc, n);

    return 0;
}
```

**Compile & Run:**
```bash
gcc sjf.c -o sjf && ./sjf
```

---

## Implementation in C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>
#include <iomanip>
using namespace std;

struct Process {
    int pid, at, bt, ct, tat, wt;
};

void sjf(vector<Process>& proc) {
    int n = proc.size();
    int completed = 0, current_time = 0;
    vector<bool> done(n, false);

    while (completed < n) {
        int idx = -1, min_bt = INT_MAX;

        for (int i = 0; i < n; i++) {
            if (!done[i] && proc[i].at <= current_time) {
                if (proc[i].bt < min_bt ||
                   (proc[i].bt == min_bt && proc[i].at < proc[idx].at) ||
                   (proc[i].bt == min_bt && proc[i].at == proc[idx].at && proc[i].pid < proc[idx].pid)) {
                    min_bt = proc[i].bt;
                    idx = i;
                }
            }
        }

        if (idx == -1) {
            current_time++;  // CPU idle
            continue;
        }

        proc[idx].ct  = current_time + proc[idx].bt;
        proc[idx].tat = proc[idx].ct  - proc[idx].at;
        proc[idx].wt  = proc[idx].tat - proc[idx].bt;
        current_time  = proc[idx].ct;
        done[idx] = true;
        completed++;
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

    sjf(proc);
    printResults(proc);

    return 0;
}
```

**Compile & Run:**
```bash
g++ sjf.cpp -o sjf && ./sjf
```

---

## Implementation in Python

```python
import math

def sjf(processes):
    """
    processes: list of dicts with keys 'pid', 'at', 'bt'
    Non-preemptive Shortest Job First
    """
    n = len(processes)
    done = [False] * n
    current_time = 0
    completed = 0

    while completed < n:
        # Find arrived, unfinished processes
        candidates = [
            (i, p) for i, p in enumerate(processes)
            if not done[i] and p['at'] <= current_time
        ]

        if not candidates:
            current_time += 1  # CPU idle
            continue

        # Pick shortest burst; tie-break: earliest arrival, then smallest PID
        idx, selected = min(candidates, key=lambda x: (x[1]['bt'], x[1]['at'], x[1]['pid']))

        selected['ct']  = current_time + selected['bt']
        selected['tat'] = selected['ct'] - selected['at']
        selected['wt']  = selected['tat'] - selected['bt']
        current_time = selected['ct']
        done[idx] = True
        completed += 1

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

    result = sjf(processes)
    print_results(result)
```

**Run:**
```bash
python3 sjf.py
```

---

## Expected Output

```
P     AT     BT     CT     TAT    WT
-----------------------------------
P1    0      7      7      7      0
P2    2      4      12     10     6
P3    4      1      8      4      3
P4    5      4      16     11     7

Average TAT = 8.00
Average WT  = 4.00
```

---

## SJF vs FCFS (Same Dataset Comparison)

| Metric | FCFS | SJF |
|--------|------|-----|
| Avg TAT | 11.25 | 8.00 |
| Avg WT | 5.75 | 4.00 |
| Starvation? | ❌ No | ✅ Yes |
| Complexity | O(n log n) | O(n²) |

---

## Summary

| Property | Value |
|----------|-------|
| Type | Non-Preemptive |
| Starvation | ✅ Yes (long processes may starve) |
| Optimal WT | ✅ Yes (among non-preemptive algorithms) |
| Burst Known? | Must be known/estimated in advance |
| Best for | Batch systems with known job lengths |

---

*Previous: [01 — FCFS](./01_fcfs.md) | Next: [03 — SRTF (Preemptive SJF)](./03_srtf.md)*
