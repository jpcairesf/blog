+++
title = "Data Structures at Low Level: An In-Depth Guide"
date = 2024-10-04T15:00:00-03:00
draft = false
+++

When we write software, we rely heavily on **data structures**. At a high level, these structures help us organize, manage, and store data. But at a low level, the way these structures interact with computer memory and hardware becomes crucial for efficiency and performance.

In this post, I’ll explain the fundamental data structures from a low-level perspective, discussing how they work with memory, comparing their efficiency, and analyzing their time complexity for different use cases. The aim is to give you a solid understanding of data structures so you can make informed decisions when writing code.

> **_TL;DR:_** Data structures such as arrays, stacks, linked lists, hash tables, and trees interact with memory differently. Understanding their behavior and structure helps optimize performance and pick the right one.

---

### Why understanding data structures at a low level matters

While modern programming languages abstract a lot of the complexity away, understanding how data structures function at a low level—how they interact with memory, cache, and the CPU—can significantly impact performance. Using the wrong structure for large datasets or high-frequency operations can lead to unnecessary memory usage, cache misses, or slow access times.

Knowing in depth how data structures are stored in memory, how they access data, and how they grow or shrink can help you optimize your code, especially in systems that require low-latency or handle large volumes of data.

---

### Arrays: Sequential and Cache-Friendly

An **array** is the simplest data structure. It consists of a contiguous block of memory where elements of the same type are stored sequentially. This means every element is located next to the other, and accessing any element by index is extremely fast: just **O(1)** in time complexity.

- **Memory Interaction**: Because elements are stored contiguously, arrays have excellent **cache locality**. When the CPU loads one element into its cache, neighboring elements are loaded as well. This makes iterating through arrays very fast.
- **Operations**: Accessing an element in an array is direct. For example, given an array `arr[]`, the address of an element at index `i` is calculated by: `address = base_address + i * size_of_element`. This formula allows for **constant-time access**. Arrays excel when you need fast access by index but are inefficient when inserting or deleting elements, as this requires shifting the entire array (**O(n) complexity**).
- **Use Cases**: Arrays are ideal for fixed-size collections or when fast random access is required, such as in **lookup tables** or **buffers**.

```c
int arr[5] = {1, 2, 3, 4, 5};
// Access element at index 3
int x = arr[3]; // O(1)

// Insert element at index 2 (requires shifting elements)
int insertIndex = 2;
for (int i = 4; i > insertIndex; i--) {
    arr[i] = arr[i - 1];
}
arr[insertIndex] = 99; // O(n)

// Delete element at index 1 (requires shifting elements)
int deleteIndex = 1;
for (int i = deleteIndex; i < 4; i++) {
    arr[i] = arr[i + 1];
}
arr[4] = 0; // O(n)
```
By the way, when your favorite programming language allows you to create an array without specifying its size, it's not a "real" array in the traditional sense. Behind the scenes, it likely uses a dynamic array implementation. Initially, it allocates a fixed-size array based on the data type of the elements. As you add more elements and exceed the current capacity, the language automatically creates a larger array and copies the existing elements over. This resizing process can impact performance, especially if it happens frequently.

### Stacks: LIFO and Memory Efficient
A stack is a linear data structure that follows the Last In, First Out (LIFO) principle. The most recent item added is the first one to be removed. Stacks are often used for managing function calls, undo operations, and recursive algorithms.

- **Memory Interaction**: A stack is often implemented using arrays or linked lists. However, in low-level operations, the CPU also uses a call stack for managing function calls. The stack pointer register in the CPU keeps track of the top of the stack. Recursion stacks are considered into space complexity of algorithms.
- **Operations**: Operations like push (adding an element) and pop (removing an element) are **O(1)** because they only affect the top of the stack. In assembly, pushing an element means moving the stack pointer and adding the value. Popping means reading the top value and then moving the stack pointer back. These actions directly use CPU instructions like PUSH and POP.
- **Use Cases**: Stacks are used in memory management (function call stacks), depth-first search (DFS) algorithms, and reversing sequences.
```c
stack[++top] = value;  // Push operation
value = stack[top--];  // Pop operation
```
> **Homework**: compare stacks built using arrays and linked lists

### Linked Lists: Dynamic but Less Cache-Friendly
A linked list consists of nodes, where each node contains data and a reference (or pointer) to the next node in the list. Unlike arrays, linked lists don’t require contiguous memory, allowing them to grow and shrink dynamically.

- **Memory Interaction**: Since the nodes are scattered in memory, linked lists do not benefit from cache locality. This means that accessing nodes in sequence can result in cache misses, making them slower than arrays for iteration.
- **Operations**: Linked lists are good for inserting and deleting elements if you already know where the node is (**O(1)**), but slower for accessing by index (**O(n)**) since you have to go through the list. Each node points to the next one. To insert a new node, you just change the current node's pointer to the new one, and the new node's pointer to the next in the list.
- **Use case**: Linked lists are ideal when you need frequent insertions or deletions, such as event queues or memory management systems.
```c
struct Node {
    int data;
    struct Node* next;
};

// Insert a new node after a given node (O(1))
void insertAfter(struct Node* prev_node, int new_data) {
    struct Node* new_node = (struct Node*) malloc(sizeof(struct Node));
    new_node->data = new_data;
    new_node->next = prev_node->next;
    prev_node->next = new_node;
}

// Delete a node with a given key (O(n))
void deleteNode(struct Node** head_ref, int key) {
    struct Node* temp = *head_ref, *prev;
    // If the head node holds the key (O(1))
    if (temp != NULL && temp->data == key) {
        *head_ref = temp->next;
        free(temp);
        return;
    }
    // Search for the key to be deleted (O(n))
    while (temp != NULL && temp->data != key) {
        prev = temp;
        temp = temp->next;
    }
    // Unlink the node and free memory (O(1))
    if (temp != NULL) {
        prev->next = temp->next;
        free(temp);
    }
}
```
### Hash Tables: Fast Access with Memory Overhead
A hash table stores key-value pairs and uses a hash function to map keys to an index in an underlying array. Ideally, hash tables provide **O(1)** average time complexity for insertions, deletions, and lookups.

- **Memory Interaction**: Hash tables use an array for storage, with each index acting as a "bucket" that holds one or more key-value pairs. When two keys hash to the same index, a collision occurs, and additional memory is required to store the extra entries. This may involve linked lists (chaining) to hold multiple values in one bucket, or open addressing, where the algorithm searches for another open slot in the array. This flexibility allows hash tables to dynamically grow, but collisions can increase memory overhead.
- **Operations**: In an ideal case, insertions, deletions, and lookups are O(1), thanks to direct access via the hash function. However, in the worst case, if many keys hash to the same index (for example, due to poor distribution in the hash function), multiple elements may be placed in the same bucket. When all elements hash to the same index, the time complexity for operations degrades to O(n), as you have to traverse a linked list within the bucket to find or update the correct entry. This scenario can be mitigated by using a high-quality hash function and keeping the load factor low.
- **Use case**: Hash tables are perfect for fast lookups in cases like dictionaries, caches, or indexing, where constant-time access is prioritized.
```c
struct Entry {
    int key;
    int value;
};

// Insert into a hash table bucket (average O(1), worst-case O(n) due to collisions)
bucket[index].push_back(Entry{key, value});

// Search for a key in the hash table (average O(1), worst-case O(n) due to collisions)
for (Entry entry : bucket[index]) {
    if (entry.key == key) {
        return entry.value;
    }
}

// Delete a key from the hash table (average O(1), worst-case O(n) due to collisions)
bucket[index].remove_if([key](Entry entry) { return entry.key == key; });
```
> **Homework**: look for a full hash table implementation in C and see how bucket collisions are handled

### Trees: Hierarchical with Balanced Efficiency
A tree is a hierarchical data structure where each node has a value and pointers to its child nodes. The most common tree is the binary search tree (BST), where each node has two children, and the left child is less than the parent node, while the right child is greater.

- **Memory Interaction**: Like linked lists, trees are scattered in memory and do not benefit much from cache locality. However, balanced trees (like AVL trees or red-black trees) ensure that no branch grows too long, preventing performance degradation.
- **Operations**: Balanced trees offer **O(log n)** time complexity for insertions, deletions, and lookups, making them more efficient than linked lists for ordered data. In a binary search tree, inserting a node involves traversing the tree from the root to find the correct position, ensuring the tree remains ordered. Balancing the tree involves rotation operations, which are simple pointer manipulations. A BST can also be traversed in different orders: in-order, pre-order and post-order.
- **Use case**: Trees are used in database indexing (B-trees), priority queues (binary heaps), and any system requiring ordered data retrieval.

```c
struct TreeNode {
    int data;
    struct TreeNode* left;
    struct TreeNode* right;
};

// In-order Traversal (Left, Root, Right)
void inorderTraversal(struct TreeNode* root) {
    if (root != NULL) {
        inorderTraversal(root->left);   // Visit left subtree
        printf("%d ", root->data);      // Visit node
        inorderTraversal(root->right);  // Visit right subtree
    }
}

// Pre-order Traversal (Root, Left, Right)
void preorderTraversal(struct TreeNode* root) {
    if (root != NULL) {
        printf("%d ", root->data);      // Visit node
        preorderTraversal(root->left);   // Visit left subtree
        preorderTraversal(root->right);  // Visit right subtree
    }
}

// Post-order Traversal (Left, Right, Root)
void postorderTraversal(struct TreeNode* root) {
    if (root != NULL) {
        postorderTraversal(root->left);   // Visit left subtree
        postorderTraversal(root->right);  // Visit right subtree
        printf("%d ", root->data);        // Visit node
    }
}
```

### Choosing the Right Data Structure
Choosing the right data structure depends on the problem you are solving. Always keep in mind the time complexity, advantages and disadvantages of each one.

| Data Structure | 	Access	  | Insertion | 	Deletion |  	Search  |
|----------------|:---------:|:---------:|:---------:|:---------:|
| Array          |   	O(1)   |   	O(n)   |   	O(n)   |   	O(n)   |
| Stack          |   	O(n)   |   	O(1)   |   	O(1)   |   	O(n)   |
| Linked List    |   	O(n)   |   	O(1)   |   	O(1)   |   	O(n)   |
| Hash Table     |   	O(1)   |  	O(1)	   |   O(1)    |   	O(1)   |
| Binary Tree    | 	O(log n) | 	O(log n) | 	O(log n) | 	O(log n) |

**Array**: When you need fast random access and know the size of the data.
**Stack**: When you need to manage last-in, first-out operations, especially for managing program execution and recursion.
**Linked List**: When you need dynamic resizing and frequent insertions or deletions.
**Hash Table**: When you need fast lookups, especially with key-value pairs.
**Tree**: When you need a balanced structure for ordered data and efficient range queries.

Understanding the low-level interactions between these data structures and memory helps you write more efficient code. Whether it's choosing a structure that leverages cache locality or ensuring that insertion times remain low, the right choice can make a significant impact on your software's performance.