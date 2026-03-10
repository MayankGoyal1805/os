# 04 — Round Robin (RR)

## What is Round Robin?

Round Robin is the **most widely used** scheduling algorithm in time-sharing systems. Every process gets a fixed slice of CPU time called the **time quantum (TQ)**. Once a process uses up its quantum, it is **preempted** and moved to the back of the ready queue, even if it hasn't finished.

- **Type**: Preemptive  
- **Key property**: Fairness — every process gets an equal share of the CPU cyclically  
- **Designed for**: Interactive and time-sharing operating systems (used in most modern OS schedulers at their core)

---

## Key Terms

| Term | Full Form | Note |
|------|-----------|------|
| **AT** | Arrival Time | When process enters ready queue |
| **BT** | Burst Time | Total CPU time needed |
| **TQ** | Time Quantum | Max CPU slice per turn |
| **CT** | Completion Time | When process finishes entirely |
| **TAT** | Turnaround Time | CT − AT |
| **WT** | Waiting Time | TAT − BT |
| **RT** | Response Time | Time of first CPU access − AT |

---

## How Time Quantum Affects Performance

| Time Quantum | Effect |
|-------------|--------|
| Too small | Very high context switch overhead (approaches overhead of process switching) |
| Too large | Degenerates to FCFS |
| Just right | Good balance of responsiveness and throughput |
| Rule of thumb | 80% of CPU bursts should be shorter than TQ |

---

## How It Works — Step by Step

### Example (TQ = 2)

| Process | Arrival Time (AT) | Burst Time (BT) |
|---------|-------------------|-----------------|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 1 |
| P4 | 3 | 2 |

### Queue Simulation

We maintain a **ready queue**. At each step, the front process runs for min(remaining, TQ) units.

| Time | Running | Queue after slice | Remaining BTs |
|------|---------|-------------------|---------------|
| 0–2 | P1 (2 units) | [P2, P3, P4, P1] | P1=3, P2=3, P3=1, P4=2 |
| 2–4 | P2 (2 units) | [P3, P4, P1, P2] | P1=3, P2=1, P3=1, P4=2 |
| 4–5 | P3 (1 unit) → done | [P4, P1, P2] | P1=3, P2=1, P4=2 |
| 5–7 | P4 (2 units) → done | [P1, P2] | P1=3, P2=1 |
| 7–9 | P1 (2 units) | [P2, P1] | P1=1, P2=1 |
| 9–10 | P2 (1 unit) → done | [P1] | P1=1 |
| 10–11 | P1 (1 unit) → done | [] | — |

### Gantt Chart

```
| P1 | P2 | P3 | P4 | P1 | P2 | P1 |
0    2    4    5    7    9    10   11
```

### Calculate Metrics

| Process | AT | BT | CT | TAT = CT−AT | WT = TAT−BT |
|---------|----|----|----|-------------|-------------|
| P1 | 0 | 5 | 11 | 11 | 6 |
| P2 | 1 | 3 | 10 | 9  | 6 |
| P3 | 2 | 1 | 5  | 3  | 2 |
| P4 | 3 | 2 | 7  | 4  | 2 |

- **Average TAT** = (11 + 9 + 3 + 4) / 4 = **6.75**
- **Average WT**  = (6 + 6 + 2 + 2) / 4 = **4.0**

> Response time is great — P3 (shortest) gets CPU at t=4, which is much earlier than FCFS/SJF.

---

## Advantages & Disadvantages

### ✅ Advantages
- **No starvation** — every process is guaranteed CPU time
- **Great response time** for short processes in interactive systems
- Simple and fair

### ❌ Disadvantages
- **Context switch overhead** — frequent switches cost time
- **High average waiting time** if TQ is poorly chosen
- Longer TAT than SJF for the same workload

---

## Implementation Logic (Pseudocode)

```
sort processes by arrival time
add first process to ready_queue
remaining[i] = proc[i].bt
current_time = 0

while ready_queue is not empty:
    p = dequeue front of ready_queue
    run_time = min(remaining[p], TQ)
    remaining[p] -= run_time
    current_time += run_time

    // Enqueue newly arrived processes during this slice
    for each process q not yet queued:
        if q.at <= current_time:
            enqueue q

    if remaining[p] == 0:
        p.ct  = current_time
        p.tat = p.ct - p.at
        p.wt  = p.tat - p.bt
    else:
        re-enqueue p at back of ready_queue
```

---

## Implementation in C

```c
#include <stdio.h>

#define MAX 10

typedef struct {
    int pid, at, bt, ct, tat, wt;
} Process;

void roundRobin(Process proc[], int n, int tq) {
    int remaining[MAX], queue[MAX * 100];
    int front = 0, rear = 0;
    int inQueue[MAX] = {0};
    int completed = 0;
    int current_time = 0;

    for (int i = 0; i < n; i++) remaining[i] = proc[i].bt;

    // Sort by AT first
    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - i - 1; j++)
            if (proc[j].at > proc[j+1].at) {
                Process tmp = proc[j]; proc[j] = proc[j+1]; proc[j+1] = tmp;
                int t = remaining[j]; remaining[j] = remaining[j+1]; remaining[j+1] = t;
            }

    // Enqueue first arrived process
    queue[rear++] = 0;
    inQueue[0] = 1;

    while (completed < n) {
        if (front == rear) {
            // Queue empty — CPU idle, find next arriving process
            int next = -1, min_at = __INT_MAX__;
            for (int i = 0; i < n; i++)
                if (!inQueue[i] && remaining[i] > 0 && proc[i].at < min_at) {
                    min_at = proc[i].at; next = i;
                }
            current_time = proc[next].at;
            queue[rear++] = next;
            inQueue[next] = 1;
        }

        int idx = queue[front++];
        int run_time = remaining[idx] < tq ? remaining[idx] : tq;
        remaining[idx] -= run_time;
        current_time += run_time;

        // Enqueue newly arrived processes
        for (int i = 0; i < n; i++)
            if (!inQueue[i] && proc[i].at <= current_time && remaining[i] > 0) {
                queue[rear++] = i;
                inQueue[i] = 1;
            }

        if (remaining[idx] == 0) {
            proc[idx].ct  = current_time;
            proc[idx].tat = proc[idx].ct  - proc[idx].at;
            proc[idx].wt  = proc[idx].tat - proc[idx].bt;
            completed++;
        } else {
            queue[rear++] = idx;  // Re-enqueue
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
        {1, 0, 5},
        {2, 1, 3},
        {3, 2, 1},
        {4, 3, 2}
    };
    int n = sizeof(proc) / sizeof(proc[0]);
    int tq = 2;

    roundRobin(proc, n, tq);
    printResults(proc, n);

    return 0;
}
```

**Compile & Run:**
```bash
gcc rr.c -o rr && ./rr
```

---

## Implementation in C++

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>
#include <iomanip>
using namespace std;

struct Process {
    int pid, at, bt, ct, tat, wt;
};

void roundRobin(vector<Process>& proc, int tq) {
    int n = proc.size();
    sort(proc.begin(), proc.end(), [](const Process& a, const Process& b) {
        return a.at < b.at;
    });

    vector<int> remaining(n);
    for (int i = 0; i < n; i++) remaining[i] = proc[i].bt;

    queue<int> q;
    vector<bool> inQueue(n, false);

    int current_time = 0, completed = 0;
    q.push(0); inQueue[0] = true;

    while (completed < n) {
        if (q.empty()) {
            // Find next arriving process
            int next = -1, min_at = INT_MAX;
            for (int i = 0; i < n; i++)
                if (!inQueue[i] && remaining[i] > 0 && proc[i].at < min_at) {
                    min_at = proc[i].at; next = i;
                }
            current_time = proc[next].at;
            q.push(next); inQueue[next] = true;
        }

        int idx = q.front(); q.pop();
        int run_time = min(remaining[idx], tq);
        remaining[idx] -= run_time;
        current_time += run_time;

        // Enqueue newly arrived
        for (int i = 0; i < n; i++)
            if (!inQueue[i] && proc[i].at <= current_time && remaining[i] > 0) {
                q.push(i); inQueue[i] = true;
            }

        if (remaining[idx] == 0) {
            proc[idx].ct  = current_time;
            proc[idx].tat = proc[idx].ct  - proc[idx].at;
            proc[idx].wt  = proc[idx].tat - proc[idx].bt;
            completed++;
        } else {
            q.push(idx);  // Re-enqueue
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
        {1, 0, 5},
        {2, 1, 3},
        {3, 2, 1},
        {4, 3, 2}
    };
    int tq = 2;

    roundRobin(proc, tq);
    printResults(proc);

    return 0;
}
```

**Compile & Run:**
```bash
g++ rr.cpp -o rr && ./rr
```

---

## Implementation in Python

```python
from collections import deque

def round_robin(processes, tq):
    """
    Round Robin scheduling with time quantum `tq`.
    processes: list of dicts with 'pid', 'at', 'bt'
    """
    n = len(processes)
    processes.sort(key=lambda p: p['at'])

    remaining = [p['bt'] for p in processes]
    in_queue  = [False] * n
    queue     = deque()

    current_time = 0
    completed    = 0

    queue.append(0)
    in_queue[0] = True

    while completed < n:
        if not queue:
            # CPU idle — jump to next arriving process
            next_idx = min(
                (i for i in range(n) if not in_queue[i] and remaining[i] > 0),
                key=lambda i: processes[i]['at']
            )
            current_time = processes[next_idx]['at']
            queue.append(next_idx)
            in_queue[next_idx] = True

        idx = queue.popleft()
        run_time = min(remaining[idx], tq)
        remaining[idx] -= run_time
        current_time   += run_time

        # Enqueue newly arrived processes
        for i in range(n):
            if not in_queue[i] and processes[i]['at'] <= current_time and remaining[i] > 0:
                queue.append(i)
                in_queue[i] = True

        if remaining[idx] == 0:
            p = processes[idx]
            p['ct']  = current_time
            p['tat'] = p['ct'] - p['at']
            p['wt']  = p['tat'] - p['bt']
            completed += 1
        else:
            queue.append(idx)  # Re-enqueue

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
        {'pid': 3, 'at': 2, 'bt': 1},
        {'pid': 4, 'at': 3, 'bt': 2},
    ]
    result = round_robin(processes, tq=2)
    print_results(result)
```

**Run:**
```bash
python3 rr.py
```

---

## Expected Output

```
P     AT     BT     CT     TAT    WT
-----------------------------------
P1    0      5      11     11     6
P2    1      3      10     9      6
P3    2      1      5      3      2
P4    3      2      7      4      2

Average TAT = 6.75
Average WT  = 4.00
```

---

## Summary

| Property | Value |
|----------|-------|
| Type | Preemptive |
| Starvation | ❌ No (guaranteed CPU time) |
| Response Time | ✅ Excellent for interactive systems |
| Key Parameter | Time Quantum (TQ) |
| Best for | Time-sharing / interactive OS |

---

*Previous: [03 — SRTF](./03_srtf.md) | Next: [05 — Priority Scheduling](./05_priority.md)*
