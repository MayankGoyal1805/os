# Circular-LOOK (C-LOOK) Disk Scheduling

## Introduction
The **C-LOOK algorithm** is an enhanced version of both the C-SCAN and LOOK algorithms. Like LOOK, the head moves only as far as the last request in a specific direction instead of moving to the disk's end. Like C-SCAN, after reaching the last request in the chosen direction, it does not reverse its direction; rather, it jumps to the lowest/highest request pending on the other side and continues servicing in the original direction.

This prevents the time wasted in scanning empty cylinders that have no pending requests at the extreme ends of the disk.

## How it Works
1.  Let `Request Array` represent an array storing indexes of tracks that have been requested. Let `head` be the position of the disk head.
2.  The head moves in a specific direction (e.g., towards higher track numbers).
3.  It services requests one by one until it reaches the maximum track number requested.
4.  Once the head finishes servicing the maximum requested track, it jumps directly to the lowest requested track.
5.  After jumping, it resumes moving in the same direction and services the remaining tracks.

## Advantages
- **No Unnecessary Scans:** Does not waste time moving head to the end of the disk if no requests are pending there.
- **Fairness:** Like C-SCAN, it reduces the starvation of cylinders situated at the edge of the disk.
- **Better Throughput:** Lower variance in waiting time and better throughput than C-SCAN.

## Disadvantages
- **Overhead Calculation:** There is overhead to calculate the extreme requests (lowest and highest) before the disk head movement can be reversed/jumped.

---

## Implementations

We consider a request queue `[176, 79, 34, 60, 92, 11, 41, 114]`, head at `50`, and it initially moves towards the right.

### 1. C Implementation
```c
#include <stdio.h>
#include <stdlib.h>

void sort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}

void CLOOK(int arr[], int head, int size) {
    int seek_count = 0;
    int distance, cur_track;
    int left[size], right[size];
    int l_size = 0, r_size = 0;
    int seek_sequence[size];
    int seq_size = 0;

    for (int i = 0; i < size; i++) {
        if (arr[i] < head)
            left[l_size++] = arr[i];
        if (arr[i] > head)
            right[r_size++] = arr[i];
    }

    sort(left, l_size);
    sort(right, r_size);

    // First service the requests on the right side of the head.
    for (int i = 0; i < r_size; i++) {
        cur_track = right[i];
        seek_sequence[seq_size++] = cur_track;
        distance = abs(cur_track - head);
        seek_count += distance;
        head = cur_track;
    }

    // Once reached the right end's last request, jump to the first request on the left.
    // The jump increases the seek count but we only jump to the smallest request.
    if(l_size > 0) {
        seek_count += abs(head - left[0]);
        head = left[0];

        // Now service the requests on the left side in the same direction.
        for (int i = 0; i < l_size; i++) {
            cur_track = left[i];
            seek_sequence[seq_size++] = cur_track;
            distance = abs(cur_track - head);
            seek_count += distance;
            head = cur_track;
        }
    }

    printf("Total number of seek operations = %d\n", seek_count);
    printf("Seek Sequence is:\n");
    for (int i = 0; i < seq_size; i++) {
        printf("%d ", seek_sequence[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = { 176, 79, 34, 60, 92, 11, 41, 114 };
    int head = 50;
    int size = sizeof(arr) / sizeof(arr[0]);
    
    CLOOK(arr, head, size);
    
    return 0;
}
```

### 2. C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

using namespace std;

void CLOOK(vector<int> arr, int head) {
    int seek_count = 0;
    int distance, cur_track;
    vector<int> left, right;
    vector<int> seek_sequence;

    for (int i = 0; i < arr.size(); i++) {
        if (arr[i] < head)
            left.push_back(arr[i]);
        if (arr[i] > head)
            right.push_back(arr[i]);
    }

    sort(left.begin(), left.end());
    sort(right.begin(), right.end());

    // Service right
    for (int i = 0; i < right.size(); i++) {
        cur_track = right[i];
        seek_sequence.push_back(cur_track);
        distance = abs(cur_track - head);
        seek_count += distance;
        head = cur_track;
    }

    // Jump to the smallest element
    if(!left.empty()){
        seek_count += abs(head - left[0]);
        head = left[0];

        // Service left in the same direction
        for (int i = 0; i < left.size(); i++) {
            cur_track = left[i];
            seek_sequence.push_back(cur_track);
            distance = abs(cur_track - head);
            seek_count += distance;
            head = cur_track;
        }
    }

    cout << "Total number of seek operations = " << seek_count << endl;
    cout << "Seek Sequence is:" << endl;
    for (int i = 0; i < seek_sequence.size(); i++) {
        cout << seek_sequence[i] << " ";
    }
    cout << endl;
}

int main() {
    vector<int> arr = { 176, 79, 34, 60, 92, 11, 41, 114 };
    int head = 50;

    CLOOK(arr, head);
    
    return 0;
}
```

### 3. Python Implementation
```python
def c_look(arr, head):
    seek_count = 0
    distance = 0
    left = []
    right = []
    seek_sequence = []

    for track in arr:
        if track < head:
            left.append(track)
        if track > head:
            right.append(track)

    left.sort()
    right.sort()

    for track in right:
        seek_sequence.append(track)
        distance = abs(track - head)
        seek_count += distance
        head = track

    if len(left) > 0:
        seek_count += abs(head - left[0])
        head = left[0]

        for track in left:
            seek_sequence.append(track)
            distance = abs(track - head)
            seek_count += distance
            head = track

    print(f"Total number of seek operations = {seek_count}")
    print("Seek Sequence is:")
    print(" ".join(map(str, seek_sequence)))

if __name__ == "__main__":
    arr = [176, 79, 34, 60, 92, 11, 41, 114]
    head = 50
    c_look(arr, head)
```

### 4. Java Implementation
```java
import java.util.*;

public class CLOOK {

    public static void calculateCLOOK(int[] arr, int head) {
        int seek_count = 0;
        int distance, cur_track;
        Vector<Integer> left = new Vector<Integer>();
        Vector<Integer> right = new Vector<Integer>();
        Vector<Integer> seek_sequence = new Vector<Integer>();

        for (int i = 0; i < arr.length; i++) {
            if (arr[i] < head)
                left.add(arr[i]);
            if (arr[i] > head)
                right.add(arr[i]);
        }

        Collections.sort(left);
        Collections.sort(right);

        for (int i = 0; i < right.size(); i++) {
            cur_track = right.get(i);
            seek_sequence.add(cur_track);
            distance = Math.abs(cur_track - head);
            seek_count += distance;
            head = cur_track;
        }

        if (!left.isEmpty()) {
            seek_count += Math.abs(head - left.get(0));
            head = left.get(0);

            for (int i = 0; i < left.size(); i++) {
                cur_track = left.get(i);
                seek_sequence.add(cur_track);
                distance = Math.abs(cur_track - head);
                seek_count += distance;
                head = cur_track;
            }
        }

        System.out.println("Total number of seek operations = " + seek_count);
        System.out.println("Seek Sequence is:");
        for (int i = 0; i < seek_sequence.size(); i++) {
            System.out.print(seek_sequence.get(i) + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] arr = { 176, 79, 34, 60, 92, 11, 41, 114 };
        int head = 50;
        
        calculateCLOOK(arr, head);
    }
}
```
