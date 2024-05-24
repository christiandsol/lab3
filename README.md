# Hash Hash Hash
TODO introduction

## Building
```shell
make
```

## Running
```shell
./hash-table-tester -t n -s n
```
Where `n` is the number of threads and `n` is the
size of the hash table entries per thread, by default
t is 4 and s is 25000

## First Implementation
In the `hash_table_v1_add_entry` function, I added a lock 
surrounding the entirety of the hash table and locked 
the hash table before adding an entry each time. Hence each 
running thread will lock the hash table before adding an entry, 
ensuring that only one thread can access the hash table at a time.

### Performance
```shell
❯ ./hash-table-tester -t 8 -s 50000
Generation: 57,857 usec
Hash table base: 236,365 usec
  - 0 missing
Hash table v1: 653,484 usec
  - 0 missing
...

❯ ./hash-table-tester
Generation: 9,345 usec
Hash table base: 16,410 usec
  - 0 missing
Hash table v1: 58,989 usec
  - 0 missing
...

❯ ./hash-table-tester -t 3 -s 50000
Generation: 30,045 usec
Hash table base: 46,507 usec
  - 0 missing
Hash table v1: 119,286 usec
  - 0 missing
...
```
Version 1 is a little slower than the base version. This is 
because the lock is acquired and released for each entry added
to the hash table. This is a lot of overhead for the lock as
the number of threads increases (the pthread library has to 
be induced, locks have to be initialized and dealt with, and much
other reason for overhead). It is basically the base version with 
overhead. On average, it was around 2.97 times slower than the base, 
(2.56, 3.59, 2.7 respectively for each)

## Second Implementation
In the `hash_table_v2_add_entry` function, I added a lock to 
each "bucket" of the hash table. This way, each thread will
lock the bucket before adding an entry to it. This way,
threads can add entries to different buckets at the same time
without having to wait for each other. Likewise, the threads
can run in parallel and not have to wait for each other to
add entries to the hash table.

### Performance
```shell
❯ ./hash-table-tester -t 8 -s 50000
Generation: 57,857 usec
Hash table base: 236,365 usec
  - 0 missing
...
Hash table v2: 64,283 usec
  - 0 missing

❯ ./hash-table-tester
Generation: 9,345 usec
Hash table base: 16,410 usec
  - 0 missing
...
Hash table v2: 4,992 usec
  - 0 missing

❯ ./hash-table-tester -t 3 -s 50000
Generation: 30,045 usec
Hash table base: 46,507 usec
  - 0 missing
...
Hash table v2: 12,606 usec
  - 0 missing

```

Version 2 demonstrates significant performance 
improvement over both the base and v1 implementations.
By using fine-grained locking, multiple threads can operate
on different parts of the hash table simultaneously, reducing
contention and overhead. This is especially evident when 
the number of threads is increased and the number of buckets
does as well. On average, the ratio of v2/base was 0.282(with 
.272, .304, .271 respectively for each), Demontsrating a sufficiently 
large speed up.

## Cleaning up
```shell
make clean
```
