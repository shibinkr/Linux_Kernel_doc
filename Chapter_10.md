# Chapter 10: Kernel Data Structures

Linux kernel relies heavily on efficient data structures to manage resources, processes, memory, and devices. This chapter explores essential kernel data structures, their usage, performance considerations, and practical examples.

---

## 10.1 Linked Lists

**Description:** Kernel provides doubly linked lists using `struct list_head` for flexible insertion and deletion.

**Use Cases:**

* Task queues
* Device lists
* Resource tracking
* File descriptors tracking

**Example:**

```c
#include <linux/list.h>
struct my_node {
    int data;
    struct list_head list;
};
LIST_HEAD(my_list);

void add_node(int value) {
    struct my_node *new_node = kmalloc(sizeof(*new_node), GFP_KERNEL);
    new_node->data = value;
    list_add(&new_node->list, &my_list);
}

void traverse_list(void) {
    struct my_node *ptr;
    list_for_each_entry(ptr, &my_list, list) {
        printk("Data: %d\n", ptr->data);
    }
}
```

**Pros:** Simple insertion/removal, memory efficient.

**Cons:** Linear traversal is slower for large lists.

---

## 10.2 Hash Tables

**Description:** Kernel hash tables provide efficient key-based access, using `struct hlist_head` and macros like `hash_add` and `hash_for_each`.

**Use Cases:**

* Process ID mapping
* Device indexing
* Network socket tables
* Inode caching

**Example:**

```c
#include <linux/hashtable.h>
DEFINE_HASHTABLE(my_table, 4);
struct my_entry {
    int key;
    struct hlist_node node;
};

void add_entry(int k) {
    struct my_entry *e = kmalloc(sizeof(*e), GFP_KERNEL);
    e->key = k;
    hash_add(my_table, &e->node, k);
}
```

**Pros:** Fast O(1) lookup for most cases.

**Cons:** Collisions can occur; requires a good hash function.

---

## 10.3 Red-Black Trees

**Description:** Balanced binary search trees (`struct rb_root`) that maintain order with O(log n) operations.

**Use Cases:**

* Memory region tracking (vm\_area\_struct)
* Scheduler tasks
* Interval trees for device resource management

**Example:**

```c
#include <linux/rbtree.h>
struct rb_root my_tree = RB_ROOT;
struct my_node {
    int key;
    struct rb_node node;
};
```

**Pros:** Efficient for ordered data and fast search, insert, delete.

**Cons:** More complex to implement than lists.

---

## 10.4 Queues (FIFO)

**Description:** Kernel provides circular FIFO queues with `struct kfifo` for buffered sequential data.

**Use Cases:**

* Network packet buffering
* Character device input/output buffers
* Logging buffers

**Example:**

```c
#include <linux/kfifo.h>

DEFINE_KFIFO(my_fifo, int, 128);

kfifo_in(&my_fifo, &data, 1);
kfifo_out(&my_fifo, &data_out, 1);
```

**Pros:** Efficient, thread-safe FIFO management.

**Cons:** Fixed size; resizing requires careful handling.

---

## 10.5 Bitmaps

**Description:** Compact representation of boolean flags using `unsigned long` arrays. Provides fast operations for setting, clearing, and testing bits.

**Use Cases:**

* CPU/core masks
* Resource allocation tracking
* Flags for kernel features

**Example:**

```c
#include <linux/bitmap.h>

unsigned long bitmap[1];

set_bit(3, bitmap);
test_bit(3, bitmap);
clear_bit(3, bitmap);
```

**Pros:** Space-efficient, very fast bit operations.

**Cons:** Limited to bit manipulations; larger bitmaps require more memory.

---

## 10.6 Specialized Kernel Data Structures

* **Lists of lists (`hlist`)**: For hash buckets and memory efficiency.
* **RCU-protected lists**: Allow lockless read access.
* **IDR/IDA**: Efficient integer ID mapping for resources.
* **Queues (`wait_queue_head_t`)**: Synchronization primitives for blocking I/O.
* **Sparse arrays**: Efficient for large memory maps with sparse entries.

**Use Cases:**

* Device driver resource allocation
* Scheduling and task management
* File system inode and directory management
* Memory mapping and virtual memory handling

---

## 10.7 Summary

* Kernel data structures are optimized for speed, memory efficiency, and concurrency.
* Linked lists are simple and flexible.
* Hash tables provide fast key-based lookups.
* Red-black trees maintain sorted data efficiently.
* FIFOs and queues manage sequential data.
* Bitmaps efficiently track boolean flags.
* Specialized structures like RCU lists, IDRs, and wait queues provide advanced concurrency handling.

Understanding these structures is critical for kernel development, ensuring proper resource management, synchronization, and high-performance operations.
