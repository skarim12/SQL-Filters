# Applying Filters and Joins to SQL Queries: Security Investigation Lab

Using SQL filters and joins to investigate a suspicious login incident, identify employee machines for security updates, and connect data across tables.

## Overview

Databases store large volumes of data across tables, and SQL (Structured Query Language) is how analysts search them efficiently. In this project, I used SQL filters and joins to query a sample organization's database for **login activity**, **employee information**, and **device assignments**, mirroring tasks a SOC analyst performs during incident triage and security patching.

**Skills demonstrated:** `SELECT` · `FROM` · `WHERE` · `AND` · `OR` · `NOT` · `LIKE` · `%` wildcard · `INNER JOIN` · `LEFT JOIN` · `RIGHT JOIN` · `ON`

## Database Tables

| Table | Purpose |
|---|---|
| `log_in_attempts` | Login date, time, country, IP address, username, and whether each attempt succeeded (`success = 1`) or failed (`success = 0`) |
| `employees` | Employee department, office location, username, and assigned `device_id` |
| `machines` | Company devices, identified by `device_id` |

---

## Part 1: Login Investigation

### 1. After-hours failed login attempts

**Scenario:** My team detected a security incident after business hours that could be a brute force attempt. I filtered for failed logins after 18:00.

```sql
SELECT *
FROM log_in_attempts
WHERE login_time > '18:00' AND success = 0;
```

**How it works:**
- `SELECT *` returns every column.
- `FROM log_in_attempts` sources the login log table.
- `WHERE login_time > '18:00'` isolates after-hours activity.
- `AND success = 0` adds a second condition so only failed attempts are returned.


---

### 2. Login attempts on specific dates

**Scenario:** A suspicious event occurred on 2022-05-09. To investigate, I reviewed login activity for that day and the day before.

```sql
SELECT *
FROM log_in_attempts
WHERE login_date = '2022-05-09' OR login_date = '2022-05-08';
```

**How it works:**
- `OR` returns rows matching *either* date.
- Results include both successful and failed attempts, giving full context around the event.


---

### 3. Login attempts outside of Mexico

**Scenario:** I searched for suspicious activity originating outside of Mexico.

```sql
SELECT *
FROM log_in_attempts
WHERE NOT country LIKE 'MEX%';
```

**How it works:**
- `LIKE 'MEX%'` matches any country value that *starts with* "MEX." The `%` wildcard accounts for different spellings, such as `MEX` or `Mexico`.
- `NOT` negates the match, returning every country *except* Mexico.


---

## Part 2: Employee Lookups for Security Updates

### 4. Marketing employees in the East building

**Scenario:** The team needed to update machines for Marketing employees in the East building.

```sql
SELECT *
FROM employees
WHERE department = 'Marketing' AND office LIKE 'EAST%';
```

**How it works:**
- `department = 'Marketing'` is an exact match.
- `AND` requires the second condition to also be true.
- `office LIKE 'EAST%'` matches any office beginning with "EAST," regardless of room number.


---

### 5. Employees in Finance or Sales

**Scenario:** The security team needed updates on employees in the Finance and Sales departments.

```sql
SELECT *
FROM employees
WHERE department = 'Finance' OR department = 'Sales';
```

**How it works:**
- `OR` returns employees who match either department.


---

### 6. All employees not in IT

**Scenario:** IT had already received the update, so every other department still needed it.

```sql
SELECT *
FROM employees
WHERE NOT department = 'Information Technology';
```

**How it works:**
- `NOT` negates the condition, returning every department except Information Technology.


---

## Part 3: Joining Tables

Joins combine rows from two tables using a shared column. Here, `device_id` links `machines` to `employees`, and `username` links `employees` to `log_in_attempts`.

### 7. Match employees to their machines (INNER JOIN)

**Scenario:** The department needed machines matched to the employees who use them.

```sql
SELECT *
FROM machines
INNER JOIN employees ON machines.device_id = employees.device_id;
```

**How it works:**
- `INNER JOIN` returns only rows with a match in **both** tables.
- `ON machines.device_id = employees.device_id` defines the shared key.
- The result is a combined table of each employee and their assigned machine.

---

### 8. Find every machine, including unassigned ones (LEFT JOIN)

**Scenario:** The organization needed to identify devices sitting unassigned in the closet.

```sql
SELECT *
FROM machines
LEFT JOIN employees ON machines.device_id = employees.device_id;
```

**How it works:**
- `LEFT JOIN` keeps **every row from the first table** (`machines`), even without a match.
- Machines with no assigned user return `NULL` in the employee columns, which flags them as unassigned.


---

### 9. Find every employee, including those without a laptop (RIGHT JOIN)

**Scenario:** The organization needed to see all employees, even those who had not been issued a laptop.

```sql
SELECT *
FROM machines
RIGHT JOIN employees ON machines.device_id = employees.device_id;
```

**How it works:**
- `RIGHT JOIN` keeps **every row from the second table** (`employees`), even without a match.
- Employees with no device return `NULL` in the machine columns.


---

### 10. Connect employees to their login attempts (INNER JOIN)

**Scenario:** To continue the incident investigation, I needed login activity tied to specific employees.

```sql
SELECT *
FROM employees
INNER JOIN log_in_attempts ON employees.username = log_in_attempts.username;
```

**How it works:**
- `ON employees.username = log_in_attempts.username` links each login attempt to an employee.
- The result shows each user's IP address, login time, and whether the attempt succeeded, which is the context needed to trace suspicious activity back to an account.


---

## Summary

This project was a hands-on introduction to SQL filtering and joins in a security context.

| Operator | Use in this lab |
|---|---|
| `AND` | Require multiple conditions (failed login **and** after hours) |
| `OR` | Match any of several values (two dates, two departments) |
| `NOT` | Exclude values (outside Mexico, not in IT) |
| `LIKE` + `%` | Match patterns (`MEX%`, `EAST%`) |
| `INNER JOIN` | Keep only rows that match in both tables |
| `LEFT JOIN` | Keep all rows from the first table (find unassigned machines) |
| `RIGHT JOIN` | Keep all rows from the second table (find employees without devices) |

Filters narrow large datasets to exactly the records needed, and joins connect related data across tables. Together they make investigations faster and more accurate than searching raw logs.

## Connect

- LinkedIn: [linkedin.com/in/karimsaminur123](https://linkedin.com/in/karimsaminur123)
- Medium: [medium.com/@karimsaminur123](https://medium.com/@karimsaminur123)
