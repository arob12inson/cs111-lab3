# Hash Hash Hash
This is a thread safe hash table implementation. Specifically, it adds entries to the hash table
in a thread safe manner. I hope.

## Building
```shell
make
```

## Running
```shell
./hash-table-tester -t thread_number -s num_items
```

The hash table tester function will test the base implementation, V1, and V2 entries.
It outputs how many entries were missed (i.e. not inserted to the list), and the time
to insert all the items to the list

## First Implementation
In the `hash_table_v1_add_entry` function, I added:

1 lock for the hash table. In order to insert into the hash table, you must hold the hash table lock,
preventing others from stepping on each other's feet and losing data.

### Performance
```shell
./hash-table-tester -t 4 -s 50000
```
```
#Output1
Generation: 32,546 usec
Hash table base: 270,489 usec
  - 0 missing
Hash table v1: 505,256 usec
  - 0 missing
Hash table v2: 72,082 usec
  - 0 missing

```

```
#Output2
Generation: 26,160 usec
Hash table base: 268,675 usec
  - 0 missing
Hash table v1: 503,896 usec
  - 0 missing
Hash table v2: 70,252 usec
  - 0 missing
```

```
#Output3
Generation: 26,517 usec
Hash table base: 269,334 usec
  - 0 missing
Hash table v1: 500,732 usec
  - 0 missing
Hash table v2: 69,024 usec
  - 0 missing
```

| Implementation   | Average Time    |
|--------------- | --------------- |
|  base  |    |
| V1   | Item2.2   |
| V2   | Item2.3   |


Version 1 is slower than the base version.

This is because we effectively don't make it multithreaded, since 1 lock means there is only 1 thread
can look at the hash table, which is a very coarse grained lock. Only 1 thread is modifying the hash
table at any time, plus the overhead of locking/unlocking adds time. That is why this implementation 
is slow




## Second Implementation
In the `hash_table_v2_add_entry` function, I: 

I added a lock for each hash table entry/bucket. 

### Performance
```shell
./hash-table-tester -t 4 -50000 
```

TODO more results, speedup measurement, and analysis on v2

results:


When a thread is waiting for a lock, it isn't doing anything. In addition, these concurrent write issues happen very infrequently 
(chance that 2 threads write to the same thread << chance 2 threads write to different threads). Using these 2 assumptions, 
I wrote code that tried to prevent a thread from waiting for a lock unless absolutely necessary. The solution I cam up with 
was to give hash table entry their own lock, since most of the time 2 threads are inserting in 2 different entires anyways, we don't
want this situation to cause a thread to block/spin. 

Analogously, imagine a bathroom with 10 stalls. There are 4 people in the bathroom at some time. We don't want 2 people to use 
the same stall at once, for obvious reasons. We could lock all the stalls when one person starts using a stall, but then the 
3 people are stuck waiting for the 1 guy to finish, despite there being other unoccupied stalls. This approach is what V1's 
approach is, which explains why it is slow. Instead, it would make more sense to give each stall its own lock, so that when 
one person using stall 1 doesn't prevent someone else from using stall 2, 3, 4, etc. In the rare chance that someone tries to
enter a stall that is already in use, they won't be able to because the stall is locked. This approach is analogous to the 
approach of V2, where we give each hash table bucket their own lock.


## Cleaning up
```shell
make clean
```
