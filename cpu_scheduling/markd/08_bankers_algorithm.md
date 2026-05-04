# Banker's Algorithm

The Banker's Algorithm is a resource allocation and deadlock avoidance algorithm developed by Edsger Dijkstra that tests for safety by simulating the allocation of predetermined maximum possible amounts of all resources, and then makes an "s-state" check to test for possible deadlock conditions for all other pending activities, before deciding whether allocation should be allowed to continue.

## Data Structures
Let $n$ be the number of processes in the system and $m$ be the number of resource types.
*   **Available:** Vector of length $m$. If `Available[j] = k`, there are $k$ instances of resource type $R_j$ available.
*   **Max:** $n \times m$ matrix. If `Max[i, j] = k`, then process $P_i$ may request at most $k$ instances of resource type $R_j$.
*   **Allocation:** $n \times m$ matrix. If `Allocation[i, j] = k`, then $P_i$ is currently allocated $k$ instances of $R_j$.
*   **Need:** $n \times m$ matrix. If `Need[i, j] = k`, then $P_i$ may need $k$ more instances of $R_j$ to complete its task. `Need[i, j] = Max[i, j] - Allocation[i, j]`.

## Safety Algorithm
The algorithm for finding out whether or not a system is in a safe state can be described as follows:
1.  Let `Work` and `Finish` be vectors of length $m$ and $n$, respectively. Initialize:
    `Work = Available`
    `Finish[i] = false` for $i = 0, 1, ..., n-1$.
2.  Find an index $i$ such that both:
    a) `Finish[i] == false`
    b) `Need_i <= Work`
    If no such $i$ exists, go to step 4.
3.  `Work = Work + Allocation_i`
    `Finish[i] = true`
    Go to step 2.
4.  If `Finish[i] == true` for all $i$, then the system is in a safe state.

## Resource-Request Algorithm for Process $P_i$
Let $Request_i$ be the request vector for process $P_i$. If $Request_i[j] = k$, then process $P_i$ wants $k$ instances of resource type $R_j$.
1.  If $Request_i \le Need_i$, go to step 2. Otherwise, raise error condition (process exceeded maximum claim).
2.  If $Request_i \le Available$, go to step 3. Otherwise $P_i$ must wait (resources not available).
3.  Pretend to allocate requested resources to $P_i$ by modifying the state as follows:
    `Available = Available - Request_i`
    `Allocation_i = Allocation_i + Request_i`
    `Need_i = Need_i - Request_i`
    *   If safe $\Rightarrow$ the resources are allocated to $P_i$.
    *   If unsafe $\Rightarrow$ $P_i$ must wait, and the old resource-allocation state is restored.

---

## Implementations

### 1. C Implementation
```c
#include <stdio.h>
#include <stdbool.h>

int main() {
    int n, m, i, j, k;
    n = 5; // Number of processes
    m = 3; // Number of resources
    int alloc[5][3] = { { 0, 1, 0 }, { 2, 0, 0 }, { 3, 0, 2 }, { 2, 1, 1 }, { 0, 0, 2 } };
    int max[5][3] = { { 7, 5, 3 }, { 3, 2, 2 }, { 9, 0, 2 }, { 2, 2, 2 }, { 4, 3, 3 } };
    int avail[3] = { 3, 3, 2 };

    int f[n], ans[n], ind = 0;
    for (k = 0; k < n; k++) {
        f[k] = 0;
    }
    
    int need[n][m];
    for (i = 0; i < n; i++) {
        for (j = 0; j < m; j++)
            need[i][j] = max[i][j] - alloc[i][j];
    }
    
    int y = 0;
    for (k = 0; k < n; k++) {
        for (i = 0; i < n; i++) {
            if (f[i] == 0) {
                int flag = 0;
                for (j = 0; j < m; j++) {
                    if (need[i][j] > avail[j]){
                        flag = 1;
                        break;
                    }
                }
                if (flag == 0) {
                    ans[ind++] = i;
                    for (y = 0; y < m; y++)
                        avail[y] += alloc[i][y];
                    f[i] = 1;
                }
            }
        }
    }
    
    int flag = 1;
    for(int i=0;i<n;i++)
    {
      if(f[i]==0)
      {
        flag=0;
        printf("The following system is not safe\n");
        break;
      }
    }
    
    if(flag==1)
    {
      printf("Following is the SAFE Sequence\n");
      for (i = 0; i < n - 1; i++)
        printf(" P%d ->", ans[i]);
      printf(" P%d\n", ans[n - 1]);
    }
    
    return 0;
}
```

### 2. C++ Implementation
```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
    int n = 5; // Number of processes
    int m = 3; // Number of resources

    vector<vector<int>> alloc = { { 0, 1, 0 }, { 2, 0, 0 }, { 3, 0, 2 }, { 2, 1, 1 }, { 0, 0, 2 } };
    vector<vector<int>> max_need = { { 7, 5, 3 }, { 3, 2, 2 }, { 9, 0, 2 }, { 2, 2, 2 }, { 4, 3, 3 } };
    vector<int> avail = { 3, 3, 2 };

    vector<int> f(n, 0), ans(n, 0);
    int ind = 0;
    vector<vector<int>> need(n, vector<int>(m, 0));

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++)
            need[i][j] = max_need[i][j] - alloc[i][j];
    }

    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            if (f[i] == 0) {
                bool flag = false;
                for (int j = 0; j < m; j++) {
                    if (need[i][j] > avail[j]) {
                        flag = true;
                        break;
                    }
                }
                if (!flag) {
                    ans[ind++] = i;
                    for (int y = 0; y < m; y++)
                        avail[y] += alloc[i][y];
                    f[i] = 1;
                }
            }
        }
    }

    bool is_safe = true;
    for(int i=0;i<n;i++) {
        if(f[i]==0) {
            is_safe = false;
            cout << "The system is not safe" << endl;
            break;
        }
    }

    if(is_safe) {
        cout << "Following is the SAFE Sequence" << endl;
        for (int i = 0; i < n - 1; i++)
            cout << " P" << ans[i] << " ->";
        cout << " P" << ans[n - 1] << endl;
    }

    return 0;
}
```

### 3. Python Implementation
```python
def bankers_algorithm():
    n = 5  # Number of processes
    m = 3  # Number of resources

    alloc = [[0, 1, 0], [2, 0, 0], [3, 0, 2], [2, 1, 1], [0, 0, 2]]
    max_need = [[7, 5, 3], [3, 2, 2], [9, 0, 2], [2, 2, 2], [4, 3, 3]]
    avail = [3, 3, 2]

    f = [0] * n
    ans = [0] * n
    ind = 0

    need = [[0 for _ in range(m)] for _ in range(n)]
    for i in range(n):
        for j in range(m):
            need[i][j] = max_need[i][j] - alloc[i][j]

    for _ in range(n):
        for i in range(n):
            if f[i] == 0:
                flag = 0
                for j in range(m):
                    if need[i][j] > avail[j]:
                        flag = 1
                        break
                
                if flag == 0:
                    ans[ind] = i
                    ind += 1
                    for y in range(m):
                        avail[y] += alloc[i][y]
                    f[i] = 1

    is_safe = True
    for i in range(n):
        if f[i] == 0:
            is_safe = False
            print("The system is not safe")
            break

    if is_safe:
        print("Following is the SAFE Sequence")
        print(" -> ".join([f"P{i}" for i in ans]))

if __name__ == "__main__":
    bankers_algorithm()
```

### 4. Java Implementation
```java
public class BankersAlgorithm {

    public static void main(String[] args) {
        int n = 5; // Number of processes
        int m = 3; // Number of resources

        int[][] alloc = { { 0, 1, 0 }, { 2, 0, 0 }, { 3, 0, 2 }, { 2, 1, 1 }, { 0, 0, 2 } };
        int[][] max = { { 7, 5, 3 }, { 3, 2, 2 }, { 9, 0, 2 }, { 2, 2, 2 }, { 4, 3, 3 } };
        int[] avail = { 3, 3, 2 };

        int[] f = new int[n];
        int[] ans = new int[n];
        int ind = 0;
        
        int[][] need = new int[n][m];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++)
                need[i][j] = max[i][j] - alloc[i][j];
        }

        for (int k = 0; k < n; k++) {
            for (int i = 0; i < n; i++) {
                if (f[i] == 0) {
                    boolean flag = false;
                    for (int j = 0; j < m; j++) {
                        if (need[i][j] > avail[j]) {
                            flag = true;
                            break;
                        }
                    }

                    if (!flag) {
                        ans[ind++] = i;
                        for (int y = 0; y < m; y++)
                            avail[y] += alloc[i][y];
                        f[i] = 1;
                    }
                }
            }
        }

        boolean isSafe = true;
        for (int i = 0; i < n; i++) {
            if (f[i] == 0) {
                isSafe = false;
                System.out.println("The system is not safe");
                break;
            }
        }

        if (isSafe) {
            System.out.println("Following is the SAFE Sequence");
            for (int i = 0; i < n - 1; i++) {
                System.out.print("P" + ans[i] + " -> ");
            }
            System.out.println("P" + ans[n - 1]);
        }
    }
}
```
