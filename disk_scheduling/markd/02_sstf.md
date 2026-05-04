# Shortest Seek Time First (SSTF) Disk Scheduling

## Introduction
Shortest Seek Time First (SSTF) selects the disk I/O request which requires the least disk arm movement from its current position regardless of the direction. Thus, the request with the shortest seek time is selected first. This algorithm is an improvement over FCFS as it significantly reduces the total seek time.

## How it Works
1.  Let the `Request Array` represent an array storing indexes of tracks that have been requested. Let `head` be the position of the disk head.
2.  Find the positive distance of all tracks in the request array from the current head.
3.  Find a track from the requested array which has not been accessed/serviced yet and has a minimum distance from the head.
4.  Increment the total seek count with this distance.
5.  Currently accessed track becomes the new head.
6.  Go to step 2 until all requests in the request array have been serviced.

## Advantages
- **Better Performance:** The average response time decreases compared to FCFS. Total seek time is substantially reduced.
- **High Throughput:** More requests can be serviced in a given amount of time.

## Disadvantages
- **Starvation:** It may cause starvation for a request if it has a higher seek time (farther away from the head) compared to the continuously incoming new requests that are closer to the head.
- **Overhead:** The algorithm requires an overhead to calculate the seek time in advance for all pending requests to find the minimum.
- **Variance in Response Time:** High variance in response time because some requests might be serviced very quickly while others might have to wait indefinitely.

---

## Implementations

We consider a request queue `[176, 79, 34, 60, 92, 11, 41, 114]` and the initial head position at `50`.

### 1. C Implementation
```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

void SSTF(int arr[], int head, int size) {
    int seek_count = 0;
    int distance;
    int visited[size];
    for (int i = 0; i < size; i++) visited[i] = 0;
    
    printf("Seek Sequence is:\n");

    for (int i = 0; i < size; i++) {
        int min_dist = INT_MAX;
        int index = -1;
        
        // Find the unvisited track with the minimum distance
        for (int j = 0; j < size; j++) {
            if (!visited[j]) {
                distance = abs(arr[j] - head);
                if (distance < min_dist) {
                    min_dist = distance;
                    index = j;
                }
            }
        }
        
        visited[index] = 1;
        seek_count += min_dist;
        head = arr[index];
        printf("%d ", head);
    }

    printf("\nTotal number of seek operations = %d\n", seek_count);
}

int main() {
    int arr[] = { 176, 79, 34, 60, 92, 11, 41, 114 };
    int head = 50;
    int size = sizeof(arr) / sizeof(arr[0]);
    
    SSTF(arr, head, size);
    
    return 0;
}
```

### 2. C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <climits>

using namespace std;

void SSTF(vector<int>& arr, int head) {
    int seek_count = 0;
    int size = arr.size();
    vector<bool> visited(size, false);
    vector<int> seek_sequence;

    for (int i = 0; i < size; i++) {
        int min_dist = INT_MAX;
        int index = -1;

        for (int j = 0; j < size; j++) {
            if (!visited[j]) {
                int distance = abs(arr[j] - head);
                if (distance < min_dist) {
                    min_dist = distance;
                    index = j;
                }
            }
        }

        visited[index] = true;
        seek_count += min_dist;
        head = arr[index];
        seek_sequence.push_back(head);
    }

    cout << "Total number of seek operations = " << seek_count << endl;
    cout << "Seek Sequence is:" << endl;
    for (int track : seek_sequence) {
        cout << track << " ";
    }
    cout << endl;
}

int main() {
    vector<int> arr = { 176, 79, 34, 60, 92, 11, 41, 114 };
    int head = 50;
    
    SSTF(arr, head);
    
    return 0;
}
```

### 3. Python Implementation
```python
def sstf(arr, head):
    seek_count = 0
    visited = [False] * len(arr)
    seek_sequence = []

    for _ in range(len(arr)):
        min_dist = float('inf')
        index = -1
        
        for j in range(len(arr)):
            if not visited[j]:
                distance = abs(arr[j] - head)
                if distance < min_dist:
                    min_dist = distance
                    index = j
                    
        visited[index] = True
        seek_count += min_dist
        head = arr[index]
        seek_sequence.append(head)

    print(f"Total number of seek operations = {seek_count}")
    print("Seek Sequence is:")
    print(" ".join(map(str, seek_sequence)))

if __name__ == "__main__":
    arr = [176, 79, 34, 60, 92, 11, 41, 114]
    head = 50
    sstf(arr, head)
```

### 4. Java Implementation
```java
public class SSTF {

    public static void calculateSSTF(int[] arr, int head) {
        int seek_count = 0;
        int size = arr.length;
        boolean[] visited = new boolean[size];
        int[] seek_sequence = new int[size];

        for (int i = 0; i < size; i++) {
            int min_dist = Integer.MAX_VALUE;
            int index = -1;

            for (int j = 0; j < size; j++) {
                if (!visited[j]) {
                    int distance = Math.abs(arr[j] - head);
                    if (distance < min_dist) {
                        min_dist = distance;
                        index = j;
                    }
                }
            }

            visited[index] = true;
            seek_count += min_dist;
            head = arr[index];
            seek_sequence[i] = head;
        }

        System.out.println("Total number of seek operations = " + seek_count);
        System.out.println("Seek Sequence is:");
        for (int track : seek_sequence) {
            System.out.print(track + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] arr = { 176, 79, 34, 60, 92, 11, 41, 114 };
        int head = 50;
        
        calculateSSTF(arr, head);
    }
}
```
