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

# Core Syntax and Collections

- Python uses indentation to define blocks

- Common built-in types include integers, floats, complex numbers, booleans, strings, bytes, lists, tuples, dictionaries, sets, ranges, and `None`

    ```python
    name = "Alice"
    scores = [91, 84, 97]
    user = {"id": 1, "name": name}
    unique_tags = {"python", "api"}
    point = (10, 20)
    ```

- Lists are ordered and mutable. Tuples are ordered and normally used as fixed records. Sets store unique hashable values. Dictionaries map hashable keys to values

- Indexing begins at zero. Negative indexes count from the end

- Slicing uses `[start:stop:step]` and excludes the stop index

    ```python
    first_three = values[:3]
    every_second = values[::2]
    reversed_copy = values[::-1]
    ```

- Unpacking assigns elements to names

    ```python
    first, second, *remaining = values
    x, y = y, x
    ```

- Comprehensions create collections from iteration and optional conditions

    ```python
    squares = [number * number for number in numbers]
    active_by_id = {user.id: user for user in users if user.active}
    normalized = {tag.strip().lower() for tag in tags}
    ```

- Prefer a normal loop when a comprehension requires multiple side effects or becomes difficult to read

# Conditions and Iteration

    ```python
    if score >= 90:
        grade = "A"
    elif score >= 80:
        grade = "B"
    else:
        grade = "C"
    ```

- Empty containers, zero, `None`, and `False` are falsy. Most other values are truthy

- `is` checks object identity. `==` checks value equality

- Use `is None` and `is not None` for None checks

- `for` iterates over an iterable. `while` repeats while a condition is true

- Useful iteration helpers:

    `enumerate(values)`: Index and value

    `zip(left, right)`: Corresponding elements

    `range(start, stop, step)`: Integer sequence

    `sorted(values, key=...)`: Sorted list

    `reversed(values)`: Reverse iterator

    `any(values)` / `all(values)`: Boolean aggregation

- `break` exits a loop. `continue` begins the next iteration. A loop `else` executes when the loop finishes without `break`

# Functions

    ```python
    def create_user(
        email: str,
        *,
        active: bool = True,
    ) -> dict[str, object]:
        """Create a serializable user value."""
        return {"email": email, "active": active}
    ```

- Positional arguments match order. Keyword arguments match name

- A `/` marker ends positional-only parameters. A `*` marker begins keyword-only parameters

- `*args` collects extra positional arguments. `**kwargs` collects extra keyword arguments

- Default values are created once when the function is defined

- Avoid mutable defaults:

    ```python
    def add_item(item, items=None):
        if items is None:
            items = []
        items.append(item)
        return items
    ```

- Functions are objects and may be passed, returned, stored, and decorated

- A lambda is a small anonymous expression function. Use `def` for multi-step behavior

- Python closures retain names from an enclosing scope. `nonlocal` rebinds an enclosing function name, while `global` rebinds a module name

- Docstrings explain public purpose, parameters, returns, errors, and important behavior

# Iterators and Generators

- An iterable can produce an iterator. An iterator returns successive values through `__next__`

- `iter(value)` gets an iterator. `next(iterator)` gets one value and eventually raises `StopIteration`

- A generator function uses `yield` and pauses between values

    ```python
    def read_batches(items, size):
        for index in range(0, len(items), size):
            yield items[index:index + size]
    ```

- Generators process large or streaming data without storing every result at once

- Generator expressions use parentheses: `(transform(x) for x in source)`

- `yield from` delegates to another iterable/generator

- Iterators are consumed. Convert to a list only when repeated access or all values in memory are actually required

# Classes and Dataclasses

    ```python
    from dataclasses import dataclass, field


    @dataclass(slots=True)
    class Notebook:
        id: str
        title: str
        tags: list[str] = field(default_factory=list)

        def rename(self, title: str) -> None:
            if not title.strip():
                raise ValueError("title is required")
            self.title = title
    ```

- Instance methods receive `self`. Class methods receive `cls`. Static methods receive neither automatically

- `@property` exposes method-controlled attribute access

- Dataclasses generate common initialization, representation, and equality behavior for data-oriented classes

- Use `default_factory` for mutable dataclass defaults

- `slots=True` can reduce per-instance memory and prevent arbitrary new attributes

- Inheritance models subtype relationships. Composition is often simpler when one object merely uses another

- Method Resolution Order controls lookup across inheritance and is visible through `Class.mro()`

- `super()` delegates to the next implementation in the MRO

# Exceptions and Context Managers

    ```python
    try:
        result = process(path)
    except FileNotFoundError as error:
        raise DataLoadError(f"Missing file: {path}") from error
    else:
        publish(result)
    finally:
        release_lock()
    ```

- Catch the narrow exception the code can handle

- `raise NewError(...) from error` preserves causal context

- Avoid swallowing exceptions with an empty `except`

- Custom exceptions should describe domain failure categories

- A context manager performs setup and guaranteed cleanup around a `with` block

    ```python
    with open("data.txt", encoding="utf-8") as file:
        text = file.read()
    ```

- Create context managers with `__enter__`/`__exit__` or `contextlib.contextmanager`

# Modules, Packages, and Imports

- A module is a Python file. A package is an importable directory/tree of modules

- Imports execute a module once per interpreter process and cache it in `sys.modules`

- Use absolute package imports for clarity. Relative imports are useful inside a package when deliberate

- Code under `if __name__ == "__main__":` runs only when the file is executed as the entry point

    ```python
    def main() -> None:
        run_application()


    if __name__ == "__main__":
        main()
    ```

- Avoid circular imports by separating shared types/behavior and keeping dependency direction clear

- `pyproject.toml` is the modern central configuration for packaging and many tools

- Virtual environments isolate project dependencies

    ```bash
    python -m venv .venv
    .venv\Scripts\activate
    python -m pip install -r requirements.txt
    ```

- Dependency lockfiles improve repeatability. Keep runtime and development dependencies deliberate

# Type Hints

- Type hints improve static checking and documentation but do not enforce runtime values by themselves

    ```python
    from collections.abc import Iterable
    from typing import TypeVar

    T = TypeVar("T")


    def first(values: Iterable[T]) -> T | None:
        return next(iter(values), None)
    ```

- Common forms include `list[str]`, `dict[str, int]`, `str | None`, `Callable`, `Literal`, `Protocol`, `TypedDict`, and generics

- A `Protocol` defines structural behavior: an object matches by implementing required members

- `TypedDict` describes dictionary keys for static checking

- Narrow `object` or `Any` values before using them. Prefer `object` when callers must prove the actual type

- Use mypy, Pyright, or another type checker in CI for projects depending on type guarantees

- Runtime data from JSON, environment variables, and APIs still requires parsing/validation with code or a library such as Pydantic

# Files, JSON, and Serialization

    ```python
    import json
    from pathlib import Path

    path = Path("users.json")
    users = json.loads(path.read_text(encoding="utf-8"))
    path.write_text(json.dumps(users, indent=2), encoding="utf-8")
    ```

- JSON supports objects, arrays, strings, numbers, booleans, and null, not arbitrary Python objects

- Use `csv`, `sqlite3`, `tomllib`, and other standard modules for matching formats

- Pickle can execute code while loading. Never unpickle untrusted data

- Validate paths and file sizes for user-controlled uploads

- Use atomic replace patterns when a partially written file would be harmful

# Async Python

- `asyncio` runs cooperative asynchronous tasks on an event loop

    ```python
    import asyncio


    async def main():
        left, right = await asyncio.gather(
            fetch("https://example.com/a"),
            fetch("https://example.com/b"),
        )


    asyncio.run(main())
    ```

- `async def` returns a coroutine when called. It runs when awaited or scheduled

- Async is useful for high-concurrency I/O, not automatically for CPU-heavy computation

- Blocking calls inside async code block the event loop. Use async libraries or deliberate thread/process execution

- `TaskGroup` provides structured concurrency where sibling failures and cleanup are coordinated

- Use timeouts, cancellation-safe cleanup, semaphores for concurrency bounds, and queues for producer/consumer flow

- Threads share memory and suit blocking I/O. Processes have separate memory and can run CPU work in parallel

# Testing and Tooling

- `pytest` discovers test functions and provides fixtures, parametrization, markers, and useful assertion reporting

    ```python
    import pytest


    @pytest.mark.parametrize(
        ("left", "right", "expected"),
        [(1, 2, 3), (-1, 1, 0)],
    )
    def test_add(left, right, expected):
        assert add(left, right) == expected
    ```

- Fixtures provide reusable setup/cleanup and may use `yield`

- Unit tests isolate deterministic behavior. Integration tests verify databases, networks, files, and framework wiring

- Mock only a boundary whose real behavior is inappropriate for the test. Excessive mocking tests assumptions instead of integration

- Common tools include Ruff for lint/format, Pyright/mypy for types, pytest for tests, coverage.py for coverage, and pre-commit for local checks

- Use logging instead of print statements for production diagnostics

- Add a main entry point, deterministic configuration, timeouts, and useful errors before calling a script production-ready

# Performance

- Choose the correct algorithm and data structure before micro-optimizing syntax

- Measure with `timeit`, `cProfile`, or a sampling profiler

- Built-ins and standard-library operations are often implemented efficiently

- Generators reduce memory for streaming workflows

- Cache only pure/stable expensive work and include every relevant input in the key

- Use NumPy/PyTorch vectorized operations for large numeric arrays instead of Python element loops where appropriate

- The standard CPython interpreter lock affects CPU-bound Python threads. Use processes, native libraries, or appropriate runtimes for CPU parallelism

- Optimize after measuring representative workloads and preserve correctness tests
