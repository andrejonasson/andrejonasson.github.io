---
layout: post
title: Efficiently retrieving <span>$k$</span> greatest elements
---
### Naive approach: Argsort
Say you want to retrieve the top <span>$k$</span> elements from an array of <span>$n$</span> elements. A simple approach would involve sorting the array and slicing the last <span>$k$</span> elements.

```python
import numpy as np

a = np.array([0.2, 0.1, 0.4, 0.025, 0.0125, 0.25, 0.0125])

def sorted_top_k(a, k):
  sorted_top_k_idx = np.argsort(a)[-k:]
  return a[sorted_top_k_idx]

# sorted_top_k(a, 3) == array([0.2 , 0.25, 0.4 ])
```
This approach consumes <span>$O(n \log n)$</span> operations by sorting the array. In circumstances when <span>$n$</span> is large, it can be wasteful to sort the whole array since all we really need is the top <span>$k$</span> elements for some specific <span>$k$</span>.

### Argpartition
An algorithm that achieves this is introselect. It finishes when all elements greater than the <span>$k$</span>th greatest element occupies indices greater than <span>$k$</span> and the <span>$k$</span>th greatest element is in its final sorted order. With an implementation that output an array in this manner we could just slice the last <span>$k$</span> elements off and achieve our goal without resorting to sorting the whole array. The `numpy` function `argpartition` lets us do just that and more.
 
```python
>>> a = np.array([0.2, 0.1, 0.4, 0.025, 0.0125, 0.25, 0.0125])

# Second greatest element in final sorted order, 
# with <= to the left and >= to the right
>>> a[np.argpartition(a, 1)]
# array([0.0125, 0.0125, 0.4   , 0.025 , 0.2   , 0.25  , 0.1   ])

# half sorted array
>>> a[np.argpartition(a, np.arange(len(a) // 2))]
array([0.0125, 0.0125, 0.025 , 0.4   , 0.2   , 0.25  , 0.1   ])

# `np.argpartition` allows for negative indices
>> np.argpartition(a, [len(a)-2, len(a)-1]) \
>>    == np.argpartition(a, [-2, -1])
array([ True,  True,  True,  True,  True,  True,  True])

# first and last element in their final sorted order
>>> a[np.argpartition(a, [0, -1])]
array([0.0125, 0.1   , 0.0125, 0.025 , 0.2   , 0.25  , 0.4   ])
```

If you pass an array and an index <span>$k$</span> to `argpartition` and then rearrange the original array with the array of indices returned by the function, then all elements smaller than or equal to the <span>$k$</span>th greatest element occupies indices less than <span>$k$</span> and therefore the <span>$k$</span>th greatest element is in its final sorted order. If you instead pass an array of indices <span>$k_1 ,..., k_m$</span> this property holds for all <span>$m$</span> indices.

Another way of putting it, after applying the function `argpartition` with a single argument <span>$k$</span> to some array <span>$\\{a_{i}\\}_{i=1}^{n}$</span> it will return an array of indices <span>$\\{i_j\\}_{j=1}^n$</span> such that the rearranged array <span>$\\{a_{i_j}\\}_{j=1}^n$</span> is in an order that satisfy <span>$a_k \ge a_i$</span> for all <span>$1 \le i \lt k$</span> and <span>$a_k \le a_i$</span> for all <span>$k \lt i \le n$</span>. If instead an array of indices <span>$\\{k_i\\}_{i=1}^m$</span> is passed to the function then the property will hold for all <span>$k$</span> in <span>$\\{k_i\\}_{i=1}^m$</span>.

Returning to our initial task, we can select the top <span>$k$</span> elements efficiently by using `argpartition`
```python
a = np.array([0.2, 0.1, 0.4, 0.025, 0.0125, 0.25, 0.0125])

def top_k(a, k):
  top_k_idx = np.argpartition(a, -k)[-k:]
  return a[top_k_idx]

# top_k(a, 3) == array([0.2 , 0.25, 0.4 ])
# NOTE: it happens to be sorted, but it is not guaranteed
```
This approach consumes only <span>$O(n)$</span> operations to find the top <span>$k$</span> elements.

``` python
def sorted_top_k(a, k):
  return top_k(a, k).sort()


def sorted_top_k_alternative(a, k):
  top_k_idx = np.argpartition(a, range(-k, 0))[-k:]
  return a[top_k_idx]
```
 To achieve a sorted array simply sort after or during the `argpartition`. This final approach uses <span>$O(n)$</span> followed by <span>$O(k \log k)$</span> to sort the top <span>$k$</span> elements. The cost <span>$O(k \log k)$</span> is negligible compared to <span>$O(n)$</span> when <span>$n \gg k$</span>.
 
 If you need the resulting array in descending order you can enter a reversed view of the sorted array by making use of the stride argument of the slice operator `array[::-1]`.

If you are interested in understanding how selection algorithms work, you can have a look at my [implementation of quickselect](https://gist.github.com/andrejonasson/17c9e9641e1bfd1134f5481ba6f99c32).