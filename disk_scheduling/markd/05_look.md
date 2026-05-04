# LOOK Disk Scheduling

## Introduction
The **LOOK algorithm** is similar to the SCAN disk scheduling algorithm except for the difference that the disk arm, instead of going to the end of the disk, goes only as far as the last request to be serviced in front of the head. It then reverses its direction from there only. Thus, it prevents the extra delay which occurred due to unnecessary traversal to the end of the disk.

## How it Works
1.  Let `Request Array` represent an array storing indexes of tracks that have been requested. Let `head` be the position of the disk head, and `direction` be the direction the head is moving initially.
2.  The head moves in the given direction and services the requests as it goes.
3.  If the direction is left, it services requests in decreasing order until it reaches the request with the smallest track number.
4.  Unlike SCAN, it does not go to track 0. It stops at the lowest request and reverses its direction.
5.  If the direction is right, it services requests in increasing order until it reaches the request with the highest track number. Again, it does not go to the very end of the disk; it stops at the highest request and reverses its direction.

## Advantages
- **Better Performance than SCAN:** Provides better performance than the SCAN algorithm because it does not move towards the end of the disk if no requests are pending in that direction.
- **Lower Variance:** Variance in waiting time and response time is low.

## Disadvantages
- **Overhead:** There is an overhead of finding the end requests in advance before reversing the direction.
- **Wait time:** Like SCAN, the requests just behind the head when it reverses direction have to wait.

---

## Implementations

We consider a request queue `[176, 79, 34, 60, 92, 11, 41, 114]`, head at `50`, and initial direction `right`.

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

void LOOK(int arr[], int head, int size, int direction) {
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

    int run = 2;
    while (run--) {
        if (direction == 0) { // left
            for (int i = l_size - 1; i >= 0; i--) {
                cur_track = left[i];
                seek_sequence[seq_size++] = cur_track;
                distance = abs(cur_track - head);
                seek_count += distance;
                head = cur_track;
            }
            direction = 1; // reverse direction
        } else if (direction == 1) { // right
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
    int direction = 1; // 0 for left, 1 for right
    
    LOOK(arr, head, size, direction);
    
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

void LOOK(vector<int> arr, int head, string direction) {
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
    string direction = "right";

    LOOK(arr, head, direction);
    
    return 0;
}
```

### 3. Python Implementation
```python
def look(arr, head, direction):
    seek_count = 0
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
    direction = "right"
    look(arr, head, direction)
```

### 4. Java Implementation
```java
import java.util.*;

public class LOOK {

    public static void calculateLOOK(int[] arr, int head, String direction) {
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
        String direction = "right";
        
        calculateLOOK(arr, head, direction);
    }
}
```
