# Merge Sort (Onlogn)

```python
def mergeSort (list: List[int]) -> List[int]:
    if len(list) == 0:
        return "[]"
    if len(list) == 1:
        return list
    mid = len(list) // 2

    left = list[:mid]
    right = list[mid:]

    self.mergeSort(left)
    self.mergeSort(right)

    def mergeTwo(a: List[int], b: List[int]) -> List[int]:
        merged = []
        i = j = 0
        while i < len(a) and j < len(b):
            if a[i] >= b[j]
                merged.append(a[i])
                i += 1
            else:
                merged.append(b[j])
                j += 1
        merged.append(a[i:])
        merged.append(b[j:])

    return mergeTwo(left, right)

    #for linked lists skip helper function and use:
    dummy = cur = ListNode()
    while left and right:
        if left.val >= right.val:
            cur.next = left
            left = left.next
        else: 
            cur.next = right
            right = right.next
    cur.next = left or right
    return dummy.next
```

# Selection Sort (On^2)

```python
def selectionSort(arr):
		for i in range(len(arr)):
			minIdx = i:
			for j in range(i + 1, len(arr)):
				if arr[j] < arr[minIdx]:
					minIdx = j
			arr[i], arr[minIdx] = arr[minIdx], arr[i]
		return arr
```

# Insertion Sort (On^2)

```python
def insertionSort(arr):
		for i in range(1, len(arr)):
			key = arr[i]
			j = i - 1
			while j >= 0 and key < arr[j]:
				arr[j + 1] = arr[j]
				j -= 1
			arr[j + 1] = key
		return arr
```
