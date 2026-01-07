# Universal Machine: High-Performance System Emulator 
![C](https://img.shields.io/badge/C-00599C?style=for-the-badge&logo=c&logoColor=white)
![Assembly](https://img.shields.io/badge/Assembly-x86-red?style=for-the-badge)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

**A 32-bit virtual machine engineered for extreme speed and cache efficiency, capable of executing complex binary programs.**

## Academic Integrity Notice
> **Note:** To adhere to Tufts University's Academic Integrity policies regarding core Computer Science coursework, the source code (`.c`, `.h`) for this project is **kept private**.
> 
> The documentation below details the system architecture, optimization strategies, and performance benchmarks.
## Performance Benchmarks
**Objective:** Optimize the execution of the "Sandmark" benchmark (approx. 80 million instructions) by identifying and eliminating CPU bottlenecks.

| Version | Execution Time | Speedup | Key Change |
| :--- | :--- | :--- | :--- |
| **Baseline** | 23.13s | 1.0x | Initial Modular Implementation |
| **Pass 1** | 16.58s | 1.39x | Replaced `Bitpack` library with inline bitwise ops |
| **Pass 2** | 14.60s | 1.58x | Switched `uint64_t` to `uint32_t` (Cache locality) |
| **Pass 3** | 10.06s | 2.29x | Replaced Hanson `Seq_T` with raw C Arrays |
| **Pass 4** | 6.05s | 3.82x | Inlined instruction logic (Removed function overhead) |
| **Final** | **5.48s** | **4.22x** | Custom Allocator & Loop Invariant Code Motion |

## System Architecture
The machine emulates a Turing-complete 32-bit architecture featuring:
* **8 General Purpose Registers:** Implemented as a highly accessible `uint32_t` array.
* **Segmented Memory Model:** A dynamic collection of memory segments, where Segment 0 holds the instructions.
* **I/O Subsystem:** Handles ASCII input/output .

### The Execution Engine
The core loop runs on a strict **Fetch-Decode-Execute** cycle:
1.  **Fetch:** Retrieves the 32-bit instruction word from Segment 0 at the Program Counter.
2.  **Decode:** Parses the word into Opcode (4 bits) and Registers (3 bits each).
3.  **Execute:** Dispatches to one of 14 instructions (Conditional Move, Map Segment, NAND, etc.).

## Optimization Strategy
I transitioned the codebase from a modular, object-oriented C style to a "flat," hardware-sympathetic architecture. Using **Callgrind** profiling, I targeted specific bottlenecks:

### 1. Removing Abstraction Layers
* **Problem:** The initial implementation relied on generic `Seq_T` and `Stack_T` data structures. Every memory access required a function call and void pointer casting.
* **Solution:** Replaced all generic structures with **raw C arrays** (`Segment *segments`) and a manual **free-list** (`uint32_t *freeIDs`).

### 2. Instruction Decoding Optimization
* **Problem:** Using a library to extract opcodes involved complex validation logic.
* **Solution:** Replaced library calls with **inline bitwise operators** (`>>`, `&`). This reduced instruction decode time to a single CPU cycle. 

### 3. Loop Invariant Code Motion
* **Problem:** The emulator fetches instructions from "Segment 0" on every cycle. In the original code, this required a double-dereference inside the loop.
* **Solution:** Cached the pointer to Segment 0 (`seg0`) **outside** the main execution loop, drastically improving L1 cache hits. 

### 4. Custom Memory Allocator
* **Problem:** `malloc` and `free` overhead was significant during high-frequency segment mapping.
* **Solution:** Implemented a segment recycling system. Instead of freeing memory, unmapped segments are added to a "Free Array" and reused immediately for new allocation requests.

---
*Created by Kevin Lu*
