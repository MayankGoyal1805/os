# SCAN Disk Scheduling (Elevator Algorithm)

## Introduction
In the SCAN algorithm (also known as the Elevator Algorithm), the disk arm moves in a particular direction (either towards the center or towards the outer edge) and services the requests coming in its path. After reaching the end of the disk, it reverses its direction and again services the requests arriving in its path. 

Because of its behavior of moving like an elevator, servicing requests in one direction and then reversing, it is widely referred to as the Elevator Algorithm.

## How it Works
1.  Let `Request Array` represent an array storing indexes of tracks that have been requested. Let `head` be the position of disk head, and `direction` be the direction in which the head is moving initially (e.g., 'left' or 'right').
2.  The head moves in the given direction.
3.  If the direction is left (towards track 0), it visits all requested tracks in decreasing order until it reaches 0. After reaching 0, it reverses its direction to right.
4.  If the direction is right (towards the highest track), it visits all requested tracks in increasing order until it reaches the highest track. After reaching the end, it reverses its direction to left.
5.  It continues servicing requests in the current direction until all requests are handled.

## Advantages
- **High Throughput:** Services multiple requests concurrently in a sweep.
- **Low Variance:** Provides a lower variance of response time compared to SSTF.
- **No Starvation:** Prevents starvation of requests as the head sweeps the entire disk continuously.

## Disadvantages
- **Long Waiting Time:** Requests for locations just visited by the disk arm have to wait for the arm to traverse the entire disk and come back, leading to a long waiting time for newly arrived requests behind the head.

---

## Implementations

We consider a request queue `[176, 79, 34, 60, 92, 11, 41, 114]`, head at `50`, disk size `200` (tracks `0` to `199`), and initial direction `left`.

### 1. C Implementation
```c
#include <stdio.h>
#include <stdlib.h>

// Function to sort an array
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

void SCAN(int arr[], int head, int size, int disk_size, int direction) {
    int seek_count = 0;
    int distance, cur_track;
    int left[size + 1], right[size + 1];
    int l_size = 0, r_size = 0;
    int seek_sequence[size + 2];
    int seq_size = 0;

    // Appending end values depending on direction
    if (direction == 0) // left
        left[l_size++] = 0;
    else if (direction == 1) // right
        right[r_size++] = disk_size - 1;

    for (int i = 0; i < size; i++) {
        if (arr[i] < head)
            left[l_size++] = arr[i];
        if (arr[i] > head)
            right[r_size++] = arr[i];
    }

    // Sort left and right arrays
    sort(left, l_size);
    sort(right, r_size);

    int run = 2;
    while (run--) {
        if (direction == 0) {
            for (int i = l_size - 1; i >= 0; i--) {
                cur_track = left[i];
                seek_sequence[seq_size++] = cur_track;
                distance = abs(cur_track - head);
                seek_count += distance;
                head = cur_track;
            }
            direction = 1; // reverse direction
        } else if (direction == 1) {
            for (int i = 0; i < r_size; i++) {
                cur_track = right[i];
                seek_sequence[seq_size++] = cur_track;
                distance = abs(cur_track - head);
                seek_count += distance;
                head = cur_track;
            }
            direction = 0; // reverse direction
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
    int disk_size = 200;
    int direction = 0; // 0 for left, 1 for right
    
    SCAN(arr, head, size, disk_size, direction);
    
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

void SCAN(vector<int> arr, int head, int disk_size, string direction) {
    int seek_count = 0;
    int distance, cur_track;
    vector<int> left, right;
    vector<int> seek_sequence;

    if (direction == "left")
        left.push_back(0);
    else if (direction == "right")
        right.push_back(disk_size - 1);

    for (int i = 0; i < arr.size(); i++) {
        if (arr[i] < head)
            left.push_back(arr[i]);
        if (arr[i] > head)
            right.push_back(arr[i]);
    }

    sort(left.begin(), left.end());
    sort(right.begin(), right.end());

    int run = 2;
    while (run--) {
        if (direction == "left") {
            for (int i = left.size() - 1; i >= 0; i--) {
                cur_track = left[i];
                seek_sequence.push_back(cur_track);
                distance = abs(cur_track - head);
                seek_count += distance;
                head = cur_track;
            }
            direction = "right";
        } else if (direction == "right") {
            for (int i = 0; i < right.size(); i++) {
                cur_track = right[i];
                seek_sequence.push_back(cur_track);
                distance = abs(cur_track - head);
                seek_count += distance;
                head = cur_track;
            }
            direction = "left";
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
    int disk_size = 200;
    string direction = "left";

    SCAN(arr, head, disk_size, direction);
    
    return 0;
}
```

### 3. Python Implementation
```python
def scan(arr, head, disk_size, direction):
    seek_count = 0
    left = []
    right = []
    seek_sequence = []

    if direction == "left":
        left.append(0)
    elif direction == "right":
        right.append(disk_size - 1)

    for track in arr:
        if track < head:
            left.append(track)
        if track > head:
            right.append(track)

    left.sort()
    right.sort()

    run = 2
    while run > 0:
        if direction == "left":
            for track in reversed(left):
                seek_sequence.append(track)
                distance = abs(track - head)
                seek_count += distance
                head = track
            direction = "right"
        elif direction == "right":
            for track in right:
                seek_sequence.append(track)
                distance = abs(track - head)
                seek_count += distance
                head = track
            direction = "left"
        run -= 1

    print(f"Total number of seek operations = {seek_count}")
    print("Seek Sequence is:")
    print(" ".join(map(str, seek_sequence)))

if __name__ == "__main__":
    arr = [176, 79, 34, 60, 92, 11, 41, 114]
    head = 50
    disk_size = 200
    direction = "left"
    scan(arr, head, disk_size, direction)
```

### 4. Java Implementation
```java
import java.util.*;

public class SCAN {

    public static void calculateSCAN(int[] arr, int head, int disk_size, String direction) {
        int seek_count = 0;
        int distance, cur_track;
        Vector<Integer> left = new Vector<Integer>();
        Vector<Integer> right = new Vector<Integer>();
        Vector<Integer> seek_sequence = new Vector<Integer>();

        if (direction.equals("left"))
            left.add(0);
        else if (direction.equals("right"))
            right.add(disk_size - 1);

        for (int i = 0; i < arr.length; i++) {
            if (arr[i] < head)
                left.add(arr[i]);
            if (arr[i] > head)
                right.add(arr[i]);
        }

        Collections.sort(left);
        Collections.sort(right);

        int run = 2;
        while (run-- > 0) {
            if (direction.equals("left")) {
                for (int i = left.size() - 1; i >= 0; i--) {
                    cur_track = left.get(i);
                    seek_sequence.add(cur_track);
                    distance = Math.abs(cur_track - head);
                    seek_count += distance;
                    head = cur_track;
                }
                direction = "right";
            } else if (direction.equals("right")) {
                for (int i = 0; i < right.size(); i++) {
                    cur_track = right.get(i);
                    seek_sequence.add(cur_track);
                    distance = Math.abs(cur_track - head);
                    seek_count += distance;
                    head = cur_track;
                }
                direction = "left";
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
        int disk_size = 200;
        String direction = "left";
        
        calculateSCAN(arr, head, disk_size, direction);
    }
}
```
