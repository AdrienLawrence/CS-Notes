# Overview

- SQL is the language used to interact with databases

- Relational Database Management Systems (RDBMs) store organized, uniform data in tables with rows and columns

- These relational databases use SQL to communicate with other code (i.e. requesting info on a certain row)

# CRUD

- Every interaction with a database comes down to CRUD, the four main operation types

- C -> Create [INSERT]

- R -> Read [SELECT]

- U -> Update [UPDATE]

- D -> Delete [DELETE]

1. CREATE Data [INSERT]: 

    ```SQL
    -- Insert a single user
    INSERT INTO users (name, email, age)
    VALUES ('Alice', 'alice@example.com', 30);

    -- Insert multiple users
    INSERT INTO users (name, email, age) VALUES
    ('Bob', 'bob@example.com', 25),
    ('Charlie', 'charlie@example.com', 35);
    ``` 
    Adds New Rows to the 'users' table

2. READ Data [SELECT]:
    ```SQL
    -- Select all columns (*) from users
    SELECT * FROM users;

    -- Select specific columns
    SELECT name, email FROM users;

    -- Filter with WHERE
    SELECT name, email 
    FROM users 
    WHERE age > 25;

    -- Sort with ORDER BY
    SELECT name, age
    FROM users
    ORDER BY age DESC;

    -- Limit results
    SELECT name, email
    FROM users
    LIMIT 10;
    ```
    Accesses Data in the users table for usage

3. UPDATE Data [UPDATE]:
    ```SQL
    -- Update a specific user's email
    UPDATE users
    SET email = 'new.alice@example.com'
    WHERE name = 'Alice';

    -- Give everyone a birthday (increase age by 1)
    UPDATE users
    SET age = age + 1;
    ```

    Updates email column in the row where name = 'Alice'

    Note: Always use WHERE unless updating every row

4. DELETE Data [DELETE]:
    ```SQL
    -- Delete a specific user
    DELETE FROM users
    WHERE name = 'Bob';

    -- Delete all users under 18
    DELETE FROM users
    WHERE age < 18;
    ```
    Deletes the row where name = 'Bob' and where age < 18

    Note: Always use WHERE unless deleting the whole database


    - SQL is used through strings with database methods

    - Ex. 
    ```python 
    # Using a library like 'psycopg2' for PostgreSQL
    cursor.execute("SELECT * FROM users WHERE age > %s",   (25,))
    ```

    Note: Never include user input in SQL calls for security reasons

# Operations

- SQL is different from simple database queries because it can combine or relate data from multiple tables logically

- JOIN at its core merges tables based on a common key or condition

- There are multiple JOIN types based off of different relationships

1. INNER JOIN - Intersection:

    Ex: 
    ```SQL
    SELECT users.name, orders.item
    FROM users
    INNER JOIN orders ON users.id = orders.user_id;
    ```
    This Joins the tables 'users' and 'orders' only where the 'user.id' key matches the 'orders.user_id' key

2. LEFT JOIN - Left + Intersection:

    Ex.
    ```SQL
    SELECT users.name, orders.item
    FROM users
    LEFT JOIN orders ON users.id = orders.user_id;
    ```

    This joins the entire 'users' table and the intersection between users.id and orders.user_id

3. RIGHT JOIN - Intersection + Right:

    Ex.
    ```SQL
    SELECT users.name, orders.item
    FROM users
    FULL OUTER JOIN orders ON users.id = orders.user_id;
    ```

    This joins the intersection between users.id and orders.user_id and the entire 'orders' table

4. FULL OUTER JOIN - Left + Intersection + Right:

    Ex. 
    ```SQL
    SELECT users.name, orders.item
    FROM users
    FULL OUTER JOIN orders ON users.id = orders.user_id;
    ```
    This joins the entire 'users' and 'orders' table, matched where possible and NULL where no matching value is found

- WHERE (condition) is a conditional used to filter tables before JOIN is performed (Ex. Filtering users by 'age >= 18' to get adults then performing a join operation)

- GROUP BY (column_name) is used to group rows with the same given column values into summary rows for further operation like COUNT(), SUM(), AVG(), etc.

- HAVING (condition) is used like WHERE but after GROUP BY has been applied. It is used for aggregates

- SELECT (column(s)) selects columns to display, * is used to select all columns. SELECT generates a new copy table with functional changes

- ORDER BY (ASC or DESC) sorts a table by ascending or descending for a given column. Can be used for multiple columns sequentially (Ex. ORDER BY countr ASC, age DESC)

- LIMIT (number) limits the number of rows displayed

- ORDER OF OPERATIONS: 

    1. FROM
    2. WHERE
    3. GROUP BY
    4. HAVING
    5. SELECT
    6. ORDER BY
    7. LIMIT

# Aggregate Operations

- COUNT(column) counts the number of rows

- SUM(column) adds the values of each row for a given numeric column

- AVG(column) Calculates the average of a numeric column

- MIN() / MAX() Finds the minimum/maximum numeric value in a column

Note: NULL values are skipped

# Subqueries

- Subqueries are SELECT statements inside of SELECT statements


