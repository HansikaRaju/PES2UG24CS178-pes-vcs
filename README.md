# PES-VCS Lab Report  
**Name:** Hansika R 
**SRN:** PES2UG24CS178

---

# Overview
This project implements a simplified version control system (**PES-VCS**) inspired by Git. It demonstrates key filesystem concepts such as:

- Content-addressable storage
- Tree-based directory structures
- Staging area (index)
- Commit history management
- Branching and garbage collection (analysis)

---

# Phase 1: Object Storage

## Objective
Implement a content-addressable object store using SHA-256 hashing.

## Implementation Summary
- Implemented `object_write`:
  - Added header (`blob/tree/commit`)
  - Computed SHA-256 hash
  - Stored objects in `.pes/objects/XX/YYY...`
  - Used atomic write (temp file → rename)

- Implemented `object_read`:
  - Read object file
  - Parsed header
  - Verified integrity using hash

## Output

### Screenshot 1A: Test Output
All object storage tests passing.

![Screenshot 1A](https://github.com/user-attachments/assets/8fd565b8-b795-44e4-8106-fc6b3abecc8d)

### Screenshot 1B: Object Store Structure
Sharded directory structure.

![Screenshot 1B](https://github.com/user-attachments/assets/d9a4d2c2-1caa-4de6-a210-1e8f7841a03c)

---

# Phase 2: Tree Objects

## Objective
Implement directory structures using tree objects.

## Implementation Summary
- Implemented `tree_from_index`
- Created hierarchical tree structure from file paths
- Stored tree objects in object store
- Ensured deterministic serialization

## Output

### Screenshot 2A: Tree Test Output

![Screenshot 2A](https://github.com/user-attachments/assets/68b340b5-57b5-4c18-9d51-b86f84536f8b)

### Screenshot 2B: Raw Tree Object (Hex Dump)

![Screenshot 2B](https://github.com/user-attachments/assets/ba9e94e1-0ddc-4af1-b9c1-aba597f0758f)

---

# Phase 3: Index (Staging Area)

## Objective
Implement staging area for tracking file changes.

## Implementation Summary
- `index_load`: Reads `.pes/index`
- `index_save`: Writes index atomically
- `index_add`: Adds files to index and object store

## Output

### Screenshot 3A: Workflow (init → add → status)

![Screenshot 3A](https://github.com/user-attachments/assets/4b27b4df-7d0d-4c71-83f7-68fdb5bf4f54)

### Screenshot 3B: Index File

![Screenshot 3B](https://github.com/user-attachments/assets/76837132-994d-4865-929f-827f2749d163)

---

# Phase 4: Commits and History

## Objective
Implement commit creation and history traversal.

## Implementation Summary
- Implemented `commit_create`
- Built tree from index
- Linked commits using parent pointer
- Updated HEAD reference

## Output

### Screenshot 4A: Commit Log

![Screenshot 4A](https://github.com/user-attachments/assets/dd1e4ef8-05cb-4c05-9334-12e4c19a3b53)

### Screenshot 4B: Object Growth

![Screenshot 4B](https://github.com/user-attachments/assets/db53d154-e7d7-4b51-a918-46cb94024b5d)

### Screenshot 4C: HEAD and Branch Reference

![Screenshot 4C](https://github.com/user-attachments/assets/898e2ca2-bbf8-462b-b29f-31c7040b59fa)

---

# Final Integration Test

All integration tests executed successfully.

![Integration Test 1](https://github.com/user-attachments/assets/8ca5a46c-c2c9-44cf-8719-90835e492388)  
![Integration Test 2](https://github.com/user-attachments/assets/831172bb-1190-4b37-ab3a-99109846bd3c)
![Integration Test 3](https://github.com/user-attachments/assets/fc8e1c02-4e9f-4477-8343-c4311630255e)

---

# Phase 5: Branching and Checkout

## Q5.1: Implementation of `pes checkout <branch>`

Steps:
1. Read `.pes/refs/heads/<branch>` → get commit hash  
2. Read commit → extract tree  
3. Restore working directory using tree  
4. Update `.pes/HEAD`  

Challenges:
- Deleting extra files
- Handling modified files
- Managing nested directories
- Updating index

---

## Q5.2: Detecting Dirty Working Directory

Steps:
1. Compare file metadata (`mtime`, `size`) with index  
2. Detect modified files  
3. Compare blob hashes between branches  
4. If modified + different → block checkout  

---

## Q5.3: Detached HEAD

- HEAD points directly to commit hash  
- New commits are not linked to any branch  
- Risk: commits may be lost  


---

# Phase 6: Garbage Collection

## Q6.1: Unreachable Object Deletion

Algorithm:
1. Start from all branch heads  
2. Traverse commits via parent pointers  
3. Traverse trees recursively  
4. Store reachable hashes in a hash set  
5. Delete objects not in set  

Estimate:
- ~100,000 commits → ~2 million objects  

---

## Q6.2: GC Race Condition

Problem:
- GC runs while commit is being created  
- New object not yet referenced  
- GC deletes it → corruption  

Solution (Git):
- Grace period (no deletion of recent objects)  
- Locking mechanism  
- Atomic writes  

---

# Conclusion

This project provided hands-on understanding of:
- How Git internally stores data
- Filesystem-level design of version control
- Efficient storage using hashing and trees

---

# Repository Structure

.pes/
├── objects/
├── refs/
│ └── heads/
├── index
└── HEAD
