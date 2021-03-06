##Integrity Constraints in SQL

Most

#Structured Query Language (SQL) 

##Structured Query Language (SQL) 
is made up of two sub-languages

- SQL Data Manipulate Language (DML)
	- Select statements
		- perform queries
	- Insert, Update, Delete statements
		- modify the instance of a  table
- SQL Data Definition Language (DDL)
	- Create, Drop statements
- Modify the database schema
	- Grant, Revoke statements
		- enforce the security model
		- Data Control Language (DCL)

History of SQL Language

-developed at IBM (System R project)
-Standarized during 1980s

SQL 86 (ANSI, 1987 ISO)
basic relationally - complete language
bag semantics, NULLs, ordering, grouping/aggregation, outer joins

SQL-89 (minor revision)

SQL-92 (major revision)
syntactic extensions: CASE, nesting
new syntax for OUTER, JOIN

SQL 1999(major revision)
divides languages into CoreSQL + packages
CoreSQL - basically SQL-92 + recursion
some packages
object oriented
active databases (triggers, enhanced ICs)
stored procedures + external language bindings

SQL: 2003 (minor revision)
more package: XML, OLAP (windowing funcions)

...SQL:2006, SQL:2008, SQL: 2011

Note:

Indexes - not part of the standard
Vendor-specific physical structures
new physical structures without modifying the standard

##SQL DML: Queries

`select LastName, HireDate from Employee where Salary > 100000`

-operational semantics
filter out rows for which the condition is false
eliminate attributes not listed in the select clause
-result - A RELATION with
	-One attribute for each expression in the select clause
	-# rows <= cardinality of the specified relation

SQL is declarative (non-navigational)

##Multisets

relational model : relations are sets
SQL standard: tables are multisets (a.k.a bags)
duplicate tuples may be stored
SQL queries may result in duplicates even if none of this input tables contain duplicates
select distinct used to eliminate duplicates from a query result

`select distinct LastName, HireDate from Employee where  Salary > 100000`

##SQL Query Involving Several Relations
```
select P.ProjNo, E.LastName from Employee E, Project P
where P.RespEmp = E.EmpNo and P.DeptNo = 'E21'
```
For each project which department E21 is responsible, find the name of the employee in charge of that project

operational semantics
form cross product
filter rows
eliminate attributes
- result cardinality <= cardinality of cross product
- correlation names (table references) -> specify source of attributes 
	-can use relation names, if no relation appear twice in the query


##Observations: Queries, Instances, Schema

```
select P.ProjNo, E.LastName from Employee E, Project P
where P.RespEmp = E.EmpNo and P.DeptNo = 'E21'
```
-query -written for a particular schema, works on any instance
-schema- identifies what can be counted on
-How do I know that RespEmp values in Project will correspond to EmpNO values in Employee?
-Does the query have to worry about what to do if a Project have no responsible Employee

No, because of a foreign key constraint from RespEmp to EmpNo, and because RespEmp is defined as not NULL

##The SQL Basic Query Block

`select attribute-expression-list from relation-list where condition`

Note
The result of such a query is a relation which has one attribute for each element of the query’s attribute-expression-lst

attribute-expression-list can contain
relation attributes
expressions involving attributes, eg salary + bonus
built in functions

the SQL basic block represented in Relational Algebra: 𝜋 … (σ … (...✕ …))


##The SQL “Where” Clause

Conditions (i.e boolean valued expressions) may include
arithmetic operators: +, -, *, /
comparisons ==, <>, <, <=, >, >=
logical connectives: and , or , not
```
select E.LastName from Employee E, Department D. Employee M where E.WorkDept = D.DeptNo and D.MgrNo = M.EmpNO and E.Salary > M. salary
```
List of last names of employees who make more than their managers

Note:
Conjunctive queries: select-from-where (without or , not)

Examples:

List the identifiers of employees working on software support (E21) projects as of January 1, 2000.
```
select EmpNo 
from Emp_Act A, Project P 
where A.ProjNo = P.ProjNo 
and P.DeptNo = 'E21' 
and EmStDate <= '01/01/2000'
and EmEndDate <= '01/01/2000'
```
the syntax above works in some RDBMs (relational data base management systems), however in SQL-99 datetime literals look like DATE '2000/01/01'

Does this query return duplicates?
Yes if an employee has more than one connection to E21 projects in the Emp_Act table

###Question
List the identifiers and names of department managers who are also responsible for projects “owned” by departments other than their own
```
select E.EmpNo, E.LastName 
from Employee E, Department D, Project P 
where E.EmpNo = D.MgrNo
and E.EmpNo = P.RespEmp
and D.DeptNo <> P.DeptNo
```
Does the query return duplicates
Yes If an employee manages multiple departments, or if he manages one but is responsible for multiple projects

##The SQL “Select” Clause

Return the difference between each employee’s actual salary and a base salary of $40,000

select E.EmpNo, E.Salary - 4000 as SalaryDIff from Employee E

As above, but report zero if the actual salary is less than the base salary
```
select E.EmpNo, 
case 	when E.Salary < 40000 then 0
	else E.Salary - 40000 end
from Employee E
```
##Expressions

Expression operators include
The usual arithmetic and boolean operators
other operators eg CASE,  CAST, LIKE…
functions
numeric functions eg abs(), mod()
string functions e.g upper(), lower()
datetime functions, e.g current date()

expressions without explicit ’as’ clause are assigned default attribute names (integers)

##SQL DML: Insertion & Deletion
```
insert into Employee
values (‘0350’, ‘Sheldon’, ‘Q’, ‘Jetstream’, ‘A00’ , 01/10/2000, 25000.00);
```
Insert a single tuple into the Employee relation

delete from Employee; 

Delete all employees from the Employee table

```
delete from Employee 
where WorkDept = ‘A00’;
```
Delete all employees in department A00 from the Employee table

##Example

insert: statements can have an arbitrary query as input, but the schema of the result reaction must match the schema of the target reaction
```
insert into WellPaidEmps (EmpNo, Salary(
select EmpNo, Salary
from Employee
where Salary > 100000
```

##SQL DML: Update

```
update Employee set Salary = Salary * 1.05;
```
Increase the salary of every employee by five percent

```
update Employee set WorkDept = `E01` where WorkDept = `E21`;
```

Move all employees in department E21  into department E01

##Set Operations

UNION, INTERSECT, and EXCEPT (set difference operations)

```
select EmpNo from Employee
except
select MgrNo from Department
```

results are sets
- Q1 UNION Q2
	- includes any tuple found (at least one) in Q1 or in Q2
- Q1 INTERSECT Q2
	- incudes any tuple found (at least once) in both Q1 and Q2
- Q1 EXCEPT Q2
	- includes any tuple found (at least once) in Q1 and not found in Q2

###Examples

List the identifiers of employyes who are managers or who are responsible for some project

```
select MgrNo from Department 

union 

select RespEmp from Project
```

List the employees who are not involved in any project activites as of Jan, 1, 1920

```
select EmpNo, LastName from Employee
except
select A.EmpNo, E.LastName from Emp_Act A, Employee E 
where A.EmpNo = E.EmpNo

and '1/1/1920' betweem A.EmStDate and A.EmEnDate
```
...
##Integrity Constraints in SQL

Most commonly used- SQL schema constraints

- Not NUll
- Primary Key
- Unique
- Foreign Key
- Column or Tuple Check

Recent SQL standards also allow more powerful integrity constraints. However they are not supported by all commecial DBMSs

**Constraints over a Single Table**

We can specify complex constraints over a single table using table constraints, which ahve a form CHECK condition expression
```
CREATE TABLE Sailors ( 	sid INTEGER,
			sname CHAR(10),
			rating INTEGER,
			age REAL,
			PRIMARY KEY (sid),
			CHECK (rating >= 1 AND rating <= 10 ))
```
**Domain Constraints and Distinct Types**
A user can defined a new domain using the CREATE DOMAIN statment which uses a CHECK constraint
```
CREATE DOMAIN ratingval INTEGER DEFAULT 1
CHECK ( VALUE >= 1 AND VALUE <= 10 )
```

Once a domain is defined , the name of the domain can be used to restict column valuse in a table.

We can now use this in a schema declaration
```
rating ratingval
```

**Declaring a type:** Values of type ratingtype can be compared with each other, but they cannot be compared with values of other types
If we want to define operations on the new type, for example, an average function, we must do so explicitly; none of the existing operations on the source type carryover.
```
CREATE TYPE ratingtype AS INTEGER
```

**Assertions: Which are Constraints over Several Tables**


##Triggers

A trigger is a procedure executed by the database in response to the change to the database instance


Basic compoents of a trigger description

**Event**: Type of change that should cause trigger to fire

**Condition**: Test performed by trigger to determine whether further action is needed

**Action** Procedure executed if condition is met

Trigger only happens on some event, in the case of integrity constraint will be enforced for all changes in the table, triggers only specific cases.

Note use the integrity constraint used in the textbook
