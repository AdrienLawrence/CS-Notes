# Overview

- SQL is the language used to interact with databases

- Relational Database Management Systems (RDBMS) store organized, uniform data in tables with rows and columns

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
    RIGHT JOIN orders ON users.id = orders.user_id;
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

- WHERE (condition) is a conditional used to filter tables before JOIN is performed (Ex. Filtering users by 'WHERE age >= 18' to get adults then performing a join operation)

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

- Types: Scalar, Column, Table, Correlated

1. Scalar Subquery: Returns a single value (One row/column). Used for comparison

    Ex.
    ```SQL
    -- Find users older than average age
    SELECT name, age
    FROM users
    WHERE age > (SELECT AVG(age) FROM users);

    -- Find product with highest price
    SELECT product_name, price
    FROM products
    WHERE price = (SELECT MAX(price) FROM products);
    ```
    Here, age is compared to a SINGLE value (AVG(age)), and price is compared to a SINGLE value (MAX(price))

2. Column Subquery: Returns a single column with multiple rows. Used with IN, ANY, and ALL operators

    Ex. 
    ```SQL
    -- Find users from most common countries
    SELECT name, country
    FROM users
    WHERE country IN (
        SELECT country
        FROM users
        GROUP BY country
        HAVING COUNT(*) > 10
    );

    -- Find products that have never been ordered
    SELECT product_name
    FROM products
    WHERE product_id NOT IN (
        SELECT DISTINCT product_id
        FROM orders
    );
    ```
    The first example uses a column with every group/country with an aggregate count over 10 and returns every country which fits that condition

    The second uses a set of unique column values which appear in orders and thus have been ordered once and finds which products are not in that set

3. Table Subquery: Returns a complete table. Used in FROM clauses

    Ex. 
    ```SQL
    -- Get average salary by department
    SELECT dept_name, AVG(salary) as avg_salary
    FROM (
        SELECT department as dept_name, salary
        FROM employees
    ) AS department_data
    GROUP BY dept_name;

    -- Get top 3 highest paid employees per department
    SELECT *
    FROM (
        SELECT *,
            ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank
        FROM employees
    ) AS ranked_employees
    WHERE rank <= 3;
    ```

    The first example breaks the employees table into a smaller table with only department and salary, from which the average salary is calculated

    The second example creates a smaller table with each department partition and orders them by salary as rank, where the top 3 ranks are returned

    Note: Table subqueries MUST be given an alias (AS "") for reference

4. Correlated Subqueries - Iterative subqueries of any of the three other types (executed for each row)

    Ex. 
    ```SQL
    -- Find employees who earn more than their department average
    SELECT name, salary, department
    FROM employees e1
    WHERE salary > (
        SELECT AVG(salary)
        FROM employees e2
        WHERE e2.department = e1.department
    );

    -- Find users who have placed orders in the last 30 days
    SELECT name, email
    FROM users u
    WHERE EXISTS (
        SELECT 1
        FROM orders o
        WHERE o.user_id = u.user_id
        AND o.order_date >= CURRENT_DATE - INTERVAL '30 days'
    );
    ```

    The first example compares a given employees salary to the average salary in their department by comparing the inner copy table department (e2) to the outer table department (e1) to ensure the departments are matching.

    Note: GROUP BY and JOIN are typically more efficient

    The second example uses EXISTS to determine if a row exists for a user who has ordered an item in the past 30 days. It checks to see the user_id mathces the order and then if the order date is within the string '30 days'. 

# Indexing

- An index is a data structure used to find data without iterating

- Ex. 
    ```SQL 
    CREATE INDEX idx_users_email ON users(email); 
    ```
- Indexes are used with frequently accessed columns such as those referenced in WHERE, JOIN, or ORDER BY operations

- While they improve query speeds, they reduce the speed of operations which alter the database due to changing the index

- Most databases use a Balanced tree, which is a kind of search tree where a uniform height is maintained between leaf nodes through splitting and promotion of node values when adding data

- Each node can contain multiple data values, so searching for a particular value in this tree requires an overall binary search down the tree and a small linear search within nodes, leading to a hybrid search pattern

- Sequential, linear scans are slow for large databases hence using faster, binary index scan (O(logn) vs O(n))

- An index is an object created on columns which is used for quick row accessing

- An index sorts at compile time into a B-tree and searches at runtime for each call in a query

- Index Creation:
    ```SQL
    CREATE INDEX index_name ON table_name (column_name);
    ```
- You can create indexes on multiple columns by inserting more columns in the () parameter of the CREATE INDEX operation. These are called composite indexes

- These are not good for filtering on singular columns only. The most specific column should be put first in the parameter

- Queries automatically decide on using sequential or index searching

- You can ask if an index search will be used as opposed to a sequential search with the EXPLAIN command: 
    ```SQL
    EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';
    ```

- DROP INDEX can be used for destroying indexes and freeing memory

# Transactions and ACID

- A transaction is a single unit of work that consists of multiple SQL operations. These operations are chunked together and either succeed or fail, with no partial successes

- A transaction follows ACID properties

A (Atomicity) - Chunked operations must all succeed, if any operation fails, the chunk fails. This prevents partial completion of a transaction that can result in lost data or errors

C (Consistency) - A transaction must leave a database intact and consistent, following all constraints and rules of the transaction

I (Isolation) - Transactions operate within their own bounds, separate from all other transactions, including simultaneous transactions. Transactions have controllable access to intermediate states of other transactions

D (Durability) - Transaction database alterations are permanent and sustained (in the event of failures)

Transaction Commands: 

    BEGIN; or START TRANSACTION; - Begins transaction block

    COMMIT; - Ends transaction and saves changes 

    ROLLBACK; - Ends transaction and undoes changes made, reverting it to pre-transaction state

- You can control the isolation level of concurrent transactions with four levels of isolation:

    1. READ UNCOMMITTED: You can see other transactions uncommitted changes. This can lead to dirty reads where data is read which is rolled back later

    2. READ COMMITTED: You can see other transactions committed data. This is the most common

    3. REPEATABLE READ: (Snapshot of data) No external database changes are reflected within the transaction for separate queries, but you can change the database internally and those changes will be reflected later within the transaction. Prevents non-repeatable reads where the same row returns different values in consequential queries due to concurrent alteration from another transaction

    4. SERIALIZABLE: Forces transaction to incorporate logic that does not allow for conflicting concurrent alterations. The outcome for concurrent transactions must be the same as if the transactions were ran sequentially (forced mutual exclusion). Prevents phantom reads where a different number of rows is returned in consequential queries. Seriazable errors are caught at runtime after the COMMIT; line is reached

Example Bank Transfer Transaction:
```SQL
    -- Start the transaction
    BEGIN ISOLATION LEVEL READ COMMITTED;

    -- Attempted withdrawal
    UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';

    -- Check if withdrawal was valid using constraints within 'accounts' definition

    -- Assuming no constraints broken
    UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';

    -- End transaction, commiting changes
    COMMIT;
```

# Design and Normalization

- The goal of normalization is to prevent anomalies and minimize storing duplicate values

- An unnormalized table may store consequential rows where much of the value is duplicated with only a column or two differing, such as a poorly designed orders table with redundant user info

- PRIMARY KEY is a constraint in a table schema that signifies column values must be unique and not NULL. Adding multiple columns into the () parameter makes a composite key

- Ex. 
    ```SQL
    CREATE TABLE users (
        user_id INT, 
        name VARCHAR(100) NOT NULL,
        email VARCHAR(100) UNIQUE,
        PRIMARY KEY (user_id) -- Insert more columns here for composite key
    )

    -- Or 

    ALTER TABLE users
    ADD PRIMARY KEY (user_id);
    ```

First Normal Form (1NF): Values in the table are indivisible and singular. One row/column combination (i.e. fourth row in names column) should not hold more than one value

- Ex. Imagine an orders table with the columns:

names, phone_number

- Also imagine someone named Alice with two phone numbers, and imagine they are stored like this

names, phone_number
('Alice'), ('678-678-6789' '123-456-7890')

- Here, Alice's row has two values under the phone_number column. Imagine she deletes one phone number. Now you must remember her other phone number in order to update the phone_number column or make a new single phone number column as opposed to deleting the row with the singular phone number to be deleted if Alice had two rows each with a separate phone number

Second Normal Form (2NF): 1NF prerequisite. Every non-key attribute should include data related to the full composite key, as opposed to only being based off of part of a composite key

- Ex. Imagine a student grade table with the following columns:

student_id, course_id, course_name, grade

- Here, course_name and course_id describe the same sets of values, course_name only further describes course_id, which is only part of the composite key (student_id, course_id) used to get the specific grade for a specific student.

- Instead there should be a designated courses table with the following columns: 

course_id, course_name

Third Normal Form (3NF): 2NF prerequisite, no transitive dependencies. All non-key attributes must depend directly on the primary key, no A->B->C chain dependencies (i.e. a name column beside a non-key id column), or else the table should be split

- Ex. Imagine a similar teacher table which stores their taught course with the following columns:

teacher_last_name, course_id, course_name

- Here, course_name is not specific to the teacher's name. Instead it is specific to the course_id. course_name is just simplification of course_id, which is not part of the Primary Key (teacher_last_name). As such, it is redundant to include

Note: 2NF applies to only composite key tables, primary key tables are automatically composite

 - Rule of thumb - DRY: Don't Repeat Yourself. Avoid duplicate data

# Advanced Query Techniques

Window Functions: Perform calculations across a set of table rows related to the current row by some column value (like department) wuthout collapsing the rows

Available functions: 

    - ROW_NUMBER(): Assigns a unique number to each row (like a new primary key for each group)

    - RANK(): Ranks items by some numeric column values with gaps for ties (1, 2, 2, 4)

    - DENSE_RANK(): Works like RANK() but without gaps (1, 2, 2, 3)

    - SUM()/AVG() OVER(PARTITION BY...): Running Totals/Averages per groups

- Ex.
    ```SQL
    SELECT 
        name, 
        department, 
        salary,
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
    FROM employees;
    ```

    This query ranks employees by salary for their respective department

Common Table Expressions: Subqueries defined with WITH clauses, helping to break down logic by assigning the logic to a CTE variable

- Ex.
    ```SQL
    WITH department_avgs AS (
        SELECT 
            department, 
            AVG(salary) AS avg_salary
        FROM employees
        GROUP BY department
    )
    SELECT 
        e.name, 
        e.department, 
        e.salary, 
        d.avg_salary
    FROM employees e
    INNER JOIN department_avgs d ON e.department = d.department
    WHERE e.salary > d.avg_salary;
    ```

    This CTE finds employees who earn more than their department average

    # Constraints and Schema

    # Advanced Indexing

    # Performance Optimization

    # Security

    # Procedural SQL

    # PostgreSQL