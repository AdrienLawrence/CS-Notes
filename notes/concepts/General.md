# Decorators, Wrappers and HOF's

- Higher Order Functions (HOF) deal with using functions as variables. They may either take a function as an input or output/return a function, or both. 

- Ex. Input function: Function application modifiers (def applyTwice(func, arg): takes in a function and its argument and applies the function twice to the argument. Other examples are some embedded functions like map which applies functions to every index in a list argument or filter which eliminate indices in a list argument based on a function which returns True/False for every element)

- Ex. Output function: Outer functions which operate with inner functions and return their product

- Decorators take in a function in order to temporarily alter its implementation or add some markers around it and return an altered function. This does not change the implementation of the altered function outside of the scope of the decorator.

- Applications include caching, logging, or authentication 

- Called with @decorator_name above a function (Ex. “@timer”)

- To pass arguments to a decorator, enclose the decorator in an outer function and call the outer function with @outer_function and its argument (Ex. “@repeat(3)”)

- A wrapper is used within a decorator to take in the functions arguments when altering the function. It is necessary to include within a decorator because a decorator cannot take in a function and its arguments. The wrapper includes the altered logic. 

- “(*args, **kwargs)” represents any range of original function parameters to be passed from a higher function to the inner altered function

- A decorator transfers the function to the wrapper for processing. The decorator is the gift wrapper who decides how to wrap a gift while a wrapper is the wrapping paper that literally wraps the gift

Higher Order Functions:
Accepts Function:	

```python
def apply_to_list(func, lst):  # ← Takes a function `func`
    		return [func(x) for x in lst]  # Applies func to each item

def square(x):
    		return x * x

numbers = [1, 2, 3]
squared_numbers = apply_to_list(square, numbers)  # Uses `square` as input
print(squared_numbers)  # Output: [1, 4, 9]
```
Returns Function:

```python
def make_multiplier(n):  # ← Returns a function
    def multiplier(x):
        return x * n
    return multiplier

double = make_multiplier(2)  # Returns `multiplier` with n=2
print(double(5))  # Output: 10
```

Decorator/Wrapper:

```python
import time

def timer(func):                   # ← Takes a function (`func`)
    def wrapper(*args, **kwargs):  # ← Returns a function (`wrapper`)
        start = time.time()
        result = func(*args, **kwargs)  # Calls the original function
        print(f"Time: {time.time() - start:.2f}s")
        return result
    return wrapper

@timer  # ← Decorator syntax (equivalent to `slow_func = timer(slow_func)`)
def slow_func():
    time.sleep(1)

slow_func()  # Output: "Time: 1.00s"
```

# Heaping

- Heaping is a method of sorting which involves sorting a list into a binary tree in place using the heapq. module

- heaping does not sort a list sequentially from smallest to largest, but sorts it hierarchically by levels of magnitude with the single smallest element at the bottom

- A heapify(list) operation is used to sort a list to fit the heap invariant and form a binary tree hierarchy 

- The only guarantee is that the first element is the smallest element in the list. The list will be loosely ordered from then on

- Heaping is not used in situations which require sequential sorting, but in situations which deal with constantly accessing the smallest and largest elements of a list (K largest/K smallest, top K elements, max/min)

- A prime use case is priority queues, which deal with accessing and removing the first element, and adding new elements. Accessing the smallest/largest element of a heap is O(1), and removing/adding elements is O(log(n)), saving runtime with large lists. 

- They may also be used for Dijkstra’s algorithm and maintaining the top K elements in large or infinite datasets

- Can also compare tuples on an element-by-element basis

- There are two types of heaps: min-heaps and max-heaps. Heaps are by default min-heaps, meaning the first element is the smallest and the magnitude grows with new levels. Heapify can only create min-heaps by default. 

- You may implement a max-heap by converting every list element to a negative, which flips their sequential relation. By making a min-heap with negative elements you can access those elements by converting them back into positive values which in effect creates a max-heap.

heapq functions:

```python

import heapq

heapify(list)

heappush(heap, item), 

heappop(heap), 

heappushpop(heap, item), 

heapreplace(heap, item), 

nlargest(n, iterable, key=None) (n is number of largest elements, key is comparison function, by default values are compared against each other)
nsmallest(n, iterable, key=None) 

Heappushpop() pushes first and pops second, while heapreplace() pops first and pushes second

There is no built-in delete function, the most common method is to mark the desired element as invalid and skip it when popped. Otherwise you can remove the item but then the heap must be resorted (O(n)).
```
