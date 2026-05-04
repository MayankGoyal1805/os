# First Come First Serve (FCFS) Disk Scheduling

## Introduction
First-Come, First-Serve (FCFS) is the simplest of all the Disk Scheduling Algorithms. In FCFS, the requests are addressed in the order they arrive in the disk queue. This means that the first request to arrive will be the first one to be serviced by the read/write head.

## How it Works
1.  Let the `Request Array` represent an array storing indexes of tracks that have been requested, in the order of their arrival. Let `head` be the current position of the disk head.
2.  Iterate through the request array one by one.
3.  Calculate the absolute distance of the current request from the head.
4.  Add the distance to the total seek time.
5.  The currently serviced request position becomes the new head position.
6.  Repeat steps 2-5 until all requests have been serviced.

## Advantages
- **Fairness:** Every request gets a fair chance to be processed.
- **No Starvation:** There is no indefinite postponement of any request.
- **Simplicity:** It is extremely simple to understand, implement, and program.

## Disadvantages
- **Inefficiency:** It does not try to optimize the seek time.
- **Performance:** It may not provide the best possible average service time. It often results in a higher total seek time compared to other algorithms, especially if requests are scattered randomly across the disk.

---

## Implementations

For the implementations below, we consider a request queue `[176, 79, 34, 60, 92, 11, 41, 114]` and the initial head position at `50`.

### 1. C Implementation
```c
#include <stdio.h>
#include <stdlib.h>

void FCFS(int arr[], int head, int size) {
    int seek_count = 0;
    int distance, cur_track;

    for (int i = 0; i < size; i++) {
        cur_track = arr[i];
        
        // Calculate absolute distance
        distance = abs(cur_track - head);
        
        // Increase the total seek count
        seek_count += distance;
        
        // Accessed track is now the new head
        head = cur_track;
    }

    printf("Total number of seek operations = %d\n", seek_count);
    printf("Seek Sequence is:\n");
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = { 176, 79, 34, 60, 92, 11, 41, 114 };
    int head = 50;
    int size = sizeof(arr) / sizeof(arr[0]);
    
    FCFS(arr, head, size);
    
    return 0;
}
```

### 2. C++ Implementation
```cpp
#include <iostream>
#include <cmath>
#include <vector>

using namespace std;

void FCFS(vector<int>& arr, int head) {
    int seek_count = 0;
    int distance, cur_track;

    for (int i = 0; i < arr.size(); i++) {
        cur_track = arr[i];
        distance = abs(cur_track - head);
        seek_count += distance;
        head = cur_track;
    }

    cout << "Total number of seek operations = " << seek_count << endl;
    cout << "Seek Sequence is:" << endl;
    for (int i = 0; i < arr.size(); i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
}

int main() {
    vector<int> arr = { 176, 79, 34, 60, 92, 11, 41, 114 };
    int head = 50;
    
    FCFS(arr, head);
    
    return 0;
}
```

### 3. Python Implementation
```python
def fcfs(arr, head):
    seek_count = 0
    
    for cur_track in arr:
        # Calculate absolute distance
        distance = abs(cur_track - head)
        
        # Increase the total seek count
        seek_count += distance
        
        # Accessed track is now the new head
        head = cur_track

    print(f"Total number of seek operations = {seek_count}")
    print("Seek Sequence is:")
    print(" ".join(map(str, arr)))

if __name__ == "__main__":
    arr = [176, 79, 34, 60, 92, 11, 41, 114]
    head = 50
    fcfs(arr, head)
```

### 4. Java Implementation
```java
public class FCFS {

    public static void calculateFCFS(int[] arr, int head) {
        int seek_count = 0;
        int distance, cur_track;

        for (int i = 0; i < arr.length; i++) {
            cur_track = arr[i];
            distance = Math.abs(cur_track - head);
            seek_count += distance;
            head = cur_track;
        }

        System.out.println("Total number of seek operations = " + seek_count);
        System.out.println("Seek Sequence is:");
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] arr = { 176, 79, 34, 60, 92, 11, 41, 114 };
        int head = 50;
        
        calculateFCFS(arr, head);
    }
}
```
