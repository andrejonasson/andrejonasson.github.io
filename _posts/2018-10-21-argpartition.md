---
layout: post
title: Efficiently choosing `k` greatest elements (`argpartition`)
---
## Naive approach: Argsort
Say you want to get the top $$k$$  elements in an array of $$n$$  elements. A simple approach would involve sorting the array and slicing the last $$k$$  elements.

```python
import numpy as np

array = np.array([0.2, 0.1, 0.4, 0.025, 0.0125, 0.25, 0.0125])

def sorted_top_k(array, k):
  " Returns top k elements from an array of n elements in O(n log n) "
  sorted_top_k_idx = np.argsort(array)[-k:]
  return array[sorted_top_k_idx]

sorted_top_3_values = sorted_top_k(array, 3)  # array([0.2 , 0.25, 0.4 ])
```
This approach consumes $$O(n \log n)$$  operations by sorting the array. In circumstances when $$n$$  is large, it can be wasteful to sort the whole array since all we really need is the top $$k$$  elements for some specific $$k$$ .

## Argpartition
An algorithm that achieves this is introselect. It finishes when all elements greater than the $$k$$ th greatest element occupies indices greater than $$k$$  and the $$k$$ th greatest element is in its final sorted order. With an implementation that output an array in this manner we could just slice the last $$k$$  elements off and achieve our goal without resorting to sorting the whole array. The `numpy` function `argpartition` lets us do just that and more.
 
```python
>>> array = np.array([0.2, 0.1, 0.4, 0.025, 0.0125, 0.25, 0.0125])

# Second greatest element in final sorted order, 
# with <= to the left and >= to the right
>>> array[np.argpartition(array, 1)]
array([0.0125, 0.0125, 0.4   , 0.025 , 0.2   , 0.25  , 0.1   ])

# half sorted array
>>> array[np.argpartition(array, np.arange(len(array) // 2))]
array([0.0125, 0.0125, 0.025 , 0.4   , 0.2   , 0.25  , 0.1   ])

# `np.argpartition` allows for negative indices
>> np.argpartition(array, [len(array)-2, len(array)-1]) == np.argpartition(array, [-2, -1])
array([ True,  True,  True,  True,  True,  True,  True])

# first and last element in their final sorted order
>>> array[np.argpartition(array, [0, -1])]
array([0.0125, 0.1   , 0.0125, 0.025 , 0.2   , 0.25  , 0.4   ])
```

If you pass an array and an index $$k$$  to `argpartition` and then rearrange the original array with the array of indices returned by the function, then all elements smaller than or equal to the $$k$$ th greatest element occupies indices less than $$k$$  and therefore the $$k$$ th greatest element is in its final sorted order. If you instead pass an array of indices $$k_1 ,..., k_m$$  this property holds for all $$m$$  indices.

Another way of putting it, after applying the function `argpartition` with a single argument $$k$$  to some array $$\{a_{i}\}_{i=1}^{n}$$  it will return an array of indices $$\{i_j\}_{j=1}^n$$  such that the rearranged array $$\{a_{i_j}\}_{j=1}^n$$  is in an order that satisfy $$a_k \ge a_i$$  for all $$1 \le i \lt k$$  and $$a_k \le a_i$$  for all $$k \lt i \le n$$ . If instead an array of indices $$\{k_i\}_{i=1}^m$$  is passed to the function then the property will hold for all $$k$$  in $$\{k_i\}_{i=1}^m$$ .

Returning to our initial task, we can select the top $$k$$  elements efficiently by using `argpartition`
```python
array = np.array([0.2, 0.1, 0.4, 0.025, 0.0125, 0.25, 0.0125])

def top_k(array, k):
  " Returns top k elements from an array of n elements in O(n) "
  top_k_idx = np.argpartition(array, -k)[-k:]
  return array[top_k_idx]

top_3_values = top_k(array, 3)  # array([0.2 , 0.25, 0.4 ])
# NOTE: it happens to be sorted, but it is not guaranteed
```
This approach consumes only $$O(n)$$  operations to find the top $$k$$  elements.

``` python
def sorted_top_k(array, k):
  " Returns top k elements in sorted order from an array of n elements in O(n + k log k) "
  return top_k(array, k).sort()


def sorted_top_k_alternative(array, k):
  " Returns top k elements in sorted order from an array of n elements in O(n + k log k) "
  top_k_idx = np.argpartition(array, range(-k, 0))[-k:]
  return array[top_k_idx]
```
 To achieve a sorted array simply sort after or during the `argpartition`. This final approach uses $$O(n)$$  followed by $$O(k \log k)$$  to sort the top $$k$$  elements. The cost $$O(k \log k)$$  is negligible compared to $$O(n)$$  when $$n \gg k$$ .
 
 If you need the resulting array in descending order you can enter a reversed view of the sorted array by making use of the stride argument of the slice operator `array[::-1]`.

If you are interested in understanding how selection algorithms work, you can have a look at my [implementation of quickselect](https://gist.github.com/andrejonasson/17c9e9641e1bfd1134f5481ba6f99c32).