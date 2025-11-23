-- ---------------------------------------------------------
-- Library Project Management – Complete SQL Setup & Tasks
-- ---------------------------------------------------------

/* ===========================================
   1. DROP TABLES (if already exists)
=========================================== */
'''sql
DROP TABLE IF EXISTS return_status;
DROP TABLE IF EXISTS issued_status;
DROP TABLE IF EXISTS members;
DROP TABLE IF EXISTS books;
DROP TABLE IF EXISTS employees;
DROP TABLE IF EXISTS branch;
'''
/* ===========================================
   2. CREATE TABLES
=========================================== */

CREATE TABLE branch (
    branch_id       VARCHAR(50) PRIMARY KEY,
    manager_id      VARCHAR(50),
    branch_address  VARCHAR(50),
    contact_no      VARCHAR(50)
);

CREATE TABLE employees (
    emp_id     VARCHAR(30) PRIMARY KEY,
    emp_name   VARCHAR(30),
    position   VARCHAR(30),
    salary     INT,
    branch_id  VARCHAR(30)
);

CREATE TABLE books (
    isbn         VARCHAR(20) PRIMARY KEY,
    book_title   VARCHAR(60),
    category     VARCHAR(50),
    rental_price FLOAT,
    status       VARCHAR(50),
    author       VARCHAR(40),
    publisher    VARCHAR(50)
);

CREATE TABLE members (
    member_id       VARCHAR(50) PRIMARY KEY,
    member_name     VARCHAR(50),
    member_address  VARCHAR(50),
    reg_date        DATE
);

CREATE TABLE issued_status (
    issued_id         VARCHAR(20) PRIMARY KEY,
    issued_member_id  VARCHAR(20),
    issued_book_name  VARCHAR(75),
    issued_date       DATE,
    issued_book_isbn  VARCHAR(30),
    issued_emp_id     VARCHAR(10)
);

CREATE TABLE return_status (
    return_id        VARCHAR(50) PRIMARY KEY,
    issued_id        VARCHAR(40),
    return_book_name VARCHAR(80),
    return_date      DATE,
    return_book_isbn VARCHAR(50)
);

/* ===========================================
   3. FOREIGN KEYS
=========================================== */

ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY (issued_member_id) REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_isbn
FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_emp
FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branches
FOREIGN KEY (branch_id) REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued
FOREIGN KEY (issued_id) REFERENCES issued_status(issued_id);

/* ===========================================
   4. PROJECT TASK QUERIES
=========================================== */

-- Q1: Create a new book record
INSERT INTO books (isbn, book_title, category, rental_price, status, author, publisher)
VALUES ('978-1-7865-65-2', 'To kill a Mockingbird', 'Classic', 6.00, 'Yes', 'Marper Lee', 'J.B. Lippipcott & Co.');

-- Q2: Update member address
UPDATE members
SET member_address = '123 Main St.'
WHERE member_id = 'C101';

-- Q3: Delete record from issued_status
DELETE FROM issued_status WHERE issued_id = 'IS121';

-- Q4: Retrieve all books issued by a specific employee
SELECT * FROM issued_status WHERE issued_emp_id = 'E101';

-- Q5: List members who issued more than one book
SELECT issued_member_id, COUNT(*)
FROM issued_status
GROUP BY issued_member_id
HAVING COUNT(*) > 1;

/* ===========================================
   5. CTAS: Create Book Issue Summary Table
=========================================== */

CREATE TABLE book_counts AS
SELECT 
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) AS no_issued
FROM books b
JOIN issued_status ist ON b.isbn = ist.issued_book_isbn
GROUP BY b.isbn, b.book_title;

/* ===========================================
   6. Books by Category
=========================================== */

SELECT * FROM books WHERE category = 'Fiction';

/* ===========================================
   7. Total Rental Income by Category
=========================================== */

SELECT 
    b.category,
    SUM(b.rental_price) AS total_income,
    COUNT(*) AS total_books_issued
FROM books b
JOIN issued_status ist ON b.isbn = ist.issued_book_isbn
GROUP BY b.category;

/* ===========================================
   8. Members Registered in Last 1000 Days
=========================================== */

SELECT *
FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '1000 days';

/* ===========================================
   9. Employees with Manager Name & Branch
=========================================== */

SELECT 
    e1.*,
    b.manager_id,
    e2.emp_name AS manager_name
FROM employees e1
JOIN branch b ON e1.branch_id = b.branch_id
JOIN employees e2 ON b.manager_id = e2.emp_id;

/* ===========================================
   10. CTAS: Books > $7
=========================================== */

CREATE TABLE book_price_greater_7 AS
SELECT * FROM books
WHERE rental_price > 7;

/* ===========================================
   11. Books Not Returned Yet
=========================================== */

SELECT DISTINCT 
    ist.issued_book_name,
    ist.issued_id
FROM issued_status ist
LEFT JOIN return_status rs ON ist.issued_id = rs.issued_id
WHERE rs.return_id IS NULL;

/* ===========================================
   12. Overdue Books (>30 days)
=========================================== */

SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    CURRENT_DATE - ist.issued_date AS days_overdue
FROM issued_status ist
JOIN members m ON m.member_id = ist.issued_member_id
JOIN books bk ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status rs ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL
AND (CURRENT_DATE - ist.issued_date) > 30;

/* ===========================================
   13. Stored Procedure: Add Return Records
=========================================== */

CREATE OR REPLACE PROCEDURE add_return_records(
    p_return_id VARCHAR(50),
    p_issued_id VARCHAR(40)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_isbn VARCHAR(30);
    v_book_name VARCHAR(75);
BEGIN
    INSERT INTO return_status (return_id, issued_id, return_date)
    VALUES (p_return_id, p_issued_id, CURRENT_DATE);

    SELECT issued_book_isbn, issued_book_name
    INTO v_isbn, v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
END;
$$;

/* ===========================================
   14. Branch Performance Report
=========================================== */

CREATE TABLE branch_performance AS
SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) AS total_issued,
    COUNT(rs.return_id) AS total_returned,
    SUM(bk.rental_price) AS total_revenue
FROM issued_status ist
JOIN employees e ON e.emp_id = ist.issued_emp_id
JOIN branch b ON b.branch_id = e.branch_id
LEFT JOIN return_status rs ON rs.issued_id = ist.issued_id
JOIN books bk ON bk.isbn = ist.issued_book_isbn
GROUP BY b.branch_id, b.manager_id;

/* ===========================================
   15. CTAS: Active Members (last 6 months)
=========================================== */

CREATE TABLE active_members AS
SELECT *
FROM members
WHERE member_id IN (
    SELECT DISTINCT issued_member_id
    FROM issued_status
    WHERE issued_date >= CURRENT_DATE - INTERVAL '6 months'
);

/* ===========================================
   16. Stored Procedure: Issue Book
=========================================== */

CREATE OR REPLACE PROCEDURE issue_book(
    p_issued_id VARCHAR(20),
    p_issued_member_id VARCHAR(20),
    p_issued_book_isbn VARCHAR(75),
    p_issued_emp_id VARCHAR(10)
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_status VARCHAR(10);
BEGIN
    SELECT status INTO v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN
        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books
        SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        RAISE NOTICE 'Book issued successfully: %', p_issued_book_isbn;
    ELSE
        RAISE NOTICE 'Book not available: %', p_issued_book_isbn;
    END IF;
END;
$$;
