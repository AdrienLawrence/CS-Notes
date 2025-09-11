- “===” is a strict equality operator. It works like - “==” in python and java and does not attempt to coerce types, immediately returning False if two values are different types 

- Ex: “[] === 0” evaluates to False

- “==” is a loose equality operator. It works like - “===” but instead of returning False for different types it breaks the types down until a common type is found and then evaluates

- Ex: “[] == 0” evaluates to True. [] is coerced to 0 (number) and then compared to 0 (number).
