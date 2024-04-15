---
title: "Make deletions in arrays blazingly fast"
seoTitle: "faster deletions in arrays"
seoDescription: "Delete items in arrays faster than in linked lists"
datePublished: Sat Mar 30 2024 03:52:30 GMT+0000 (Coordinated Universal Time)
cuid: cludk3vwi000008l00zzchtl3
slug: make-deletions-in-arrays-blazingly-fast
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711737103361/9ee23b9a-1d51-48b5-ad37-5794d07209c1.webp
tags: delete, linked-list, array, dynamic-arrays, delete-node-in-a-linked-list

---

**Note:** Consider the C++ vector in place of the Python list, i.e., assume a contiguous memory layout, and items as values are stored in a dynamic array instead of item references.

If we want to delete at any location without changing the order of items in the array, the time complexity of each delete would be linear, O(N) for traversal + O(N) additionally because all the items after the deleted one have to be shifted backward by one position.

A simple solution to avoid the shifts would be to mark the items as deleted instead of actually removing them. This operation is as simple as replacing the item with an item of the same type and the one you would not expect to be present in your scenario. For example, if you are holding order\_ids as type int, you wouldn't use 0 as id so choose 0 as your item delete marker. With this single optimization, we made deletion in arrays orders of magnitudes faster than linked lists deletion even though both have the same time complexity ( traversal O(N) + deletion O(1) ). This is because arrays store items compactly and contiguously which means lots of cache hits hence blazingly fast.

With the above approach, there is one issue, memory fragmentation. If we keep deleted items forever, memory consumed by those unwanted items is wasted; the compactness of the data structure is reduced which in turn increases cache misses, impacting performance over time.

Here is how we solve the above issue problem by applying the mark and sweep algo which is used by several garbage collectors. I will provide a basic implementation of that below.

```python
# setup
item_stream = list(range(1,10_001)) # possibly new items will be added to the stream.
mark_del = 0 # any value that is not expected in the dynamic array.
item_del_count = 0
sweep_run_size = len(item_stream)//2 # ideally choose same order of magnitude as item_stream

# mimicing the receiving of the item to be processed
def request_item_generator(n):
    num = 1  # Start with the first odd number
    while num <= n:  # Generate n odd numbers
        yield num
        num += 2

item_generator = request_item_generator(len(item_stream))
while item_del_count < sweep_run_size:
    item = next(item_generator)
# -------------------------------------------------------------
# mark
    item_index = item_stream.index(item)
    item_stream[item_index] = mark_del
    item_del_count += 1
    # do some processing

# sweep
# possibly parallel to main thread to minimize latency.
if item_del_count >= sweep_run_size:
    new_item_stream = [item for item in item_stream if item != mark_del]
    # pause appending of items to the dynamic array, add them into a temp buffer.
    item_stream = new_item_stream
    print(len(item_stream))
```

There are two phases in the algorithm,  
**Mark phase:** we have already discussed this above.  
**Sweep phase**: The easiest implement goes like this, when half of the existing items are marked deleted we will copy all the non-marked items into a new array and discard the old one. If you think about it, this is exactly the inverse of how dynamic arrays grow efficiently by making append operation O(1) amortized. Here we are shrinking the array while achieving the deletion time complexity of O(1) amortized, all while maintaining all the benefits of mark phase optimization with only a little penalty.

**Optimization for Sweep Phase**: You can remove deleted items and shift the remaining items in-place instead of requesting a new memory block in the heap for the new array via a sys-call, further improving the performance.  
**Tuning Sweep Threshold**: The sweep threshold (when to compact the array) can be tuned as a percentage of the array size, allowing for experimentation to find the best-performing configuration.

This is how we get significant speedup on the deletion of items in dynamic arrays/ arrays over linked lists.