# Frangipani

[Frangipani: A Scalable Distributed File System](http://nil.csail.mit.edu/6.824/2020/papers/thekkath-frangipani.pdf)i

*1990s scalable distributed file system that manages a collection of disks on multiple machines as a single shared pool of storage.*

MIT [Notes](https://pdos.csail.mit.edu/6.824/notes/l-frangipani.txt) [FAQ](http://nil.csail.mit.edu/6.824/2020/papers/frangipani-faq.txt)

## Background

What to digest:

- strong consistency
- cache coherence
- distributed transactions
- distributed crash recovery

Overall design:

- A decentralized **file system**, cache for performance
- Petal: block storage service;
  - what is stored: just like an ordinary hard disk file system
    - directores
    - i-node
    - file content blocks
    - free bitmaps

Scenario 1:
WS2(work station) run `ls /` or `cat /grades`
while WS1 is modifying the same inode

Scenario 2:
WS1 and WS2 concurrently try to create `/a`, `/b` under same directory

Scenario 3:
WS1 crashes while creating a file

- operation: allocate i-node, initialize i-node, update directory

## Components

**Petal**: block storage service; replicated

**Lock Server (LS)**, with one lock per **file/directory**, the locks are named by files/directories (really i-numbers)

```
  file  owner
  -----------
  x     WS1
  y     WS1
```

**Workstation (WS)** cache(store the lock status):

```
  file/dir  lock  content
  -----------------------
  x         busy  ... //using data right now
  y         idle  ... //holds lock but not using the cached data right now
```

## Solution for Scenario 1: Cache Coherence (revealing writes)

It is to guarantee linearizability AND caching
Example: WS1 changes file `z`, then WS2 reads `z`

1. WS1 changes file `z`

```
request lock  (WS1 -> LS)
LS put: owner(z) = WS1
grant lock (LS -> WS1)
(WS1: read+cache z data from Petal
       modify z locally
       when done, cached lock in state)
```

2. WS2 try read file `z`
3. Revoke Lock: **Coherence Protocol Msg**

```
    request lock  (WS2 -> LS)
    grant (LS -> WS2)
    revoke (LS -> WS1)
    (write update log of metadata to Petal)
    (write modified z to Petal)
    release lock (WS1 -> LS)
```

4. WS2 get the lock, and read `z` from Petal

notes:

- locks and rules force reads to see last write
- one optimization: Frangipani has shared read locks, as well as exclusive write locks

## Solution for Scenario 2: Atomic Transactions (concealing writes)

There is two operation for create/rename/delete a file, 1. create/rename initializes i-node, adds to directory entry. 2. rename. The challenge is to guarantee the atomicity.

Transactional file-system operations:

- operation corresponds to a system call (create file, remove file, rename, &c)
- WS acquires locks on all file system data that it will modify
- performs operation with all locks held
- only releases when finished
  - no other WS can see partially-completed operations

## Solution for Scenario 3: Crash Recovery (write-ahead logging)

What if a Frangipani workstation dies while holding locks?

- eg: dead WS had modified data in its cache
- eg: dead WS had started to write back modified data to Petal

Solution: Before writing any of op's cached blocks to Petal, first write log to Petal

- Log entry stored in Petal for recovery
- Before writing any of op's cached blocks to Petal, first write log to Petal
- if a crashed workstation has done some Petal writes for an operation, not not all. the writes can be completed from the log in Petal

What an log entry include:

- **note**: it is just for file metadata, not for file data
- log sequence number
- array of updates: block #, new version #, addr, new bytes

When WS receive lock revoke:

```
Play log in the Petal (Petal already store it) (P)
WS send the cached updated blocks to Petal (WS1-P)
Release the lock (WS1-LS)
```

Why version number is necessary: for linearizability of the metadata update

```
WS1: delete(d/f)(v1)               crash
WS2:               create(d/f)(v2)
WS3:                                  recover WS1
WS3 is recovering WS1's log -- but it doesn't look at WS2's log
```

- When WS1 delete the file, it add a log entry with v1
- when WS2 create a file with same name, the system will give it a version number v2
- so when replay the log of ws1, the v1<v2. so ignore it.
