# Golang Summer of Code


## Week 1

### 1. N-Way Merge Sort
Given a `int64` array with 16M unsorted numbers, use multiple `goroutine` to sort this array. Your implementation should be faster that the Quick Sort in single thread.

**Check list:**
- Pass unit tests.
- Pass test cases.
- Analysis the bottle-neck via `Go Profile`.


## Week 2

### 1. Read [MIT MapReduce Course](https://pdos.csail.mit.edu/6.824/labs/lab-1.html) and Complete Excercise.

### 2. Report 10 Most Frequent URLs from A Huge Set of URLs

Given a file containing a huge set of URLs, use `MapReduce` and `Shuffle` to get the 10 most frequent URLs and their frequency.


## Week 3

### 1. Simplified KV Server

**Check list:**
- Max size of key is 256B, value is 4KB.
- Operations: `Set`, `Get`, `Delete` and `Scan`.
- No concurrency.
- No persistent.

## Week 4

### 1. Simplified KV Server

**Check list:**
- Add concurrency support: 1000 rps at least.
- Add persistent support: 2TB SSD disk + 32GB memory.
   
## Week 5

### 1. Read [MIT Raft Course](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html) and Complete Excercise.


## Week 6

### 1. Read [MIT KV Raft Course](https://pdos.csail.mit.edu/6.824/labs/lab-kvraft.html) and Complete Excercise.


## Misc

- http://oserror.com/
