Double hashing is a technique used in hash tables (or hash maps) to resolve collisions, which occur when two or more keys are mapped to the same index (or bucket) by the hash function. When a collision occurs, double hashing provides a way to find a new index (or bucket) to store the key-value pair, without having to resort to other collision resolution methods like chaining or open addressing with linear probing.

Here's how you can explain double hashing in a simple and concise way:

### "Double hashing is a collision resolution technique used in hash tables. It employs a second hash function, called the 'step hash function', to calculate a different step size for each key in case of a collision. This step size is used to probe the hash table for the next available slot to store the key-value pair."
###  in open addressing (Liner probing) insted of going to the random alternative location this finds a next best hash index


Here's a more detailed explanation:

1. The primary hash function is used to calculate the initial index (or bucket) for a key-value pair.
2. If that index is occupied (a collision occurs), the secondary hash function, or the 'step hash function', is used to calculate a step size (typically a prime number) for that specific key.
3. The step size is then used to probe the hash table in a circular manner, skipping over occupied slots until an empty slot is found to store the key-value pair.
4. The probing sequence follows this pattern: `initial_index`, `(initial_index + step_size) % table_size`, `(initial_index + 2 * step_size) % table_size`, and so on, until an empty slot is found or the entire table has been traversed.

The key advantage of double hashing is that it ensures a unique probing sequence for each key, even if collisions occur. This means that if two keys collide at the initial index, they will follow different probing sequences, reducing the chances of clustering (multiple collisions at the same index) and improving the overall distribution of key-value pairs in the hash table.

Double hashing can be more efficient than other collision resolution techniques like chaining or linear probing, especially when the load factor (the ratio of occupied slots to the total number of slots) of the hash table is high. However, it comes with the additional overhead of calculating the step size using the secondary hash function for each collision.

# Quadratic probing

Quadratic probing is a technique used in open addressing hash tables to resolve collisions when inserting or searching for elements. It is related to linear probing, but instead of using a constant step size, it uses a quadratic function to determine the next position to probe.

In linear probing, when a collision occurs, the hash table is probed sequentially, starting from the initial index and moving to the next available slot until an empty slot is found. The probing sequence follows a linear pattern: `initial_index`, `initial_index + 1`, `initial_index + 2`, and so on.

In quadratic probing, the probing sequence is determined by a quadratic function, typically of the form:

```
(initial_index + (j * j)) % table_size
```
`initial index with square of current index mod table_size`

where `j` is an iterator that starts at 0 and is incremented by 1 for each probing attempt. The `initial_index` is the index calculated by the primary hash function.

The probing sequence for quadratic probing looks like this:

```
initial_index, (initial_index + 1^2) % table_size, (initial_index + 2^2) % table_size, (initial_index + 3^2) % table_size, ...
```

The difference between quadratic probing and linear probing lies in the way they handle collisions and the order in which they probe for the next available slot.

### Linear probing is more straightforward and easier to implement, but it has a higher probability of creating clusters of occupied slots, especially when the load factor (the ratio of occupied slots to the total number of slots) is high. This clustering can lead to a phenomenon known as primary clustering, where the probing sequence degenerates to sequential searching, which is inefficient.

### Quadratic probing, on the other hand, is more complex to implement but can provide better distribution of elements in the hash table, reducing the likelihood of primary clustering and improving performance, especially when the load factor is high.

### However, quadratic probing has a limitation: if the size of the hash table is not a prime number, there is a possibility of an infinite clustering, where the probing sequence gets stuck in a cycle without finding an empty slot. To address this issue, it is recommended to use a prime number as the size of the hash table when using quadratic probing.

In summary, quadratic probing is a collision resolution technique that can provide better distribution of elements in the hash table compared to linear probing, but it requires more complex implementation and the use of a prime number as the size of the hash table to avoid infinite clustering.
