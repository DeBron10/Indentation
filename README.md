# Indentation

In **Oracle 19c / SQL Developer**, you have **two good ways** to retrieve:

* Tasks **ending in a specific month**
* Their **parents, grandparents, ancestors**

These two approaches are:

1️⃣ **Recursive CTE (`WITH` clause)** – SQL standard
2️⃣ **Oracle Hierarchical Query (`CONNECT BY`)** – Oracle-specific and often faster

I'll explain both clearly.

---

# 1️⃣ Recursive CTE (Standard SQL Way)

This works like a **loop inside SQL**.

### Query

```sql
WITH task_tree (
    id,
    sr_no,
    title,
    parent_id,
    start_date,
    end_date,
    percent_complete,
    indent
) AS (

    -- Step 1: Find tasks ending in April 2026
    SELECT
        id,
        sr_no,
        title,
        parent_id,
        start_date,
        end_date,
        percent_complete,
        indent
    FROM project_tasks
    WHERE end_date >= DATE '2026-04-01'
      AND end_date <  DATE '2026-05-01'

    UNION ALL

    -- Step 2: Recursively fetch parents
    SELECT
        p.id,
        p.sr_no,
        p.title,
        p.parent_id,
        p.start_date,
        p.end_date,
        p.percent_complete,
        p.indent
    FROM project_tasks p
    JOIN task_tree t
        ON p.id = t.parent_id
)

SELECT DISTINCT *
FROM task_tree
ORDER BY sr_no;
```

---

### How it Works

### Step 1 — Base Query

Find tasks **ending in April**

Example:

```
Market Research
Demographic Segmentation
```

---

### Step 2 — Recursive Part

It finds **parents of those tasks**

```
Concept & Market Definition
```

Then again:

```
Design and Development of Car
```

---

### Final Output

```
Design and Development of Car
Concept & Market Definition
Market Research
Demographic Segmentation
```

---

### Why `DISTINCT`?

Because a parent may appear multiple times if several children match.

---

### Advantages

✔ Works in **many databases**
✔ Standard SQL
✔ Flexible recursion

### Disadvantages

❌ Slightly **slower in Oracle** compared to `CONNECT BY`.

---

# 2️⃣ Oracle Hierarchical Query (CONNECT BY)

This is **Oracle's native tree traversal feature**.

It is usually **shorter and faster**.

---

### Query

```sql
SELECT *
FROM project_tasks
START WITH id IN (
    SELECT id
    FROM project_tasks
    WHERE end_date >= DATE '2026-04-01'
      AND end_date < DATE '2026-05-01'
)
CONNECT BY PRIOR parent_id = id
ORDER BY sr_no;
```

---

### How it Works

### START WITH

Start from tasks **ending in April**

Example:

```
Market Research
```

---

### CONNECT BY PRIOR

```
CONNECT BY PRIOR parent_id = id
```

Meaning:

```
child.parent_id = parent.id
```

So Oracle climbs **up the hierarchy**.

---

### Traversal Example

```
Market Research
   ↑
Concept & Market Definition
   ↑
Design and Development of Car
```

---

### Output

```
Design and Development of Car
Concept & Market Definition
Market Research
```

---

# Visual Example

Your hierarchy:

```
Design and Development of Car (indent 0)

   Concept & Market Definition (indent 1)

      Market Research (indent 2)
      Competitor Analysis (indent 2)

   Design & Aesthetics (indent 1)
```

If **Market Research ends in April**, query returns:

```
Design and Development of Car
Concept & Market Definition
Market Research
```

---

# Important Oracle Hierarchy Keywords

| Keyword    | Meaning                           |
| ---------- | --------------------------------- |
| START WITH | starting node                     |
| CONNECT BY | defines parent-child relationship |
| PRIOR      | specifies direction               |

---

# Bonus (Very Useful for Gantt Systems)

Oracle can also show **tree level automatically**.

```
LEVEL
```

Example:

```sql
SELECT
    LEVEL,
    title
FROM project_tasks
START WITH parent_id IS NULL
CONNECT BY PRIOR id = parent_id;
```

Output:

```
1  Design and Development of Car
2  Concept & Market Definition
3  Market Research
```

You can use this instead of storing **indent**.

---

# Which Method Should You Use?

| Method        | Best For               |
| ------------- | ---------------------- |
| Recursive CTE | Cross-database systems |
| CONNECT BY    | Oracle systems         |

For **Oracle 19c project planning / Gantt systems**, I strongly recommend:

✅ **CONNECT BY**

because it is **faster and simpler**.

---

💡 If you want, I can also show a **very advanced Oracle query used in real project management tools** that retrieves:

* tasks ending in month
* **all ancestors**
* **all children under those ancestors**

This is what **Jira / MS Project style Gantt charts actually require.**
