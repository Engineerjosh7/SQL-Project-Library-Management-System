# 📚 SQL Project: Library Management System

![SQL Badge](https://img.shields.io/badge/SQL-Analysis-blue)
![Project Status](https://img.shields.io/badge/status-completed-brightgreen)

## Project Overview

**Project Title**: Library Management System

**Database**: `library_db`

A complete database-driven system to manage library operations, designed with MySQL. It covers database creation, table design, CRUD actions, advanced SQL queries, and procedural logic - all housed within a single project.


![library image](https://github.com/user-attachments/assets/38755a7a-d101-4dfd-ada4-4811693b087d)




## Objectives

A complete relational database project built with MySQL that manages operations in a library—including book tracking, overdue management, employee performance, member insights, and structured analytics. Designed by **Engineerjosh7**, this system leverages stored procedures, CTAS queries, and advanced SQL joins to turn library data into actionable insights.data.

## Project Structure

### 1. Database Setup
![library EER Diagram](https://github.com/user-attachments/assets/ae71abb8-f8a9-4e4c-8494-eca59c99ba35)



- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
Create database library_project;

-- create branch table
Drop table if exists branch;
create table branch
(
	branch_id varchar (10) primary key,
	manager_id varchar(10),
	branch_address varchar(30),
	contact_no varchar (15)
);


-- create employees table
Drop table if exists employees;
create table employees
(
	emp_id varchar (10) primary key,
	emp_name varchar(30),
	position varchar(20),
	salary decimal(10,2),
	branch_id varchar(10),
	foreign key (branch_id) references branch(branch_id)
);


-- create members table
Drop table if exists members;
create table members
(
	member_id varchar (10) primary key,
	member_name varchar(30),
	member_address varchar(30),
	reg_date date
);


-- create books table
Drop table if exists books;
create table books
(
            isbn varchar (50) primary key,
            book_title varchar(80),
            category varchar(30),
            rental_price decimal(10,2),
            status varchar(10),
            author varchar(40),
            publisher varchar(40)
);


-- create issued_status table
Drop table if exists issued_status;
create table issued_status
(
            issued_id varchar (10) primary key,
            issued_member_id varchar(30),
            issued_book_name varchar(30),
            issued_date date,
            issued_book_isbn varchar(50),
            issued_emp_id varchar(10),
            foreign key (issued_emp_id) references employees(emp_id),
            foreign key (issued_member_id) references members(member_id),
            foreign key (issued_book_isbn) references books(isbn)
);


-- create return_status table
Drop table if exists return_status;
create table return_status
(
            return_id varchar (10) primary key,
            issued_id varchar(30),
            return_book_name varchar(30),
            return_date date,
            return_book_isbn varchar(50),
            foreign key (issued_id) references issued_status(issued_id)
);
```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "('978-0-316-76948-0', 'The Fault in Our Stars', 'Young Adult', 6.50, 'yes', 'John Green', 'Dutton Books')"

```sql
insert into books
values ('978-0-316-76948-0', 'The Fault in Our Stars', 'Young Adult', 6.50, 'yes', 'John Green', 'Dutton Books');
select * from books;
```
**Task 2: Update an Existing Member's Address**

```sql
update members
set member_address = '108 Hero St'
where member_id = 'C103';
select * from members;
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
delete from issued_status
where issued_id = 'IS121';
select * from issued_status;
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
select * from issued_status
where issued_emp_id = 'E101';
```

**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
select
count(*) as books_issued,
issued_emp_id from issued_status
group by issued_emp_id
having count(*) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_issued_cnt AS
SELECT
   b.isbn,
   b.book_title,
   COUNT(ist.issued_id) AS issue_count
FROM issued_status as ist
JOIN books as b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;

select * from book_issued_cnt;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
select * from books
where category = 'Dystopian';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
select
   sum(bb.rental_price) Total_price,
   bb.category,
   count(*) as Total_books
from books as bb
join issued_status as istn
on bb.isbn = istn.issued_book_isbn
group by bb.category;
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
select * from members
where reg_date >= current_date() - interval 180 day;
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
select
   e1.emp_id,
   e1.emp_name,
   e1.position,
   e1.salary,
   e2.emp_id as manager_id,
   b1.branch_id,
   b1.branch_address,
   b1.contact_no,
   e2.emp_name as manager
from employees as e1 
join branch as b1
on e1.branch_id = b1.branch_id
join employees as e2
on e2.emp_id = b1.manager_id;
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
create table expensive_book as
select * from books
where rental_price > 6.0;

select * from expensive_book;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
select * from issued_status as iss
left join return_status as re
on iss.issued_id = re.issued_id
where re.issued_id is null;
```


## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
select 
   me.member_id,
   me.member_name,
   bo.book_title,
   iss.issued_date,
   datediff(current_date(), iss.issued_date) as days_overdue 
from
members as me
join issued_status as iss on me.member_id = iss.issued_member_id
join books as bo on iss.issued_book_isbn = bo.isbn
left join return_status as re on iss.issued_id = re.return_id
where re.return_date is null
and datediff(current_date(), iss.issued_date) > 30
order by me.member_id;
```

**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).

```sql
USE `library_project`;
DROP procedure IF EXISTS `add_return_records`;

USE `library_project`;
DROP procedure IF EXISTS `library_project`.`add_return_records`;
;

DELIMITER $$
USE `library_project`$$
CREATE DEFINER=`root`@`localhost` PROCEDURE `add_return_records`(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(10))
BEGIN
Declare v_isbn varchar (60);
Declare v_book_name varchar (80);

insert into return_status(return_id, issued_id, return_date, book_quality)
values
(p_return_id, P_issued_id, current_date(), p_book_quality);

select issued_book_isbn,
issued_book_name into
v_isbn,
v_book_name
from issued_status
where issued_id = p_issued_id;

update books
set status = 'yes'
where isbn = v_isbn;

select concat('Thank you for returning the book: ', v_book_name) as message;
END$$

DELIMITER ;
;

-- calling function 
CALL add_return_records('RS138', 'IS135', 'Good');

-- calling function 
CALL add_return_records('RS148', 'IS140', 'Good');

```

**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
select
   b.branch_id,
   count(distinct iss.issued_id) as number_of_books_issued,
   sum(bb.rental_price) as Total_rental,
   count(distinct re.return_id) as number_of_books_returned 
from
issued_status as iss
left join employees as em on iss.issued_emp_id = em.emp_id
left join return_status as re on iss.issued_id = re.issued_id
join books as bb on iss.issued_book_isbn = bb.isbn
join branch as b on em.branch_id = b.branch_id
group by b.branch_id;

SELECT * FROM branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql
create table active_members as
select
   me.member_id,
   me.member_name,
   count(iss.issued_id) as collect_count
from
   members as me join issued_status as iss on me.member_id = iss.issued_member_id
where
   iss.issued_date >= current_date() - interval 2 month
group by
   me.member_id, me.member_name
order by collect_count desc;

SELECT * FROM active_members;
```

**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee's name, number of books processed, and their branch.

```sql
select
   em.emp_name,
   count(distinct iss.issued_id) as books_processed,
   em.branch_id,
    b.*
from
	employees as em
join issued_status as iss on em.emp_id = iss.issued_emp_id
join branch as b on b.branch_id = em.branch_id
group by em.emp_id, em.branch_id
order by books_processed desc
limit 3;
```

**Task 18: Find Employees with the Most Book Issues Processed (monthly)**

```sql
SELECT
   em.emp_name,
   em.branch_id,
   DATE_FORMAT(iss.issued_date, '%Y-%m') AS issue_month,
   COUNT(iss.issued_id) AS books_processed
FROM 
   employees AS em
JOIN issued_status AS iss ON em.emp_id = iss.issued_emp_id
GROUP BY 
   em.emp_name, em.branch_id, issue_month
ORDER BY 
   issue_month DESC, books_processed DESC
LIMIT 3;
```

**Task 19: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.

```sql
select
   me.member_name,
   b.book_title,
   count(iss.issued_id)
from
   books as b join issued_status as iss on b.isbn = iss.issued_book_isbn
join members as me on me.member_id = iss.issued_member_id
where b.status = 'damaged'
group by me.member_id, me.member_name, b.book_title
having count(iss.issued_id) > 2;
```

**Task 20: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql
USE `library_project`;
DROP procedure IF EXISTS `issue_book`;

USE `library_project`;
DROP procedure IF EXISTS `library_project`.`issue_book`;
;

DELIMITER $$
USE `library_project`$$
CREATE DEFINER=`root`@`localhost` PROCEDURE `issue_book`(p_issued_id varchar(10),
p_member_id varchar (30),
p_issued_isbn varchar (50),
p_emp_id varchar (10))
BEGIN
declare v_status varchar(10);

-- Step 1: Check book status
select status into v_status
from books
where isbn = p_issued_isbn;

-- Step 2: Conditional logic
if status = 'yes' then
insert into issued_status(issued_id,
member_id,
issued_date,
issued_isbn,
emp_id)
values
(p_issued_id, p_member_id, current_date(),
p_issued_isbn, p_emp_id);

-- Mark book as not available
update books
set status = 'no'
where isbn = p_issued_isbn;

-- Confirmation message
select concat('Book issued successfully. ISBN: ', p_issued_isbn) as message;

else
   -- Book is unavailable
   SIGNAL SQLSTATE '45000'
   SET MESSAGE_TEXT = 'This book is currently unavailable for issue.';
   END IF;
        
END$$

DELIMITER ;
;

-- Testing The function
SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;

CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'
```

**Task 21: Create Table As Select (CTAS)**
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
    The number of overdue books.
    The total fines, with each day's fine calculated at $0.50.
    The number of books issued by each member.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines

```sql
create table overdues
as
select
   iss.issued_member_id,
   me.member_name,
   datediff(current_date, iss.issued_date) as day_overdue,
   sum((datediff(current_date, iss.issued_date) - 30) * 0.50) as ftotal_fine,
   count(*) as overdue_books,
   (SELECT COUNT(*) FROM issued_status AS ist 
   WHERE ist.issued_member_id = iss.issued_member_id) AS total_books_issued
from
issued_status as iss
left join books as b on iss.issued_book_isbn = b.isbn
left join return_status as re on iss.issued_id = re.issued_id
join members as me on me.member_id = iss.issued_member_id
where re.return_date is null
and datediff(current_date, iss.issued_date) > 30
group by iss.issued_member_id, b.book_title, b.isbn, iss.issued_date;
```


## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.
```sh
git clone https://github.com/Engineerjosh7/SQL-Project-Library-Management-System.git
```

2. **Set Up the Database**: Execute the SQL scripts in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## Author - Engineerjosh7

This project showcases SQL skills essential for database management and analysis. Connect with me through the following channels:

- **Instagram**: [Outside SQL](https://www.instagram.com/Ogunsola901/)
- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/joshua-ogunsola)
- **Email**: [Email me](joshusogunsola7@gmail.com)

I appreciate your interest in this project!
