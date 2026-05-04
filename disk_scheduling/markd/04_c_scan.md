# Circular-SCAN (C-SCAN) Disk Scheduling

## Introduction
In the SCAN algorithm, the disk arm again scans the path that has been scanned, after reversing its direction. So, it may be possible that too many requests are waiting at the other end or there may be zero or few requests pending at the scanned area.

These situations are avoided in the **CSCAN (Circular SCAN)** algorithm. In C-SCAN, the disk arm instead of reversing its direction goes to the other end of the disk and starts servicing the requests from there. So, the disk arm moves in a circular fashion, and this algorithm is also similar to SCAN algorithm and hence it is known as Circular SCAN (C-SCAN).

## How it Works
1.  Let `Request Array` represent an array storing indexes of tracks that have been requested. Let `head` be the position of the disk head.
2.  The head moves in a specific direction (e.g., towards the right / higher track numbers).
3.  It services requests one by one until it reaches the highest cylinder on the disk.
4.  Once the head reaches the highest cylinder, it immediately jumps to the lowest cylinder (track 0) without servicing any requests on its return trip.
5.  After jumping to 0, it again starts moving in the same direction (right) and servicing the remaining requests.

## Advantages
- **More Uniform Wait Time:** Provides a more uniform waiting time compared to SCAN.
- **Fairness:** It treats cylinders evenly, rather than favoring cylinders in the middle of the disk like SCAN does.

## Disadvantages
- **Extra Seek Time:** It causes more seek movements as compared to SCAN because it goes all the way to the end of the disk and jumps back to the beginning even if there are no requests at the extremes.

---

## Implementations

We consider a request queue `[176, 79, 34, 60, 92, 11, 41, 114]`, head at `50`, disk size `200` (tracks `0` to `199`). Head moves to the right initially.

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

void CSCAN(int arr[], int head, int size, int disk_size) {
    int seek_count = 0;
    int distance, cur_track;
    int left[size + 1], right[size + 1];
    int l_size = 0, r_size = 0;
    int seek_sequence[size + 2];
    int seq_size = 0;

    // Appending end values
    // In C-SCAN moving right, we append disk_size-1 and 0
    left[l_size++] = 0;
    right[r_size++] = disk_size - 1;

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

    // Once reached the right end, jump to the left end.
    head = 0;
    // adding seek count for head moving from end to start.
    seek_count += (disk_size - 1); 

    // Now service the requests on the left side.
    for (int i = 0; i < l_size; i++) {
        cur_track = left[i];
        // skip if it's the dummy 0 we just jumped to
        if(i == 0 && cur_track == 0) continue; 
        
        seek_sequence[seq_size++] = cur_track;
        distance = abs(cur_track - head);
        seek_count += distance;
        head = cur_track;
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
    int disk_size = 200;
    
    CSCAN(arr, head, size, disk_size);
    
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

void CSCAN(vector<int> arr, int head, int disk_size) {
    int seek_count = 0;
    int distance, cur_track;
    vector<int> left, right;
    vector<int> seek_sequence;

    left.push_back(0);
    right.push_back(disk_size - 1);

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

    // Jump to the beginning
    head = 0;
    seek_count += (disk_size - 1);

    // Service left
    for (int i = 0; i < left.size(); i++) {
        cur_track = left[i];
        if (i == 0 && cur_track == 0) continue;

        seek_sequence.push_back(cur_track);
        distance = abs(cur_track - head);
        seek_count += distance;
        head = cur_track;
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
    int disk_size = 200;

    CSCAN(arr, head, disk_size);
    
    return 0;
}
```

### 3. Python Implementation
```python
def c_scan(arr, head, disk_size):
    seek_count = 0
    distance = 0
    left = []
    right = []
    seek_sequence = []

    left.append(0)
    right.append(disk_size - 1)

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

    head = 0
    seek_count += (disk_size - 1)

    for i in range(len(left)):
        track = left[i]
        if i == 0 and track == 0:
            continue
            
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
    disk_size = 200
    c_scan(arr, head, disk_size)
```

### 4. Java Implementation
```java
import java.util.*;

public class CSCAN {

    public static void calculateCSCAN(int[] arr, int head, int disk_size) {
        int seek_count = 0;
        int distance, cur_track;
        Vector<Integer> left = new Vector<Integer>();
        Vector<Integer> right = new Vector<Integer>();
        Vector<Integer> seek_sequence = new Vector<Integer>();

        left.add(0);
        right.add(disk_size - 1);

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

        head = 0;
        seek_count += (disk_size - 1);

        for (int i = 0; i < left.size(); i++) {
            cur_track = left.get(i);
            if (i == 0 && cur_track == 0) continue;
            
            seek_sequence.add(cur_track);
            distance = Math.abs(cur_track - head);
            seek_count += distance;
            head = cur_track;
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
        int disk_size = 200;
        
        calculateCSCAN(arr, head, disk_size);
    }
}
```
