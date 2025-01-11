---
title: Coding My Own Ring Buffer in C++
date: 2025-01-10 01:20:11 +/-TTTT
categories: [Programming, cp]
tags: [c++, cp, DSA]     # TAG names should always be lowercase
description: Inspired by how `dmesg` uses a ring buffer to store kernel logs, I decided to code my own ring buffer in C++. Here's how it works!
#author: "Mohamed Anaddam"                     # for single entry
---

<br>

## What is a Ring Buffer?

A **ring buffer** (also known as a circular buffer) is a data structure that uses a fixed-size buffer as if it were connected end-to-end. It’s called a "ring" because once the buffer is full, new data overwrites the oldest data, creating a circular flow. Ring buffers are commonly used in scenarios where you need to efficiently manage a continuous stream of data, such as in real-time systems, logging, or kernel messaging.

### Key Features of Ring Buffers:
- **Fixed Size**: The buffer has a predefined size and doesn’t grow dynamically.
- **Overwrite Behavior**: When the buffer is full, new data overwrites the oldest data.
- **Efficient**: Ring buffers provide O(1) time complexity for both insertion and deletion.
- **Two Pointers**: A ring buffer typically uses two pointers:
  - **Head**: Points to the next location where data will be written.
  - **Tail**: Points to the next location where data will be read.

---

## Ring Buffers in `dmesg`

The `dmesg` command in Linux is used to display kernel ring buffer messages. The kernel uses a ring buffer to store log messages, which allows it to efficiently manage logs without running out of memory. Here's how it connects:

### Why the Kernel Uses a Ring Buffer:
1. **Real-Time Logging**: The kernel generates log messages continuously, and a ring buffer ensures that the most recent messages are always available.
2. **Memory Efficiency**: Since the buffer is fixed in size, it doesn’t consume unlimited memory. Older messages are automatically overwritten when the buffer is full.
3. **Performance**: Ring buffers provide fast O(1) access for both writing and reading logs, which is critical for the kernel’s performance.

---

## My Implementation of a Ring Buffer in C++

Inspired by `dmesg`, I decided to implement my own ring buffer in C++. Below is the code along with a video explanation.

### Video Explanation

Check out the video below for a detailed walkthrough of the implementation:

<style>
  .video-container {
    position: relative;
    width: 100%;
    padding-bottom: 56.25%; /* 16:9 aspect ratio */
    margin: 20px 0; /* Add some spacing around the video */
    overflow: hidden;
  }
  .video-container iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border: none; /* Remove the iframe border */
  }
</style>

<div class="video-container">
  <iframe src="https://www.youtube.com/embed/tCeNSaVh6Vk?si=2G2RECZ4mU6vHDA0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

---

### Source Code

Here’s the C++ implementation of the ring buffer:

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>
using namespace std;

class RingBuffer {
private:
    vector<int> buffer;
    int size, write_pointer, read_pointer;
    bool is_full;

public:
    RingBuffer(int size) : size(size), write_pointer(0), read_pointer(0), is_full(false) {
        buffer.resize(size);
    }

    void write(int data) {
        if (is_full) read_pointer = (read_pointer + 1) % size;
        buffer[write_pointer] = data;
        write_pointer = (write_pointer + 1) % size;
        is_full = (read_pointer == write_pointer);
    }

    int read() {
        if (is_empty()) throw runtime_error("Buffer is empty");
        int data = buffer[read_pointer];
        read_pointer = (read_pointer + 1) % size;
        is_full = false;
        return data;
    }

    bool is_empty() const {
        return !is_full && write_pointer == read_pointer;
    }
};

int main() {
    RingBuffer buffer(5);

    buffer.write(1);
    buffer.write(2);
    buffer.write(3);

    cout << buffer.read() << endl; // Output: 1
    cout << buffer.read() << endl; // Output: 2

    buffer.write(4);
    buffer.write(5);
    buffer.write(6);
    buffer.write(7);

    while (!buffer.is_empty()) {
        cout << buffer.read() << endl; // Output: 3, 4, 5, 6, 7
    }

    return 0;
}
```

### I Code, Hack, and Write !
