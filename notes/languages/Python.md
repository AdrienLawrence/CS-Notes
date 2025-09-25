- Variables point to objects, not copies. It is a pointer to a specific instance

- 'x = 5', x points to an integer object '5'

- 'a = [1, 2, 3]', a points to a list object

- Because of this behavior, variables are extra reference assignments for what they are equated to.

- Consider this code block: 
    ```python
    a = [1, 2, 3]
    b = a

    a.append(4)

    print(b)
    ```
    You would think the output would be "[1, 2, 3]", but it is actually "[1, 2, 3, 4]". This is because 'b' did not create a new list, but added a reference to the same list 'a' refers to. This means 'a' and 'b' share the same list object.

- This does not apply to primitives such as int, float, str, tuple, frozenset, or bool. These do create new object copies

- The 'copy' mechanism does apply to list, dict, set, and bytearray

- This behavior can be avoided using '.copy' or 'deepcopy()' instead of '='

.copy() and .deepcopy(): 

    - This function creates a new outer object for a new element, but if any inner elements are 'copyable' objects, they maintain the same references as the original object to be copied

- Ex. 
    ```python
    a = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

    b = a.copy()

    a.append([1, 2, 3])

    print(b) # Output: [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    ```
    Here, changing the outer list of 'a' does not change the 'b' copy object. Let's see what happens when we change an inner list

    ```python
    a[0].append(4)

    print(b) # Output: [[1, 2, 3, 4], [4, 5, 6], [7, 8, 9]]
    ```
    .copy() only makes a copy of the outer object. The inner objects are still the same as the original object. It's like putting the same items into a new bag. This means changing the inner object attributes for the original does change the inner object attributes for the copy and vice versa

    - If we were to have used 'b = a.deepcopy()', then that makes a copy of the outer object and any inner objects. It is a new bag and new items inside the new bag. Changing the inner objects of one does not affect the other

# Dunder Methods

- Dunder is a conjunction which means Double Underscore, and refers to special methods defined as '__method__', with double underscores on both sides

- Core dunder methods are: __init__, __str__, __repr__, __getitem__, and __iter__

__init__: An intializer method. In classes, this transfers parameter inputs to their respective attributes

- Ex. 
    ```python
    class Point:
        def __init__(self, x: int, y: int):
            self.x = x
            self.y = y

    p = Point(1, 2)
    print(p.x) # Output: 1
    print(p.y) # Output: 2
    ```
    The __init__ method passes the parameters to the actual 'p' object. Without this method, python does not know what to do with the values in 'Point(1, 2)' and p would not have any attributes

    Note: the 'self' keyword is necessary to refer to the instance of the object you are attempting to create or modify. 'self.x = x' is equivalent to the object saying "My x attribute equals the x attribute passed into my init method"

__str__: Allows an object to be printed using 'print(object)'. What is printed is determined by how the method is implemented

- Ex. 
    ```python
    class Point:
        def __init__(self, x: int, y: int):
            self.x = x
            self.y = y

        def __str__(self):
            return f"Point: {self.x}, {self.y}"

    p = Point(1, 2)
    print(p) # Output: Point: 1, 2
    ```
    
    Without the 'str' method printing an object will cause an error

__repr__: A printable representation of an object and its attributes for developers to see

- Ex. 
    ```python
    class Point:
        def __init__(self, x: int, y: int):
            self.x = x
            self.y = y

        def __repr__(self):
            return f"Point({self.x}, {self.y})"

    p = Point(1, 2)
    repr(p) # Output: Point(x='1', y='2')
    ```

    The 'repr' method provides the type of object of 'p' as well as its attributes

__getitem__: Allows element access to list objects for iteration or element returns

- Ex. 
    ```python
    # List class which contains Student objects
    # Assume Student class is defined
    class Students:

        def __init__(self, students=None):
            self.students = students if students is not None else []
        
        def add_students(self, student: Student):
            self.students.append(student)
        
        def __getitem__(self, i: int) -> Student:
            return self.students[i]

    student1 = Student("Joe")
    student2 = Student("Schmoe")

    student_list = Students([student1, student2])

    for student in student_list: # Output: "Joe", "Schmoe"
        print(student.name)
    ```

    The 'getitem' method provides single element results for the list object Students. This is useful for accessing singular elements, for traversal an 'iter' method would work better

    Note: List Class objects are NOT iterable without a 'getitem' or 'iter' method, even if they contain a list of elements

__iter__: Similar to 'getitem', but provides EVERY element one-by-one as opposed to a singular element. Basically functions as 'getitem' combined with a for loop

- Ex. 
    ```python
    # Same class as previous example
    class Students:

        def __init__(self, students=None):
            self.students = students if students is not None else []
        
        def add_students(self, student: Student):
            self.students.append(student)
        
        def __iter__(self):
            return iter(self.students)

        # OR (with generator function)

        def __iter__(self):
            for students in self.students: # iterates automatically
                yield student

    student1 = Student("John")
    student2 = Student("Jane")

    student_list = Students([student1, student2])

    for student in students: # Output: "John", "Jane"
        print(student.name)
    ```

    Here, 'iter' practically works the same way as 'getitem' but the differences become more nuanced in other scenarios. Usually a 'len' method is needed to prevent index errors
    
- There are other dunder methods such as: 
__eq__ (for type checking an object), 
__len__ (for having a reference to a list object's length), 
__next__ (for iterators), 
and more, but these are less commonly used and easy to look up