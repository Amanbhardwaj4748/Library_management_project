# Library_management_project
This project is all about the library management procedure. I have solved the queries by using SQL. I have imported the csv files into Postgre SQL database and solved the queries.
## --Library Project Management--
--Creating Tables--
'''sql
DROP TABLE IF EXISTS branch;
CREATE TABLE branch (
	branch_id VARCHAR(50) PRIMARY KEY,
	manager_id VARCHAR(50),
	branch_address VARCHAR(50),
	contact_no VARCHAR(50)
);

DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
	emp_id VARCHAR(30) PRIMARY KEY,
	emp_name VARCHAR(30),
	position VARCHAR(30), 
	salary INT,
	branch_id VARCHAR(30) --FK
);

DROP TABLE IF EXISTS books;
CREATE TABLE books (
	isbn VARCHAR(20) PRIMARY KEY,
	book_title VARCHAR(60),
	category VARCHAR(50),
	rental_price FLOAT,
	status VARCHAR(50),
	author VARCHAR(40),
	publisher VARCHAR(50)
);

DROP TABLE IF EXISTS members;
CREATE TABLE members (
	member_id VARCHAR(50) PRIMARY KEY,
	member_name VARCHAR(50),
	member_address varchar(50),
	reg_date DATE
);

DROP TABLE IF EXISTS issued_status; 
CREATE TABLE issued_status (
	issued_id VARCHAR(20) PRIMARY KEY,
	issued_member_id VARCHAR(20), --FK
	issued_book_name VARCHAR(75), 
	issued_date DATE,
	issued_book_isbn VARCHAR(30), --FK
	issued_emp_id VARCHAR(10)  --FK
);

DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status (
return_id VARCHAR(50) PRIMARY KEY,
issued_id VARCHAR(40),
return_book_name VARCHAR(80),
return_date DATE,
return_book_isbn VARCHAR(50)
);
'''
## **Adding the foreign Keys ##
--FOREIGN KEY
'''sql
ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY (issued_member_id)
REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_isbn
FOREIGN KEY (issued_book_isbn)
REFERENCES books(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_emp
FOREIGN KEY (issued_emp_id)
REFERENCES employees(emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branches
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id);

'''

##--**Project Task
-- **Q.1 Create a new book record -- "978-1-7865-65-2", "To kill a Mockingbird", "Classic",6.00,'Yes', 'Marper Lee', 'J.B. Lippipcott & Co.'
	''' sql
  INSERT INTO books (isbn, book_title, category, rental_price, status, author, publisher)
	VALUES
	('978-1-7865-65-2', 'To kill a Mockingbird', 'Classic',6.00,'Yes', 'Marper Lee', 'J.B. Lippipcott & Co.');
'''
##--**Q.2 Update an existing member's addressw
'''sql
UPDATE members
SET member_address = '123 Main st.'
WHERE member_id ='C101';
'''
##--**Q.3 Delete a record from the Issued Status Table 
--Delete from issed_status table with issued_id ='IS104' from the issued_status_table
'''sql
SELECT * from issued_status;

DELETE FROM issued_status
WHERE issued_id='IS121';
'''
##--**Q.4 Retrieve all books issued by a specific employee
'''sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
'''

##--**Q.5  List members who have issued more than one book 
'''sql
SELECT issued_emp_id,
COUNT(issued_id)
FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(issued_id)>1;
'''

##--**CTAS
--**Q.6 Create a summary table: Used CTAS to generate new tables based on query results -each book and total book issued
'''sql
CREATE TABLE book_counts
AS
select 
	b.isbn,
	b.book_title,
	COUNT(ist.issued_id) as no_issued
from books as b
JOIN
issued_status as ist
ON ist.issued_book_isbn=b.isbn
GROUP BY 1,2;
'''
##/***Q.7 Retrieve all books in a specific category */
'''sql
select category,
count(isbn)
from books
GROUP BY 1;

select * from books
where category='Fiction';
'''
##/***Q.8 Find the total rental income by category */
'''sql
select category,
SUM(rental_price) as total_income
from books
GROUP BY 1;

select 
b.category,
SUM(b.rental_price) as total_income,
count(*)
from 
books as b
JOIN 
issued_status as ist
ON b.isbn=ist.issued_book_isbn
GROUP BY 1;
'''
##--**Q.9 list members who registered in last 1000 days--
'''sql
select * from members
WHERE reg_date>= CURRENT_DATE-INTERVAL '180 days';

INSERT INTO members(member_id,member_name,member_address,reg_date)
VALUES
('C125','Ram','123 Main Street','2025-10-31'),
('C126','Sam','124 Main Street','2025-10-31'),
('C127','Tam','125 Main Street','2025-10-31');
'''
##--**Q.10 list the employees with their branch manager name and their branch details--
'''sql
select 
	e1.*,
	b.manager_id,
	e2.emp_name as manager_name
from 
employees as e1
JOIN
branch as b
ON e1.branch_id=b.branch_id
JOIN 
employees as e2
ON b.manager_id=e2.emp_id;
'''
##/***Q.11 Create a table of books with rental price above a certain threshold 7USD--*/
''' sql
CREATE TABLE book_price_greater_7
AS
SELECT * FROM books
where rental_price>7;

select * from book_price_greater_7;
'''
##/***Q.12 retrieve the list of books that are not being returned */
'''sql
select 
	DISTINCT ist.issued_book_name,
	ist.issued_id
from issued_status as ist
LEFT JOIN
return_status as rs
ON ist.issued_id=rs.issued_id
WHERE rs.return_id IS NULL;
'''
##/***Q.13 Identify members with overdue books
write a query to identify members who have overdue books (assume a 30-day reurn period). 
Display the member_id, member's name, book_title, issue_date and days overdue.*/

--issue_status==members==books==return_status
'''sql
select 
	ist.issued_member_id,
	m.member_name,
	bk.book_title,
	ist.issued_date, 
	rs.return_date,
	CURRENT_DATE - ist.issued_date as over_due_days
from issued_Status as ist
JOIN
members as m
	ON m.member_id=ist.issued_member_id
JOIN
books as bk
	ON bk.isbn=ist.issued_book_isbn
LEFT JOIN 
return_status as rs
	ON rs.issued_id=ist.issued_id
WHERE rs.return_date IS NULL
	AND
	  (CURRENT_DATE - ist.issued_date)>30
ORDER BY 1;
'''
##/***Q.14 Update the book status on return
Write a query to update the status of books table as "Yes" when they are returned
(based on entries in the return_status table). */
'''sql
select * from issued_status;

select * from books
where isbn='978-0-14-118776-1';

UPDATE books
SET status='no'
WHERE isbn='978-0-14-118776-1';
select * from books;
--
INSERT INTO return_status(return_id, issued_id, return_date)
VALUES
('Rs125','IS130',CURRENT_DATE);
select * from return_status;
--Store Procedure--
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(50),p_issued_id VARCHAR(40))
LANGUAGE plpgsql
AS $$

DECLARE
		v_isbn VARCHAR(30);
		v_issued_book_name VARCHAR(75);
BEGIN
	--all your logic and code--
	INSERT INTO return_status(return_id, issued_id, return_date)
	VALUES
	('p_return_id','p_issued_id',CURRENT_DATE);

	SELECT 
		issued_book_isbn,
		issued_book_name
		INTO
		v_isbn,
		v_issued_book_name
	FROM issued_status
	WHERE issued_id=p_issued_id;
	
	UPDATE books
	SET status='yes'
	WHERE isbn='v_isbn';

	RAISE NOTICE 'Thankyou for returning the book: %', v_book_name;
END;
$$
SELECT * FROM issued_Status;
CALL add_return_records('RS138','IS135');
'''
##/***Q.15 Branch Performance Report
Create a query that generates a performance report for each branch showing the number of books issued, the number of 
books returned, and the total revenue generated from the book rentals.*/
'''sql
select * from branch;
select * from issued_status;
select * from employees;
select * from books;
CREATE TABLE branch_performance
AS
select b.branch_id,
	   b.manager_id,
	   COUNT(ist.issued_id) as no_book_issued,
	   COUNT(rs.return_id) as no_of_book_return,
	   SUM(bk.rental_price) as total_revenue
from issued_status as ist
JOIN
employees as e 
ON e.emp_id=ist.issued_emp_id
JOIN branch as b
ON e.branch_id=b.branch_id
LEFT JOIN
return_status as rs
ON rs.issued_id=ist.issued_id
JOIN 
books as bk
ON ist.issued_book_isbn=bk.isbn
GROUP BY 1,2;

SELECT * from branch_performance;
'''
##/***Q.16 CTAS: Create table of active members
Use the CTAS statement to create a new table active_members containing members who have issued atleast one book in last 6 months*/
'''sql
CREATE TABLE active_member
AS
SELECT * FROM members
 WHERE member_id IN (SELECT 
					 	DISTINCT issued_member_id
				  	 FROM issued_status
				   	 WHERE 
					   issued_date>=CURRENT_DATE-INTERVAL '19 month'
				  	 );
 
SELECT * FROM active_member;
'''
##/*** Q.17 Find employees with the most book issues processed
write a query to find the top 3 employees who have processed the most book issues. Display the employee name,
number of book processed, and their branch.*/
'''sql
SELECT 
	e.emp_name,
	b.*,
	COUNT(ist.issued_id) as no_book_issued
FROM issued_status as ist
JOIN
employees as e
ON e.emp_id=ist.issued_emp_id
JOIN
branch as b
ON b.branch_id=e.branch_id
GROUP BY 1,2;
'''
##/***Q.18 Stored procedure objective:
Create a stored procedure to manage the status of books in a library system.
Discription: Write a stored procedure that updates the status of a book in the library based on
its issuance*/ 
'''sql
select * from issued_status;
CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(20), p_issued_member_id VARCHAR(20), p_issued_book_isbn VARCHAR(75),p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
		v_status VARCHAR(10);
BEGIN
	--checking if book is available 'yes'
	SELECT 
	status
	INTO
	v_status
	FROM books
	WHERE isbn =p_issued_book_isbn;

	IF V_status = 'yes' THEN
	INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
	VALUES
	(p_issued_id,p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

	UPDATE books
	SET status='no'
	WHERE isbn=p_issued_book_isbn;
	
	RAISE NOTICE 'Book record added successfully for book isbn : %', p_issued_book_isbn;


	ELSE
		RAISE NOTICE 'Sorry this book is unavailable book_isbn: %', p_issued_book_isbn; 
	END IF;
END;
$$

call issue_book('IS155','C108','978-0-553-29698-2','E104');
call issue_book('IS156','C109','978-0-553-29698-2','E104');
DROP PROCEDURE issue_book(p_issued_id VARCHAR(20), p_issued_member_id VARCHAR(20), p_issued_book_isbn VARCHAR(75),p_issued_emp_id VARCHAR(10));
'''
