# 📚 Library Management System using SQL Project — P2  

## 📌 Project Overview  

The **Library Management System** project is designed to demonstrate how SQL can be used to build and manage a relational database for real-world applications.  
It covers essential concepts like table creation, CRUD operations, CTAS (Create Table As Select), and analytical queries.  
Through this project, I practiced database design, data manipulation, and query optimization in a structured way.  

---

## 🎯 Objectives  

1. **Database Setup**: Create and populate tables for branches, employees, members, books, issued status, and return status.  
2. **CRUD Operations**: Perform insert, update, delete, and select operations.  
3. **CTAS (Create Table As Select)**: Create summary tables for quick analysis.  
4. **Data Analysis**: Write SQL queries to extract meaningful insights.  

---
## Project Structure

### 1. 🗄 Database Setup  

- **Database Name**: `library_management_db`  
- **Tables**: Branch, Employees, Members, Books, Issued Status, Return Status  
- **Design**: Includes primary keys, foreign keys, and relational integrity  

```sql
CREATE DATABASE library_db;

-- Branch Table
CREATE TABLE branch(
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

-- Employees Table
CREATE TABLE employees(
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10)
);

-- Members Table
CREATE TABLE members(
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

-- Books Table
CREATE TABLE books(
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(80),
    category VARCHAR(30),
    rental_price DECIMAL(10,2),
    status VARCHAR(10),
    author VARCHAR(30),
    publisher VARCHAR(30)
);

-- Issued Status Table
CREATE TABLE issued_status(
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member_id VARCHAR(30),
    issued_book_name VARCHAR(80),
    issued_date DATE,
    issued_book_isbn VARCHAR(50),
    issued_emp_id VARCHAR(10)
);

-- Return Status Table
CREATE TABLE return_status(
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50)
);

-- Foreign Keys
ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY (issued_member_id)
REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY (issued_book_isbn)
REFERENCES books(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees
FOREIGN KEY (issued_emp_id)
REFERENCES employees(emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id);

-- Schema Updates
ALTER TABLE books
ALTER COLUMN category TYPE VARCHAR(20);

ALTER TABLE branch
ALTER COLUMN contact_no TYPE VARCHAR(20);
```
### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

**Task 2: Update an Existing Member's Address**
```sql
UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';
```
**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
```sql
DELETE FROM issued_status
WHERE issued_id = 'IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
```

**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.
```sql
SELECT 
    issued_emp_id,
    COUNT(issued_id) AS total_book_issued
FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(issued_id) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_cnts AS
SELECT
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) AS no_issued
FROM books AS b
JOIN issued_status AS ist
    ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;

-- View Results
SELECT * FROM book_cnts;
```

### 4. Data Analysis & Findings
The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:
```sql
SELECT * FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:
```sql
SELECT 
    b.category,
    SUM(b.rental_price) AS total_rental_price,
    COUNT(*) AS total_books_issued
FROM books AS b
JOIN issued_status AS ist
    ON ist.issued_book_isbn = b.isbn
GROUP BY b.category;
```

9. **Task 9: List Members Who Registered in the Last 180 Days**:
```sql
INSERT INTO members (member_id, member_name, member_address, reg_date)
VALUES
    ('C120', 'Jack', '145 Maple St', '2025-06-01'),
    ('C121', 'Michael', '154 Maple St', '2025-05-01');

SELECT * FROM members
WHERE reg_date > CURRENT_DATE - INTERVAL '180 days';
```

10. **Task 10: List Employees with Their Branch Manager's Name and their branch details**:
```sql
SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.branch_id,
    b.branch_address,
    b.contact_no,
    e2.emp_name AS manager
FROM employees AS e1
JOIN branch AS b
    ON e1.branch_id = b.branch_id    
JOIN employees AS e2
    ON e2.emp_id = b.manager_id;
```
Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold(7.00)**:
```sql
CREATE TABLE expensive_books AS
SELECT 
    isbn,
    book_title,
    category,
    rental_price
FROM books
WHERE rental_price > 7.00;
```
Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT 
    ist.issued_id,
    ist.issued_book_isbn,
    ist.issued_emp_id,
    ist.issued_member_id,
    ist.issued_date
FROM issued_status AS ist
LEFT JOIN return_status AS rs
    ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```
