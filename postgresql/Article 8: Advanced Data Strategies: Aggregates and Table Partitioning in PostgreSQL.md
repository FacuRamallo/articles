# Article 8: Advanced Data Strategies: Aggregates and Table Partitioning in PostgreSQL

**Briefing:** Focus on techniques for advanced data analysis and managing large datasets. The first part will explain aggregate functions (COUNT, SUM, AVG, etc.), their use with the `GROUP BY` clause, and a brief mention of user-defined aggregates. The second part will provide a thorough explanation of table partitioning: its benefits (performance, manageability), the types of declarative partitioning (Range, List, Hash) with clear examples, and how partition pruning works.

---

## Introduction

Beyond basic data storage and retrieval, PostgreSQL offers powerful features for analyzing and managing large volumes of data efficiently. This article explores two key advanced strategies: **aggregate functions** for summarizing and deriving insights from data, and **table partitioning** for handling very large tables by dividing them into smaller, more manageable pieces. Mastering these techniques can significantly enhance both the analytical capabilities and the performance of your PostgreSQL databases.

---

## 1. Aggregates: Summarizing Your Data

In SQL, an **aggregate function** performs a calculation on a set of input rows (typically values within a column) and returns a single, summary result. Aggregates are indispensable for reporting, business intelligence, and any scenario requiring summarized data.

### Common Aggregate Functions

PostgreSQL provides a rich set of built-in aggregate functions. Some of the most frequently used include:

* **`COUNT(*)` or `COUNT(expression)`:** Counts the number of input rows. `COUNT(*)` counts all rows in a group, while `COUNT(expression)` counts rows where the expression is not NULL.
    ```sql
    -- Total number of employees
    SELECT COUNT(*) FROM employees;
    -- Number of distinct departments
    SELECT COUNT(DISTINCT department_id) FROM employees;
    ```
* **`SUM(expression)`:** Calculates the sum of all non-NULL input values (must be a numeric type).
    ```sql
    SELECT SUM(salary) FROM employees WHERE department_id = 10;
    ```
* **`AVG(expression)`:** Computes the average (arithmetic mean) of all non-NULL input values (must be a numeric type).
    ```sql
    SELECT AVG(order_total) FROM orders WHERE order_date >= '2024-01-01';
    ```
* **`MIN(expression)` and `MAX(expression)`:** Return the minimum and maximum non-NULL input value, respectively. Applicable to any sortable data type.
    ```sql
    SELECT MIN(unit_price), MAX(unit_price) FROM products;
    ```
* **`STRING_AGG(expression, delimiter)`:** Concatenates all non-NULL input string expressions into a single string, with each value separated by the specified delimiter.
    ```sql
    SELECT department_name, STRING_AGG(employee_name, ', ' ORDER BY employee_name) AS employee_list
    FROM departments JOIN employees USING (department_id)
    GROUP BY department_name;
    ```
* **`ARRAY_AGG(expression)`:** Collects all input values, including NULLs, into an array.
    ```sql
    SELECT product_category, ARRAY_AGG(product_name)
    FROM products
    GROUP BY product_category;
    ```

### Using Aggregates with `GROUP BY`

The true power of aggregate functions is unleashed when combined with the `GROUP BY` clause. The `GROUP BY` clause divides the rows of a table into groups where rows with the same values in the specified column(s) are grouped together. The aggregate function then calculates a summary value *for each group*.

```sql
SELECT
    p.category,
    COUNT(p.product_id) AS number_of_products,
    AVG(p.price) AS average_price,
    MAX(s.sale_date) AS latest_sale_date
FROM
    products p
LEFT JOIN
    sales s ON p.product_id = s.product_id
GROUP BY
    p.category
ORDER BY
    p.category;
