# Complete Overview: How Apps Run on CPU Cores, Threads, Memory, Binaries & Memory Management

---

# Table of Contents

1. Introduction
2. How Source Code Becomes Machine Code
3. What Is a Binary?
4. What Happens When You Run a Program?
5. What Is a Process?
6. Process Memory Layout
7. Code Segment
8. Stack Memory
9. Heap Memory
10. Stack vs Heap
11. How CPU Executes Instructions
12. CPU Cores
13. Threads
14. Process vs Thread
15. How Threads Run on CPU Cores
16. Context Switching
17. Hyperthreading / SMT
18. Operating System Scheduler
19. RAM
20. CPU Cache
21. Pointers
22. Virtual Memory
23. Swap Memory
24. Memory Allocation
25. Memory Fragmentation
26. Garbage Collection
27. Go Memory Management
28. Goroutines vs Threads
29. Full App Execution Flow
30. Out of Heap Memory Error
31. Important Mental Model
32. Learning Path
33. Final Summary

---

# 1. Introduction

When you write code in a language like:

* Go
* C
* C++
* Java
* Python

that code is only human-readable text.

Computers do NOT understand:

* variables
* functions
* classes
* interfaces

CPU only understands:

* machine instructions
* binary
* electrical signals

Everything eventually becomes:

```text
Instructions + Memory Addresses
```

---

# 2. How Source Code Becomes Machine Code

Example Go program:

```go
package main

import "fmt"

func main() {
    x := 10
    fmt.Println(x)
}
```

This code must be converted into machine code.

When you run:

```bash
go build
```

The Go compiler performs multiple steps.

## Compilation Pipeline

```text
Go Source Code
      ↓
Lexer
      ↓
Parser
      ↓
AST (Abstract Syntax Tree)
      ↓
Optimization
      ↓
Assembly Code
      ↓
Machine Code
      ↓
Executable Binary
```

---

# 3. What Is a Binary?

A binary is an executable file containing:

* machine instructions
* metadata
* linked libraries
* memory layout information

Examples:

| OS      | Binary Format |
| ------- | ------------- |
| Linux   | ELF           |
| Windows | EXE           |
| macOS   | Mach-O        |

The CPU executes instructions from the binary.

Example machine instructions:

```text
MOV
ADD
CALL
JMP
```

---

# 4. What Happens When You Run a Program?

Example:

```bash
./myapp
```

The operating system does:

```text
Disk → RAM → CPU
```

## Execution Flow

```text
Executable File
       ↓
Operating System Loader
       ↓
Process Created
       ↓
Memory Allocated
       ↓
Main Thread Created
       ↓
CPU Executes Instructions
```

---

# 5. What Is a Process?

A process is:

> A running instance of a program.

Examples:

* Chrome
* VSCode
* Terminal
* Go programs

Each process has:

* its own memory
* threads
* stack
* heap
* file descriptors
* security context

Processes are isolated from each other.

---

# 6. Process Memory Layout

Typical process memory layout:

```text
High Memory
┌─────────────────┐
│ Stack           │
│                 │
├─────────────────┤
│ Heap            │
│                 │
├─────────────────┤
│ Data Segment    │
├─────────────────┤
│ Code Segment    │
└─────────────────┘
Low Memory
```

---

# 7. Code Segment

The code segment contains:

* machine instructions
* compiled executable code

Example:

```go
fmt.Println("Hello")
```

gets converted into machine instructions stored in the code segment.

The CPU fetches instructions from this region.

---

# 8. Stack Memory

The stack stores:

* function calls
* local variables
* return addresses
* temporary values

Example:

```go
func add() {
    x := 10
}
```

`x` is usually stored on the stack.

## Stack Visualization

```text
main()
 └── add()
      └── x = 10
```

Memory:

```text
┌─────────┐
│ x = 10  │
├─────────┤
│ add()   │
├─────────┤
│ main()  │
└─────────┘
```

When the function returns:

```text
add() removed
x removed
```

## Why Stack Is Fast

Stack allocation is fast because:

* memory is contiguous
* allocation is pointer movement
* cleanup is automatic

---

# 9. Heap Memory

Heap memory is used for:

* dynamic allocations
* large objects
* objects surviving after function returns

Example:

```go
func create() *int {
    x := 10
    return &x
}
```

`x` cannot stay on stack because the function ends.

So the compiler allocates it on the heap.

---

# 10. Stack vs Heap

| Stack             | Heap                       |
| ----------------- | -------------------------- |
| Very fast         | Slower                     |
| Automatic cleanup | Requires GC/manual cleanup |
| Small             | Large                      |
| Function scoped   | Dynamic lifetime           |
| Contiguous memory | Fragmented                 |

---

# 11. How CPU Executes Instructions

CPU repeatedly performs:

```text
Fetch
Decode
Execute
```

Example instructions:

```text
MOV R1, 10
ADD R1, 20
```

CPU:

1. Fetches instruction from memory
2. Decodes meaning
3. Executes operation

This happens billions of times every second.

---

# 12. CPU Cores

A CPU contains cores.

Example:

* 4-core CPU
* 8-core CPU
* 16-core CPU

Each core can independently execute instructions.

Think:

```text
1 Core = 1 Worker
```

---

# 13. Threads

A thread is:

> A sequence of instructions being executed.

Examples inside Chrome:

* UI thread
* network thread
* rendering thread
* JavaScript engine thread

A process may contain many threads.

---

# 14. Process vs Thread

| Process             | Thread                |
| ------------------- | --------------------- |
| Independent program | Worker inside process |
| Own memory          | Shared process memory |
| Heavyweight         | Lightweight           |
| Isolated            | Shares resources      |

---

# 15. How Threads Run on CPU Cores

Suppose:

* 8 threads
* 4 CPU cores

Scheduler assigns threads to cores.

Example:

```text
Core 1 → Thread A
Core 2 → Thread B
Core 3 → Thread C
Core 4 → Thread D
```

Later:

```text
Core 1 → Thread E
```

The operating system switches threads rapidly.

---

# 16. Context Switching

Switching between threads is called:

# Context Switching

The CPU saves:

* registers
* program counter
* stack pointer

Then loads another thread state.

Too many context switches reduce performance.

---

# 17. Hyperthreading / SMT

Modern CPUs may allow:

```text
1 physical core → 2 logical threads
```

This improves CPU utilization.

But it is NOT the same as having double physical cores.

---

# 18. Operating System Scheduler

The scheduler decides:

* which thread runs
* which core executes it
* how long it runs

Example:

```text
Run Thread A for 5ms
Pause
Run Thread B
```

This creates multitasking.

---

# 19. RAM

RAM stores active data:

* application memory
* stack
* heap
* instructions

RAM is temporary memory.

When power turns off, RAM data disappears.

---

# 20. CPU Cache

CPU cache is very fast memory near the CPU.

Types:

* L1 Cache
* L2 Cache
* L3 Cache

Memory speed hierarchy:

```text
Registers
   ↓
L1 Cache
   ↓
L2 Cache
   ↓
L3 Cache
   ↓
RAM
   ↓
SSD/HDD
```

---

# 21. Pointers

A pointer stores a memory address.

Example:

```go
x := 10
p := &x
```

Suppose:

```text
x stored at 0x1000
```

Then:

```text
p = 0x1000
```

Pointers allow direct memory access.

---

# 22. Virtual Memory

Programs think they have their own continuous memory.

Reality:

The OS maps:

```text
Virtual Address → Physical RAM
```

Benefits:

* security
* isolation
* easier memory management

---

# 23. Swap Memory

If RAM becomes full:

OS may move inactive memory pages to disk.

This is called:

* Swap (Linux)
* Page File (Windows)

Disk is much slower than RAM.

---

# 24. Memory Allocation

When a program needs memory:

```go
make([]int, 1000)
```

Runtime requests memory from OS.

OS allocates RAM pages.

---

# 25. Memory Fragmentation

Heap allocations may create holes.

Example:

```text
Used
Free
Used
Free
```

This is fragmentation.

Fragmentation makes large contiguous allocations harder.

---

# 26. Garbage Collection

Languages like Go use:

# Garbage Collector (GC)

GC automatically frees unused heap memory.

Example:

```go
x := new(int)
```

After `x` becomes unreachable:

```text
GC frees memory
```

---

# 27. Go Memory Management

Go runtime manages:

* heap
* garbage collector
* goroutines
* scheduler

## Escape Analysis

Go compiler decides:

* stack allocation
* heap allocation

Example stack allocation:

```go
func a() int {
    x := 10
    return x
}
```

Example heap allocation:

```go
func a() *int {
    x := 10
    return &x
}
```

---

# 28. Goroutines vs Threads

Goroutines are lightweight.

Threads are managed by OS.

Go runtime maps:

```text
Many Goroutines
       ↓
Few OS Threads
       ↓
CPU Cores
```

This is why Go handles concurrency efficiently.

---

# 29. Full App Execution Flow

```text
Your Go Code
      ↓
Compiler
      ↓
Machine Code Binary
      ↓
OS Loads Binary Into RAM
      ↓
Process Created
      ↓
Main Thread Starts
      ↓
CPU Core Executes Instructions
      ↓
Functions Use Stack
      ↓
Dynamic Data Uses Heap
      ↓
Garbage Collector Cleans Heap
      ↓
Scheduler Moves Threads Across Cores
```

---

# 30. Out of Heap Memory Error

This means:

```text
Heap memory exhausted
```

Possible reasons:

* memory leak
* huge allocations
* infinite allocations
* GC unable to free enough memory

Example:

```go
for {
    data := make([]byte, 1_000_000)
}
```

Heap continuously grows until exhausted.

---

# 31. Important Mental Model

Computers ultimately operate using:

```text
Instructions + Memory Addresses
```

High-level concepts like:

* functions
* classes
* interfaces
* objects

are abstractions created by programming languages.

CPU only understands machine instructions.

---

# 32. Recommended Learning Path

To deeply understand computers:

1. Binary numbers
2. Bits and bytes
3. Hexadecimal
4. Memory layout
5. Stack vs heap
6. Pointers
7. CPU registers
8. Assembly basics
9. Processes and threads
10. Context switching
11. Virtual memory
12. Garbage collection
13. Go runtime internals
14. Operating systems basics
15. Computer architecture

---

# 33. Final Summary

## Computer =

```text
CPU + Memory + Operating System
```

## Program =

```text
Instructions + Data
```

## Operating System Manages

* processes
* threads
* memory
* files
* networking
* scheduling

## CPU

Executes instructions.

## RAM

Stores active program data.

## Final Execution Overview

```text
Source Code
    ↓
Compiler
    ↓
Binary Executable
    ↓
OS loads into RAM
    ↓
Process created
    ↓
Threads created
    ↓
OS scheduler assigns threads
    ↓
CPU cores execute instructions
    ↓
Stack + Heap memory used
    ↓
Garbage collector cleans heap
```
