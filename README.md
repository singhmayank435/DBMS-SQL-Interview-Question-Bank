# README — Answer Key for the DBMS + SQL Interview Question Bank

This file answers every question (Q1–Q170) from `dbms-sql-interview-question-bank.md`, in the same order. Each entry is written as **Question → Answer**, so it's easy to scan.

Reference schema used throughout:

```sql
Employees(id, name, salary, dept_id, manager_id, join_date)
Departments(id, dept_name, location)
Orders(id, customer_id, order_date, amount)
Customers(id, name, city)
```

---

# SECTION 1 — DBMS Fundamentals

**Q1. What is a DBMS?**

> A DBMS (Database Management System) is software that lets you create, store, retrieve, update, and manage data in a structured way, while handling security, concurrency, and data integrity for you — instead of you managing raw files yourself.

**Q2. What is an RDBMS, and how is it different from a DBMS?**

> An RDBMS (Relational DBMS) is a DBMS that stores data in tables (relations) with rows and columns, and enforces relationships between tables using keys. A plain DBMS doesn't have to be relational — it could be hierarchical, network-based, or file-based. RDBMS also enforces ACID properties and constraints (e.g., MySQL, PostgreSQL).

**Q3. What is a database, and how is it different from a table?**

> A database is a container that holds related data — many tables, plus indexes, views, and other objects. A table is a single structured object inside a database, made of rows and columns holding one type of entity's data (e.g., `Employees`).

**Q4. What is a table? Define row and column.**

> A table is a structured collection of related data organized into rows and columns.
> 
> - A **row** (record/tuple) is one entry — e.g., one employee.
> - A **column** (field/attribute) is one property shared by every row — e.g., `salary`.

**Q5. What is a schema?**

> A schema is the logical blueprint of a database — its tables, columns, data types, relationships, and constraints — without referring to the actual data stored in it.

**Q6. What is a Primary Key, and why is it required?**

> A Primary Key is a column (or set of columns) that uniquely identifies every row in a table. It cannot contain NULLs and must be unique. It's required so any row can be unambiguously referenced, updated, or deleted, and so other tables can point to it via foreign keys.

**Q7. What is a Candidate Key? How is it different from a Primary Key?**

> A Candidate Key is any column (or combination) that *could* qualify as the Primary Key — it's unique and never NULL. A table can have several candidate keys, but only one is chosen as the Primary Key; the rest remain candidate keys.

**Q8. What is a Super Key, and how does it relate to a Candidate Key?**

> A Super Key is any set of columns that uniquely identifies a row — it's allowed to include extra, unnecessary columns. A Candidate Key is a *minimal* Super Key (removing any column would break uniqueness). Every Candidate Key is a Super Key, but not every Super Key is a Candidate Key.

**Q9. What is a Composite Key? Give an example.**

> A Composite Key is a Primary Key made of two or more columns together. Example: in `Enrollments(student_id, course_id, enrolled_date)`, `(student_id, course_id)` together form the composite key — neither column alone is unique.

**Q10. What is an Alternate Key?**

> An Alternate Key is a Candidate Key that was **not** chosen as the Primary Key. If `Employees` has both `id` (chosen as PK) and a unique `email` column, `email` is the Alternate Key.

**Q11. What is a Foreign Key, and how is it related to the Primary Key?**

> A Foreign Key is a column in one table that references the Primary Key of another table, linking the two. Example: `Employees.dept_id` references `Departments.id`. It enforces referential integrity — you can't insert an employee pointing at a department that doesn't exist.

**Q12. How would you connect Employees and Departments using a Primary Key and Foreign Key?**

> `Departments.id` is the Primary Key. `Employees.dept_id` is a Foreign Key referencing it:
> 
> ```sql
> CREATE TABLE Departments (
>     id INT PRIMARY KEY,
>     dept_name VARCHAR(50)
> );
> 
> CREATE TABLE Employees (
>     id INT PRIMARY KEY,
>     name VARCHAR(50),
>     salary INT,
>     dept_id INT,
>     FOREIGN KEY (dept_id) REFERENCES Departments(id)
> );
> ```

**Q13. What are constraints? Name a few.**

> Constraints are rules enforced on columns to keep data accurate and consistent. Common ones: PRIMARY KEY, FOREIGN KEY, NOT NULL, UNIQUE, DEFAULT, CHECK.

**Q14. What does NOT NULL do?**

> It forces a column to always have a value — any INSERT/UPDATE that would leave it empty is rejected.

**Q15. What is UNIQUE, and how is it different from Primary Key?**

> UNIQUE ensures all values in a column are distinct (most databases allow one NULL), and a table can have several UNIQUE columns. PRIMARY KEY is unique **and** never NULL, and a table can have only one.

**Q16. What does DEFAULT do?**

> It provides a fallback value when none is supplied on INSERT.
> 
> ```sql
> ALTER TABLE Employees ADD COLUMN join_date DATE DEFAULT CURRENT_DATE;
> ```

**Q17. What is CHECK used for?**

> It enforces that a column's values satisfy a condition.
> 
> ```sql
> ALTER TABLE Employees ADD CONSTRAINT chk_salary CHECK (salary > 0);
> ```

**Q18. What is a One-to-One relationship? Give an example.**

> Each row in Table A relates to exactly one row in Table B, and vice versa. Example: `Employees` and `EmployeePassportDetails` — each employee has exactly one passport record.

**Q19. What is a One-to-Many relationship? How is it implemented for Employees/Departments?**

> One department has many employees, but each employee belongs to only one department. The Foreign Key (`dept_id`) sits on the "many" side (`Employees`) and points to the "one" side's Primary Key (`Departments.id`).

**Q20. What is a Many-to-Many relationship? Why is a junction table needed?**

> One student takes many courses, and one course has many students — that's Many-to-Many. A single FK column on either table can't represent this, so a junction/bridge table (e.g., `Enrollments(student_id, course_id)`) is used, with each column as a Foreign Key, and usually `(student_id, course_id)` together as the composite Primary Key.

---

# SECTION 2 — Database Design & Normalization

**Q21. What is data redundancy?**

> Storing the same piece of data in multiple places. It's a problem because it wastes storage and, more importantly, makes it easy for copies to fall out of sync.

**Q22. What is data inconsistency?**

> When redundant copies of the same fact get updated in some places but not others, so the database ends up disagreeing with itself about the true value.

**Q23. What are Insert, Update, and Delete anomalies?**

> - **Insert anomaly:** You can't add a new department unless you also have an employee to insert alongside it (if department data only lives inside the Employees table).
> - **Update anomaly:** If a department renames, you must update every employee row referencing it — miss one, and the data disagrees with itself.
> - **Delete anomaly:** Deleting the last employee in a department accidentally wipes out all record of that department too.

**Q24. What is a Functional Dependency?**

> `X → Y` means the value of X determines the value of Y. In `Employees(id, name, dept_id, dept_name)`: `id → name, dept_id` and `dept_id → dept_name`.

**Q25. What is Normalization, and why is it needed?**

> The process of organizing tables to reduce redundancy and eliminate anomalies, by splitting tables based on functional dependencies. It's needed to keep data consistent and easy to maintain.

**Q26. What is 1NF?**

> Every column must hold atomic (indivisible) values, with no repeating groups. A table violates 1NF if, say, `phone_numbers` stores `"9876543210, 9123456789"` in one cell. Fix: move phone numbers into their own table, one number per row, linked by a foreign key.

**Q27. What is 2NF?**

> 1NF plus: no *partial dependency* — every non-key column must depend on the **whole** composite Primary Key, not just part of it. Only relevant when the PK is composite. Example: `OrderItems(order_id, product_id, product_name, quantity)` with PK `(order_id, product_id)` — `product_name` depends only on `product_id`, which violates 2NF.

**Q28. What is 3NF? Explain transitive dependency.**

> 2NF plus: no *transitive dependency* — a non-key column must not depend on another non-key column. Example: in `Employees(id, dept_id, dept_name)`, `id → dept_id → dept_name` is transitive. Fix: move `dept_name` into its own `Departments` table.

**Q29. What is BCNF, and how is it different from 3NF?**

> A stricter version of 3NF: for every functional dependency `X → Y`, X must be a super key — no exceptions. 3NF allows a narrow exception when Y is part of a candidate key; BCNF doesn't. Mostly matters when a table has multiple overlapping candidate keys.

**Q30. Normalize `Orders(order_id, customer_name, customer_city, product_name, product_price, quantity)` to 3NF.**

> ```sql
> Customers(customer_id PK, customer_name, customer_city)
> Products(product_id PK, product_name, product_price)
> Orders(order_id PK, customer_id FK, order_date)
> OrderItems(order_id FK, product_id FK, quantity)  -- composite PK (order_id, product_id)
> ```
> 
> This removes the repeated customer/product data and splits the table by entity.

**Q31. What is denormalization, and why would you use it?**

> Deliberately reintroducing redundancy (e.g., duplicating or pre-joining columns) to cut down on JOINs, trading some redundancy/maintenance risk for faster reads — common in reporting/analytics systems.

**Q32. Normalized or denormalized for a read-heavy reporting system?**

> Denormalized. Reporting systems mostly *read* and rarely write, so redundancy risk is low, while fewer JOINs make queries much faster. Normalization mainly protects write consistency, which reporting doesn't need much of.

**Q33. How do keys change as you normalize toward 3NF?**

> An unnormalized table often has no clean key, or a wide "natural" key with lots of repeated descriptive data per row. As you normalize, each new table gets its own simple Primary Key, and Foreign Keys replace the repeated values, linking tables instead of duplicating data.

**Q34. Design a normalized (3NF) schema for a food-delivery app.**

> ```sql
> Restaurants(id PK, name, city)
> MenuItems(id PK, restaurant_id FK, item_name, price)
> Customers(id PK, name, phone)
> Orders(id PK, customer_id FK, restaurant_id FK, order_date, status)
> OrderItems(order_id FK, menu_item_id FK, quantity)  -- composite PK
> ```

**Q35. How can a Composite Key violate 2NF but not 1NF?**

> `OrderItems(order_id, product_id, product_name, quantity)` with PK `(order_id, product_id)` is fine for 1NF (values are atomic, no repeating groups). But it breaks 2NF because `product_name` depends only on `product_id` — part of the key, not the whole key — a partial dependency that 1NF doesn't check for.

---

# SECTION 3 — SQL Fundamentals

**Q36. What are SQL's main sub-languages?**

> - **DDL** — defines structure: `CREATE`, `ALTER`, `DROP`
> - **DML** — modifies data: `INSERT`, `UPDATE`, `DELETE`
> - **DQL** — reads data: `SELECT`
> - **DCL** — permissions: `GRANT`, `REVOKE`
> - **TCL** — transactions: `COMMIT`, `ROLLBACK`, `SAVEPOINT`

**Q37. Give one example command each for DDL, DML, DQL, DCL, TCL.**

> ```sql
> -- DDL
> CREATE TABLE Employees (id INT PRIMARY KEY, name VARCHAR(50));
> -- DML
> INSERT INTO Employees VALUES (1, 'Rahul');
> -- DQL
> SELECT * FROM Employees;
> -- DCL
> GRANT SELECT ON Employees TO analyst_role;
> -- TCL
> COMMIT;
> ```

**Q38. Retrieve all columns from Employees.**

> ```sql
> SELECT * FROM Employees;
> ```

**Q39. Retrieve only name and salary.**

> ```sql
> SELECT name, salary FROM Employees;
> ```

**Q40. Fetch employees with salary greater than 60000.**

> ```sql
> SELECT * FROM Employees WHERE salary > 60000;
> ```

**Q41. Fetch distinct department IDs.**

> ```sql
> SELECT DISTINCT dept_id FROM Employees;
> ```

**Q42. List employees ordered by salary, descending.**

> ```sql
> SELECT * FROM Employees ORDER BY salary DESC;
> ```

**Q43. Fetch the top 3 highest-paid employees.**

> ```sql
> SELECT * FROM Employees ORDER BY salary DESC LIMIT 3;
> ```

**Q44. Fetch employees with salary between 50000 and 80000.**

> ```sql
> SELECT * FROM Employees WHERE salary BETWEEN 50000 AND 80000;
> ```

**Q45. Fetch employees in department 10 or 20.**

> ```sql
> SELECT * FROM Employees WHERE dept_id IN (10, 20);
> ```

**Q46. Fetch employees whose name starts with 'A'.**

> ```sql
> SELECT * FROM Employees WHERE name LIKE 'A%';
> ```

**Q47. Fetch employees with no manager assigned.**

> ```sql
> SELECT * FROM Employees WHERE manager_id IS NULL;
> -- opposite:
> SELECT * FROM Employees WHERE manager_id IS NOT NULL;
> ```

**Q48. Insert a new employee.**

> ```sql
> INSERT INTO Employees (id, name, salary, dept_id, manager_id, join_date)
> VALUES (6, 'Sanjay', 65000, 20, 2, '2023-05-01');
> ```

**Q49. Update employee id 3's salary to 65000.**

> ```sql
> UPDATE Employees SET salary = 65000 WHERE id = 3;
> ```

**Q50. Delete employee id 5. What's the risk of DELETE without WHERE?**

> ```sql
> DELETE FROM Employees WHERE id = 5;
> ```
> 
> Without a WHERE clause, `DELETE FROM Employees;` removes **every row** in the table — one of the most common and costly production mistakes. Always test the same WHERE clause with a SELECT first.

---

# SECTION 4 — SQL Operators & Conditions

**Q51. Employees in department 10 AND salary greater than 50000.**

> ```sql
> SELECT * FROM Employees WHERE dept_id = 10 AND salary > 50000;
> ```

**Q52. Employees in department 10 OR department 30.**

> ```sql
> SELECT * FROM Employees WHERE dept_id = 10 OR dept_id = 30;
> ```

**Q53. Employees NOT in department 10.**

> ```sql
> SELECT * FROM Employees WHERE NOT dept_id = 10;
> ```

**Q54. What comparison operators exist? Write a query using `<>`.**

> Operators: `=`, `<>` (or `!=`), `>`, `<`, `>=`, `<=`
> 
> ```sql
> SELECT * FROM Employees WHERE dept_id <> 10;
> ```

**Q55. Compute each employee's annual salary.**

> ```sql
> SELECT name, salary * 12 AS annual_salary FROM Employees;
> ```

**Q56. Employees who joined between two dates.**

> ```sql
> SELECT * FROM Employees WHERE join_date BETWEEN '2021-01-01' AND '2022-12-31';
> ```

**Q57. Names containing 'a' and ending in 'n'.**

> ```sql
> SELECT * FROM Employees WHERE name LIKE '%a%n';
> ```
> 
> `%` matches any number of characters; `_` matches exactly one character.

**Q58. Department 10 or 20, but not managed by employee id 1.**

> ```sql
> SELECT * FROM Employees
> WHERE dept_id IN (10, 20)
>   AND NOT manager_id = 1;
> ```

**Q59. Employees with NULL manager_id or NULL dept_id — why doesn't `= NULL` work?**

> ```sql
> SELECT * FROM Employees WHERE manager_id IS NULL OR dept_id IS NULL;
> ```
> 
> `= NULL` never matches anything, because NULL means "unknown," and comparing unknown to anything with `=` gives "unknown," not true. That's why SQL needs the special `IS NULL` / `IS NOT NULL` operators.

**Q60. Combine BETWEEN, IN, and comparison operators in one query.**

> ```sql
> SELECT name, salary
> FROM Employees
> WHERE salary BETWEEN 40000 AND 100000
>   AND dept_id IN (10, 20, 30);
> ```

---

# SECTION 5 — SQL Joins

**Q61. Why are JOINs needed?**

> They let you combine rows from two related tables using a matching column (typically Primary Key ↔ Foreign Key), so you can pull in related data (like a department's name) instead of storing it redundantly on every employee row.

**Q62. What is an INNER JOIN? Retrieve employees with their department names.**

> INNER JOIN returns only rows that have a match in **both** tables.
> 
> ```sql
> SELECT e.name, d.dept_name
> FROM Employees e
> INNER JOIN Departments d ON e.dept_id = d.id;
> ```

**Q63. What is a LEFT JOIN? List all employees, including those without a department.**

> LEFT JOIN returns every row from the left table, filling in NULLs where the right table has no match.
> 
> ```sql
> SELECT e.name, d.dept_name
> FROM Employees e
> LEFT JOIN Departments d ON e.dept_id = d.id;
> ```

**Q64. What is a RIGHT JOIN? List all departments, including empty ones.**

> RIGHT JOIN returns every row from the right table, with NULLs where the left table has no match.
> 
> ```sql
> SELECT e.name, d.dept_name
> FROM Employees e
> RIGHT JOIN Departments d ON e.dept_id = d.id;
> ```

**Q65. What is a FULL OUTER JOIN? Return all employees and all departments.**

> Returns every row from both tables, matched where possible, NULLs elsewhere.
> 
> ```sql
> SELECT e.name, d.dept_name
> FROM Employees e
> FULL OUTER JOIN Departments d ON e.dept_id = d.id;
> ```

**Q66. What is a SELF JOIN? List each employee with their manager's name.**

> A SELF JOIN joins a table to itself (using aliases) to compare rows within the same table.
> 
> ```sql
> SELECT e.name AS employee, m.name AS manager
> FROM Employees e
> LEFT JOIN Employees m ON e.manager_id = m.id;
> ```

**Q67. What is a CROSS JOIN? How is it different from INNER JOIN?**

> CROSS JOIN returns the Cartesian product — every row of one table paired with every row of the other, with no matching condition at all. Useful for generating combinations, e.g., every size/color pairing for a product.
> 
> ```sql
> SELECT * FROM Employees CROSS JOIN Departments;
> ```

**Q68. Employees with department name, filtered by salary > 55000 (JOIN + WHERE).**

> ```sql
> SELECT e.name, d.dept_name
> FROM Employees e
> INNER JOIN Departments d ON e.dept_id = d.id
> WHERE e.salary > 55000;
> ```

**Q69. Chain a JOIN to Departments and a LEFT JOIN to Projects.**

> ```sql
> SELECT e.name, d.dept_name, p.project_name
> FROM Employees e
> INNER JOIN Departments d ON e.dept_id = d.id
> LEFT JOIN Projects p ON e.id = p.emp_id;
> ```
> 
> Each JOIN adds one more related table; the LEFT JOIN keeps employees who have no project.

**Q70. Combine INNER JOIN, LEFT JOIN, and a SELF JOIN in one query.**

> ```sql
> SELECT e.name AS employee, d.dept_name, m.name AS manager
> FROM Employees e
> INNER JOIN Departments d ON e.dept_id = d.id
> LEFT JOIN Employees m ON e.manager_id = m.id;
> ```

**Q71. Count employees per department (JOIN + GROUP BY preview).**

> ```sql
> SELECT d.dept_name, COUNT(e.id) AS emp_count
> FROM Departments d
> INNER JOIN Employees e ON d.id = e.dept_id
> GROUP BY d.dept_name;
> ```

**Q72. ON clause vs WHERE clause in an outer JOIN — what's the difference?**

> A condition in the **ON** clause is applied *while deciding the join*, so unmatched right-table rows still appear (with NULLs). A condition in **WHERE** is applied *after* the join, so it can filter out those NULL rows the outer join just added — silently turning it back into something like an INNER JOIN. Example: `LEFT JOIN Departments d ON e.dept_id = d.id AND d.location = 'Delhi'` keeps all employees; `... WHERE d.location = 'Delhi'` drops any employee without a Delhi department.

**Q73. Find departments with zero employees.**

> ```sql
> SELECT d.*
> FROM Departments d
> LEFT JOIN Employees e ON d.id = e.dept_id
> WHERE e.id IS NULL;
> ```

**Q74. Employees who work in the same department as 'Rahul' (SELF JOIN).**

> ```sql
> SELECT e2.*
> FROM Employees e1
> JOIN Employees e2 ON e1.dept_id = e2.dept_id
> WHERE e1.name = 'Rahul' AND e2.name <> 'Rahul';
> ```

**Q75. What happens if you JOIN two tables with no ON condition?**

> It behaves like a CROSS JOIN — every row of Table A paired with every row of Table B, giving you (rows in A) × (rows in B) results. This is almost always accidental and can silently blow up query size and performance.

**Q76. Each customer with their total number of orders.**

> ```sql
> SELECT c.name, COUNT(o.id) AS total_orders
> FROM Customers c
> LEFT JOIN Orders o ON c.id = o.customer_id
> GROUP BY c.name;
> ```

**Q77. Employees who report directly to 'Amit'.**

> ```sql
> SELECT e.*
> FROM Employees e
> JOIN Employees m ON e.manager_id = m.id
> WHERE m.name = 'Amit';
> ```

**Q78. Departments in 'Delhi' with their employees, including empty departments.**

> ```sql
> SELECT d.dept_name, e.name
> FROM Departments d
> LEFT JOIN Employees e ON d.id = e.dept_id
> WHERE d.location = 'Delhi';
> ```
> 
> This is safe because `d.location` is a **left-table** column — filtering on it in WHERE doesn't strip out the NULL-padded rows the outer join creates.

**Q79. Debugging: a LEFT JOIN is behaving like an INNER JOIN. Why?**

> Most likely: a filter on a **right-table** column was placed in the WHERE clause instead of the ON clause (see Q72). Since unmatched right-side rows show NULL there, `WHERE d.location = 'Delhi'` silently excludes them. Fix: move that condition into the ON clause.

**Q80. Employees, departments, and projects, handling missing departments and missing projects.**

> ```sql
> SELECT e.name, d.dept_name, p.project_name
> FROM Employees e
> LEFT JOIN Departments d ON e.dept_id = d.id
> LEFT JOIN Projects p ON e.id = p.emp_id;
> ```
> 
> Both LEFT JOINs preserve every employee row even when department or project is missing.

---

# SECTION 6 — SQL Aggregate Functions

**Q81. What are aggregate functions?**

> Functions that compute a single summary value from a set of rows: `COUNT` (row count), `SUM` (total), `AVG` (average), `MIN` (smallest), `MAX` (largest).

**Q82. Count total employees.**

> ```sql
> SELECT COUNT(*) FROM Employees;
> ```

**Q83. Total salary expenditure.**

> ```sql
> SELECT SUM(salary) FROM Employees;
> ```

**Q84. Average salary.**

> ```sql
> SELECT AVG(salary) FROM Employees;
> ```

**Q85. Minimum and maximum salary.**

> ```sql
> SELECT MIN(salary), MAX(salary) FROM Employees;
> ```

**Q86. Number of employees per department.**

> ```sql
> SELECT dept_id, COUNT(*) AS emp_count
> FROM Employees
> GROUP BY dept_id;
> ```

**Q87. Average salary per department.**

> ```sql
> SELECT dept_id, AVG(salary) AS avg_salary
> FROM Employees
> GROUP BY dept_id;
> ```

**Q88. Total salary per department, only employees who joined after 2021-01-01.**

> ```sql
> SELECT dept_id, SUM(salary) AS total_salary
> FROM Employees
> WHERE join_date > '2021-01-01'
> GROUP BY dept_id;
> ```

**Q89. Departments with more than 1 employee.**

> ```sql
> SELECT dept_id, COUNT(*) AS emp_count
> FROM Employees
> GROUP BY dept_id
> HAVING COUNT(*) > 1;
> ```

**Q90. WHERE vs HAVING — when does WHERE fail?**

> WHERE filters individual rows **before** grouping, so it can't reference an aggregate. HAVING filters **groups** after aggregation, so it can. `WHERE COUNT(*) > 1` fails because COUNT doesn't exist yet at that stage — you need `HAVING COUNT(*) > 1`.

**Q91. Department names with employee counts (JOIN + GROUP BY).**

> ```sql
> SELECT d.dept_name, COUNT(e.id) AS emp_count
> FROM Departments d
> JOIN Employees e ON d.id = e.dept_id
> GROUP BY d.dept_name;
> ```

**Q92. Department names with average salary above 60000 (JOIN + GROUP BY + HAVING).**

> ```sql
> SELECT d.dept_name, AVG(e.salary) AS avg_salary
> FROM Departments d
> JOIN Employees e ON d.id = e.dept_id
> GROUP BY d.dept_name
> HAVING AVG(e.salary) > 60000;
> ```

**Q93. Highest-paid employee in each department.**

> ```sql
> SELECT e.*
> FROM Employees e
> JOIN (
>     SELECT dept_id, MAX(salary) AS max_salary
>     FROM Employees
>     GROUP BY dept_id
> ) m ON e.dept_id = m.dept_id AND e.salary = m.max_salary;
> ```

**Q94. Customers with more than 2 orders.**

> ```sql
> SELECT c.name, COUNT(o.id) AS order_count
> FROM Customers c
> JOIN Orders o ON c.id = o.customer_id
> GROUP BY c.name
> HAVING COUNT(o.id) > 2;
> ```

**Q95. Departments in Bangalore: employee count and average salary, only where count > 1.**

> ```sql
> SELECT d.dept_name, COUNT(e.id) AS emp_count, AVG(e.salary) AS avg_salary
> FROM Departments d
> JOIN Employees e ON d.id = e.dept_id
> WHERE d.location = 'Bangalore'
> GROUP BY d.dept_name
> HAVING COUNT(e.id) > 1;
> ```

---

# SECTION 7 — Subqueries

**Q96. What is a subquery? What is a scalar subquery?**

> A subquery is a query nested inside another query. A scalar subquery returns exactly one value (one row, one column), so it can be used anywhere a single value is expected — like in a WHERE comparison.

**Q97. Employees earning more than the average salary.**

> ```sql
> SELECT * FROM Employees
> WHERE salary > (SELECT AVG(salary) FROM Employees);
> ```

**Q98. Second-highest salary.**

> ```sql
> SELECT MAX(salary) FROM Employees
> WHERE salary < (SELECT MAX(salary) FROM Employees);
> ```

**Q99. What is a multi-row subquery? Employees in Delhi departments.**

> A multi-row subquery returns more than one value, typically used with IN.
> 
> ```sql
> SELECT * FROM Employees
> WHERE dept_id IN (SELECT id FROM Departments WHERE location = 'Delhi');
> ```

**Q100. What is EXISTS? Departments with at least one employee.**

> EXISTS checks whether a subquery returns **any** rows (true/false) — it doesn't care about the actual values.
> 
> ```sql
> SELECT * FROM Departments d
> WHERE EXISTS (SELECT 1 FROM Employees e WHERE e.dept_id = d.id);
> ```

**Q101. What is NOT EXISTS? Customers who never placed an order.**

> ```sql
> SELECT * FROM Customers c
> WHERE NOT EXISTS (SELECT 1 FROM Orders o WHERE o.customer_id = c.id);
> ```

**Q102. IN vs EXISTS — when to prefer which?**

> IN compares against a fixed list of values — good for simple, static, non-correlated lists. EXISTS just checks row presence and can stop at the first match, which often performs better for correlated existence checks on large tables. Watch out: `NOT IN` breaks if the list contains a NULL (see Q108) — `NOT EXISTS` doesn't have that problem.

**Q103. Employees earning above their own department's average (correlated subquery).**

> ```sql
> SELECT e.*
> FROM Employees e
> WHERE e.salary > (
>     SELECT AVG(e2.salary) FROM Employees e2 WHERE e2.dept_id = e.dept_id
> );
> ```

**Q104. Correlated vs non-correlated subquery?**

> A correlated subquery references a column from the outer query, so it effectively re-evaluates per outer row. A non-correlated subquery is fully independent and runs once, standalone.

**Q105. What are ANY and ALL? Employees earning more than everyone in department 10.**

> ANY is true if the condition holds for at least one row from the subquery; ALL requires it to hold for every row.
> 
> ```sql
> SELECT * FROM Employees
> WHERE salary > ALL (SELECT salary FROM Employees WHERE dept_id = 10);
> ```

**Q106. Departments with more employees than the average department size.**

> ```sql
> SELECT dept_id, COUNT(*) AS emp_count
> FROM Employees
> GROUP BY dept_id
> HAVING COUNT(*) > (
>     SELECT AVG(cnt) FROM (
>         SELECT COUNT(*) AS cnt FROM Employees GROUP BY dept_id
>     ) sub
> );
> ```

**Q107. Employees who have never been anyone's manager.**

> ```sql
> SELECT * FROM Employees
> WHERE id NOT IN (SELECT manager_id FROM Employees WHERE manager_id IS NOT NULL);
> ```

**Q108. Why is NOT IN risky with NULLs? Rewrite safely with NOT EXISTS.**

> If the subquery's result set contains even one NULL, `NOT IN` returns an **empty result for every row** — because comparing anything to NULL is "unknown," and `NOT (... OR unknown)` never evaluates to true. Safer version:
> 
> ```sql
> SELECT * FROM Employees e
> WHERE NOT EXISTS (
>     SELECT 1 FROM Employees m WHERE m.manager_id = e.id
> );
> ```

**Q109. Top 2 highest-paid employees per department (correlated subquery approach).**

> ```sql
> SELECT * FROM Employees e1
> WHERE (
>     SELECT COUNT(*) FROM Employees e2
>     WHERE e2.dept_id = e1.dept_id AND e2.salary > e1.salary
> ) < 2
> ORDER BY e1.dept_id, e1.salary DESC;
> ```

**Q110. Departments with more than 5 employees AND above-average salary (nested subqueries).**

> ```sql
> SELECT d.dept_name, COUNT(e.id) AS emp_count, AVG(e.salary) AS avg_salary
> FROM Departments d
> JOIN Employees e ON d.id = e.dept_id
> GROUP BY d.dept_name
> HAVING COUNT(e.id) > 5
>    AND AVG(e.salary) > (SELECT AVG(salary) FROM Employees);
> ```

---

# SECTION 8 — Advanced SQL

**Q111. What is CASE? Label employees High/Medium/Low earners.**

> ```sql
> SELECT name, salary,
>   CASE
>     WHEN salary >= 80000 THEN 'High'
>     WHEN salary >= 55000 THEN 'Medium'
>     ELSE 'Low'
>   END AS salary_band
> FROM Employees;
> ```

**Q112. What is COALESCE? Show 'No Manager' instead of NULL.**

> COALESCE returns the first non-NULL value from a list.
> 
> ```sql
> SELECT name, COALESCE(manager_id, 'No Manager') AS manager
> FROM Employees;
> ```
> 
> (In practice `manager_id` is numeric, so you'd usually COALESCE a joined manager *name* rather than the raw id.)

**Q113. What is NULLIF, and how is it different from COALESCE?**

> COALESCE returns the first non-NULL value from a list of expressions. NULLIF returns NULL if two expressions are equal, otherwise returns the first — handy for turning a "sentinel" value like 0 into NULL so it's excluded from calculations, e.g., `NULLIF(bonus, 0)`.

**Q114. What is a CTE? Rewrite the average-salary-per-department query using one.**

> A CTE (`WITH ... AS (...)`) is a named, temporary result set you can reference like a table within the same query — it makes complex queries easier to read.
> 
> ```sql
> WITH dept_avg AS (
>   SELECT dept_id, AVG(salary) AS avg_salary
>   FROM Employees
>   GROUP BY dept_id
> )
> SELECT * FROM dept_avg;
> ```

**Q115. What is a Recursive CTE (basic)? Find an employee's management chain.**

> ```sql
> WITH RECURSIVE mgmt_chain AS (
>   SELECT id, name, manager_id FROM Employees WHERE id = 3
>   UNION ALL
>   SELECT e.id, e.name, e.manager_id
>   FROM Employees e
>   JOIN mgmt_chain c ON e.id = c.manager_id
> )
> SELECT * FROM mgmt_chain;
> ```
> 
> It starts from a base row, then repeatedly joins back to itself to walk up (or down) a hierarchy until no more rows match.

**Q116. Why can't GROUP BY show per-row detail alongside an aggregate?**

> GROUP BY collapses many rows into one summary row per group, so you lose individual row-level detail (e.g., you can see a department's average salary, but not each employee's own salary in the same row). This is exactly the gap window functions fill.

**Q117. What is PARTITION BY, and how is it different from GROUP BY?**

> PARTITION BY divides rows into groups for a window function, similar to GROUP BY — but it doesn't collapse rows. Each row keeps its identity while also seeing an aggregate computed over its partition.

**Q118. What is a Window Function? Show each employee's salary next to their department's total salary.**

> ```sql
> SELECT name, dept_id, salary,
>   SUM(salary) OVER (PARTITION BY dept_id) AS dept_total_salary
> FROM Employees;
> ```

**Q119. What does ROW_NUMBER() do? Assign row numbers by salary within each department.**

> ROW_NUMBER() assigns a unique, sequential number within each partition — no ties, no gaps.
> 
> ```sql
> SELECT name, dept_id, salary,
>   ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
> FROM Employees;
> ```

**Q120. What does RANK() do, vs ROW_NUMBER()?**

> RANK() also numbers rows in order, but tied rows get the **same** rank, and the next rank **skips** ahead accordingly (e.g., 1, 2, 2, 4). ROW_NUMBER() never repeats or skips.

**Q121. What does DENSE_RANK() do, vs RANK()?**

> DENSE_RANK() is like RANK(), but it does **not** skip numbers after a tie (e.g., 1, 2, 2, 3 instead of 1, 2, 2, 4).

**Q122. Top 2 highest-paid employees per department, using ranking functions.**

> ```sql
> SELECT * FROM (
>   SELECT e.*, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
>   FROM Employees e
> ) t
> WHERE rnk <= 2;
> ```
> 
> This is usually cleaner and easier to reason about than the correlated-subquery version from Q109, and it handles ties predictably.

**Q123. What does LAG() do? Show each employee's salary next to the previously joined employee's salary.**

> LAG() returns a value from a previous row in the ordered window.
> 
> ```sql
> SELECT name, join_date, salary,
>   LAG(salary) OVER (ORDER BY join_date) AS prev_salary
> FROM Employees;
> ```

**Q124. What does LEAD() do? Show each order's amount next to the customer's next order amount.**

> LEAD() is the mirror of LAG() — it looks ahead to a following row.
> 
> ```sql
> SELECT customer_id, order_date, amount,
>   LEAD(amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_order_amount
> FROM Orders;
> ```

**Q125. Remove duplicate rows using ROW_NUMBER().**

> ```sql
> WITH ranked AS (
>   SELECT *, ROW_NUMBER() OVER (PARTITION BY <dup_columns> ORDER BY id) AS rn
>   FROM SomeTable
> )
> DELETE FROM SomeTable WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
> ```

**Q126. Running total of salaries ordered by join_date.**

> ```sql
> SELECT name, join_date, salary,
>   SUM(salary) OVER (ORDER BY join_date) AS running_total
> FROM Employees;
> ```

**Q127. Count High/Medium/Low earners per department using CASE + aggregation.**

> ```sql
> SELECT dept_id,
>   SUM(CASE WHEN salary >= 80000 THEN 1 ELSE 0 END) AS high,
>   SUM(CASE WHEN salary >= 55000 AND salary < 80000 THEN 1 ELSE 0 END) AS medium,
>   SUM(CASE WHEN salary < 55000 THEN 1 ELSE 0 END) AS low
> FROM Employees
> GROUP BY dept_id;
> ```

**Q128. Second-highest-paid employee in each department, using window functions.**

> ```sql
> WITH ranked AS (
>   SELECT *, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
>   FROM Employees
> )
> SELECT * FROM ranked WHERE rnk = 2;
> ```

**Q129. Flag employees paid less than the person who joined right before them (same department).**

> ```sql
> SELECT name, dept_id, salary,
>   LAG(salary) OVER (PARTITION BY dept_id ORDER BY join_date) AS prev_salary,
>   CASE
>     WHEN salary < LAG(salary) OVER (PARTITION BY dept_id ORDER BY join_date) THEN 'Lower than predecessor'
>     ELSE 'OK'
>   END AS flag
> FROM Employees;
> ```

**Q130. Correlated subquery vs window function for "Nth highest salary per group" — which is better?**

> Both work. Correlated subqueries are more portable to very old SQL engines, but they conceptually re-run per outer row, which scales poorly on large tables and reads less clearly once nested. Window functions (RANK/DENSE_RANK) are usually more readable, computed in a single pass by the engine, and generally perform better on large datasets — most interviewers expect the window-function approach for this kind of problem.

---

# SECTION 9 — Transactions

**Q131. What is a transaction?**

> A sequence of one or more SQL operations executed as a single logical unit of work — either all of it succeeds (commits) or none of it does (rolls back).

**Q132. What are the ACID properties?**

> **A**tomicity, **C**onsistency, **I**solation, **D**urability.

**Q133. Explain Atomicity with a bank transfer example.**

> In a transfer (debit account A, credit account B), both updates must succeed together or neither happens. If the credit fails after the debit already succeeded, the whole transaction rolls back the debit too — so money is never lost mid-transfer.

**Q134. Explain Consistency in the bank transfer example.**

> The transaction must move the database from one valid state to another, respecting every rule/constraint. In a transfer, the invariant "total money across both accounts stays the same" (and no account goes negative, if that's disallowed) must hold both before and after.

**Q135. Explain Isolation — why do concurrent transactions need to be isolated?**

> Two transactions running at the same time must not see each other's in-progress, uncommitted changes — otherwise one could read and act on data that later gets rolled back, producing incorrect results.

**Q136. Explain Durability.**

> Once a transaction commits, its changes are permanent — they survive even a crash or power failure right afterward, because they've been persisted to durable storage.

**Q137. What do COMMIT and ROLLBACK do? Write a bank-transfer transaction.**

> ```sql
> BEGIN TRANSACTION;
> UPDATE Accounts SET balance = balance - 1000 WHERE id = 1;
> UPDATE Accounts SET balance = balance + 1000 WHERE id = 2;
> -- if both succeed:
> COMMIT;
> -- if either fails:
> ROLLBACK;
> ```

**Q138. What is a SAVEPOINT?**

> A marker inside a transaction that you can roll back to **without** discarding the whole transaction — useful when one part of a multi-step operation fails but earlier successful steps should be kept.

**Q139. Why must "create order" and "deduct inventory" happen in the same transaction?**

> If they're separate, a crash between the two steps can leave an order recorded with no inventory deducted (overselling), or inventory deducted with no order created (lost stock). Both changes must succeed or fail together to keep the system consistent.

**Q140. Design transaction boundaries with SAVEPOINTs for a multi-item bulk order.**

> ```sql
> BEGIN TRANSACTION;
>   UPDATE Inventory SET qty = qty - 5 WHERE product_id = 101;
>   SAVEPOINT sp1;
>   UPDATE Inventory SET qty = qty - 3 WHERE product_id = 102;
>   -- if this item fails:
>   ROLLBACK TO sp1;   -- undoes only product_id 102's update
>   -- continue with the next item, or COMMIT what succeeded
> COMMIT;
> ```

---

# SECTION 10 — Concurrency & Isolation

**Q141. What happens when two transactions run concurrently on the same data?**

> One transaction's in-progress or uncommitted changes can be seen, overwritten, or interfered with by the other, leading to incorrect reads or lost updates — unless the database controls how they interleave.

**Q142. What is a Dirty Read?**

> Transaction A reads a row that Transaction B has changed but not yet committed. If B later rolls back, A has read data that never actually existed.

**Q143. What is a Non-Repeatable Read? How is it different from a Dirty Read?**

> Transaction A reads the same row twice and gets **different** values because Transaction B updated and **committed** a change in between. Unlike a Dirty Read, the data A saw was genuinely committed — it just changed between A's two reads.

**Q144. What is a Phantom Read? How is it different from a Non-Repeatable Read?**

> Transaction A re-runs the same filtered query twice and gets a **different set of rows** (extra or missing), because Transaction B inserted or deleted matching rows in between. It's about the row *count/membership* changing, not an existing row's value changing.

**Q145. What is a Lost Update? Give a bank-balance example.**

> A reads balance = 1000. B reads balance = 1000. A adds 100 and writes 1100. B (unaware of A's change) adds 200 to its own stale read and writes 1200 — A's update is silently lost, even though both "succeeded."

**Q146. List isolation levels from least to most strict.**

> READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ → SERIALIZABLE.

**Q147. Which level prevents Dirty Reads but not Non-Repeatable Reads? Which prevents both but not Phantom Reads?**

> READ COMMITTED prevents Dirty Reads but still allows Non-Repeatable Reads and Phantom Reads. REPEATABLE READ prevents both Dirty Reads and Non-Repeatable Reads, but can still allow Phantom Reads (depending on the database engine).

**Q148. Two customers both "win" the last item in stock — which problem is this, and how do you fix it?**

> This is a Lost Update problem. Fix it with a stricter isolation level (SERIALIZABLE) or explicit row-level locking (e.g., `SELECT ... FOR UPDATE`), so the second transaction waits for, and sees, the first transaction's stock decrement before proceeding.

---

# SECTION 11 — Indexing

**Q149. What is an index?**

> A separate data structure the database maintains alongside a table to speed up lookups on specific columns — like an index at the back of a book, letting the database find rows without scanning every one.

**Q150. Why are indexes needed? How do they improve performance?**

> Without an index, the database scans every row (a full table scan) to find matches. With an index, it navigates directly to matching rows (typically via a B+Tree), dramatically cutting the rows examined — especially valuable on large tables.

**Q151. Clustered Index vs Non-Clustered Index?**

> A Clustered Index determines the **physical** storage order of a table's rows (usually built on the Primary Key) — a table can have only one. A Non-Clustered Index is a separate structure with pointers back to the actual rows, leaving physical row order unchanged — a table can have many.

**Q152. What is a Unique Index?**

> An index that enforces all indexed values are distinct, just like a UNIQUE constraint. In most databases, adding a UNIQUE constraint automatically creates a unique index behind the scenes.

**Q153. What is a Composite Index? Does column order matter?**

> An index on two or more columns together, useful when queries often filter or sort by that same combination. Column order matters a lot: an index on `(a, b)` helps queries filtering on `a` alone or on `a AND b`, but generally doesn't help a query filtering on `b` alone — the **leftmost** column must be used.

**Q154. What is Index Selectivity?**

> The ratio of distinct values to total rows in a column. High selectivity (many distinct values, like `email`) makes an index very effective. Low selectivity (few distinct values, like `gender`) means the index barely narrows results, so the optimizer often skips it and scans the table instead.

**Q155. What is a B-Tree/B+Tree, and why do database indexes use it?**

> A balanced, sorted tree structure where data lives in leaf nodes (linked together for fast range scans) and internal nodes act as navigation. Databases use it because lookups, inserts, and range queries stay efficient (O(log n)) even as the table grows huge, and it suits disk-based storage well (few, wide nodes minimize disk reads).

**Q156. What are the advantages of indexes?**

> Much faster lookups/filters/sorts/joins on indexed columns, uniqueness enforcement (for unique indexes), and can speed up ORDER BY/GROUP BY when the index already matches the required order.

**Q157. What are the disadvantages of indexes?**

> Extra storage, and every INSERT/UPDATE/DELETE must also update the index, slowing down writes. Indexing every column bloats storage and write latency for little benefit on low-selectivity or rarely-queried columns.

**Q158. When should an index NOT be used?**

> Small tables (a full scan is already fast), low-selectivity columns (barely narrows results), columns rarely used in WHERE/JOIN/ORDER BY, and write-heavy tables where index-maintenance overhead outweighs any read benefit.

**Q159. For `WHERE dept_id = 10 AND salary > 50000`, what should you index?**

> A composite index on `(dept_id, salary)` — `dept_id` leads (equality filter, narrows rows fast), and `salary` as the second column lets the engine efficiently apply the range condition within that department. Indexing `salary` alone would be weaker since it's a range condition, not the most selective filter here.

**Q160. Design an indexing strategy for a 10-million-row Orders table filtered by customer_id and sorted by order_date.**

> A composite index on `(customer_id, order_date)`: `customer_id` leads (equality filter, narrows fast), and `order_date` as the second column also serves the sort, so the engine can avoid a separate sort step. If date-range filtering is common independent of customer, consider a separate index on `order_date` too.

---

# SECTION 12 — Query Optimization

**Q161. What is EXPLAIN, and why is it useful?**

> EXPLAIN shows the plan the database engine intends to use — which indexes (if any) it'll use, join order, estimated cost/rows — without running the query's actual side effects. It's the main tool for diagnosing why a query is slow.

**Q162. What is an Execution Plan?**

> The ordered set of steps the engine will take: scan type per table (full scan vs index scan/seek), join algorithm and order, estimated rows processed, and relative cost — used to spot expensive steps.

**Q163. Full Table Scan vs Index Scan — which is slower?**

> A Full Table Scan reads every row to find matches. An Index Scan/Seek uses the index to jump straight to matching rows. Full Table Scans are generally much slower on large tables.

**Q164. Why is `SELECT *` bad practice? How would you rewrite it?**

> It pulls every column even when only a few are needed, increasing I/O, network transfer, and memory use, and can prevent the optimizer from using a faster "index-only" (covering index) plan. It also breaks silently if the table's columns change. Rewrite by listing only what you need:
> 
> ```sql
> SELECT name, salary FROM Employees;
> ```

**Q165. Why does `WHERE YEAR(join_date) = 2022` prevent index use? Rewrite it index-friendly.**

> Applying a function to an indexed column forces the database to compute that function for every row before comparing, so it can't use the index's raw stored values directly — this usually forces a full table scan. Rewrite as a range on the raw column:
> 
> ```sql
> SELECT * FROM Employees
> WHERE join_date >= '2022-01-01' AND join_date < '2023-01-01';
> ```

**Q166. How does JOIN order and indexing on JOIN columns affect performance?**

> JOIN order determines how many intermediate rows the engine processes at each step — a good optimizer tries to filter down early. In a slow multi-table JOIN's execution plan, check: which table drives the join, whether Index Scans (not Full Table Scans) are used on the JOIN columns, and whether estimated vs actual row counts match (a big mismatch suggests stale statistics).

**Q167. How would a composite index help a query with both WHERE and GROUP BY?**

> A composite index covering both the WHERE column(s) and the GROUP BY column(s) — WHERE column(s) leading — lets the engine filter efficiently using the index, and if rows already arrive pre-sorted by the GROUP BY column, it can skip a separate sort/hash step for grouping.

**Q168. What's your checklist for debugging a slow query in production?**

> 1. Run EXPLAIN to see the current plan.
> 2. Check for Full Table Scans on large tables and confirm relevant indexes exist and are used.
> 3. Avoid `SELECT *` — fetch only needed columns.
> 4. Check JOIN order and whether JOIN columns are indexed.
> 5. Check for functions wrapped around indexed columns in WHERE.
> 6. Verify table statistics are up to date.
> 7. Consider composite indexes matching common WHERE + ORDER BY/GROUP BY patterns.

**Q169. A query went from 200ms to 8 seconds after the table grew to 10 million rows — how do you investigate?**

> Start with EXPLAIN to see the current plan and compare it to what likely ran before — at 10M rows, a query that was fine with a full scan on a small table may now hit an expensive full scan or a bad join order. Check whether WHERE/JOIN columns are indexed, whether a function wraps an indexed column (Q165), and whether statistics are stale after the growth (misleading the optimizer's row estimates). Fix by adding/adjusting indexes (possibly composite, matching the filter + sort pattern), rewriting non-sargable conditions, and updating statistics — then re-check EXPLAIN to confirm an Index Scan replaces the Full Table Scan.

**Q170. Design indexing + query strategy for "top 10 highest-paid employees per department, in a given city," on 10M rows.**

> Use a composite index on `Employees(dept_id, salary DESC)` to support fast per-department salary ordering, and an index on `Departments(location)` to support the city filter. JOIN using the indexed `dept_id`/`id` columns so the engine can seek instead of scan. For "top 10 per department," a window-function query — `DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC)` filtered to `<= 10` — is generally more optimizer-friendly and readable than an equivalent correlated subquery, since it's computed in a single pass and modern optimizers handle windowed top-N patterns well; the correlated-subquery version tends to degrade faster as the table grows.

---

# Quick Reference — Topic Coverage

| Topic | Questions |
| --- | --- |
| DBMS / RDBMS Basics | Q1–Q5 |
| Keys | Q6–Q11 |
| Constraints | Q13–Q17 |
| Relationships | Q12, Q18–Q20 |
| Normalization | Q21–Q35 |
| SQL Fundamentals | Q36–Q50 |
| Operators & Conditions | Q51–Q60 |
| JOINs | Q61–Q80 |
| Aggregates / GROUP BY | Q81–Q95 |
| Subqueries | Q96–Q110 |
| CASE / COALESCE / CTE | Q111–Q115 |
| Window Functions | Q116–Q130 |
| Transactions | Q131–Q140 |
| Concurrency & Isolation | Q141–Q148 |
| Indexing | Q149–Q160 |
| Query Optimization | Q161–Q170 |
