# SQL
I got full marks on all the answers I have included in this file.

## Bank Schema

The schema for the bank database is as follows. Primary key attributes are underlined and foreign keys are noted in superscript.

- *Customer* = {*customerID*, firstName, lastName, income, birthDate }
- *Account* = {*accNumber*, type, balance, branchNumber^Branch}
- *Owns* = {*customerID^Customer*, *accNumber^Account*}
- *Transactions* = {*transNumber*, *accNumber^Account*, amount}
- *Employee* = {*sin*, firstName, lastName, salary, branchNumber^Branch}
- *Branch* = {*branchNumber*, branchName, managerSIN^Employee, budget}

#### Notes

- The *customerID* attribute (*Customer*) is a unique number that represents a customer, it is *not* a customer's SIN
- The *accNumber* attribute (*Account*) represents the account number
- The *balance* (*Account*) attribute represents the total amount in an account, and should equal the sum of the account's transaction amounts
- The *type* (*Account*) attribute represents the type of an account: chequing (CHQ), saving (SAV), or business (BUS)
- The *Owns* relation represents a many-to-many relationship between *Customer* and *Account*
- The *transNumber* attribute (*Transactions*) represents a transaction number, combined with account number it uniquely identifies a transaction
- The *branchNumber* attribute (*Branch*) uniquely identifies a branch
- The *managerSIN* attribute (*Branch*) represents the SIN of the branch manager

## Questions

Write *SQL* queries to return the data specified in questions 1 to 20.

#### Query Requirements

- Rows must be ordered as shown in the question (noted in green), order is ascending unless specified otherwise
- Columns must be printed in the order shown in the question (from left to right)
- Query results should be sent to text (not a grid)
- Every column in the result should be named, so if the query asks you to return something like *income minus salary* make sure that you include an *AS* statement to name the column
- While your queries will *not* be assessed on their efficiency, marks *may* be deducted if unnecessary tables are included in the query (for example including *Owns* and *Customer* when you only require the *customer ID* of customers who own accounts)

#### Query 1

*Last name, first name and salary* of employees who work in branch number 3, *order by last name, then first name*.

```sql
-- Question 1
SELECT lastName, firstName, salary
FROM Employee
WHERE branchNumber = 3
ORDER BY lastName, firstName;
```

#### Query 2

*Account number, account type, transaction number* and *transaction amount* of accounts with balances over $120,000, *order by account number, then transaction number*.

```sql
-- Question 2
SELECT A.accNumber, A.type, T.transNumber, T.amount
FROM Account A INNER JOIN Transactions T ON A.accNumber = T.accNumber
WHERE A.balance > 120000
ORDER BY A.accNumber, T.transNumber;
```

#### Query 3

*Last name, first name,* and *birth dates* of customers who were born before any customer named *Carol Alexander*, *order by last name then first name*.

```sql
-- Question 3
SELECT A.lastName, A.firstName, A.birthDate
FROM Customer A, Customer B
WHERE B.firstName = 'Carol' AND B.lastName = 'Alexander' AND A.birthDate < B.birthDate
ORDER BY A.lastName, A.firstName;
```

#### Query 4

*Customer ID, last name, income and account numbers* of customers with income over $60,000 who own an account with at least one transaction greater than (+)$110,000, *order by customer ID then account number*. The result should contain all the account numbers of customers who meet the criteria, even if the account itself does not contain any large transactions.

#### Query 5

*Owner customer ID, types, account numbers and balances* of *Berlin* and *London* business (type *bus*) and savings (type *sav*) accounts owned by customers who own at least one business and at least one savings account in the *Berlin* and *London* branches, *order by customer ID, then type, then account number*. That is, only include accounts owned by customers with: *sav* in *Berlin* and *bus* in *London*; *bus* in *Berlin* and *sav* in *London*; *bus* and *sav* in *Berlin*; or *bus* and *sav* in *London*.

#### Query 6

*Transaction number, transaction amount* and the *transaction amount as a percentage of the account balance* (i.e. three columns), for account number 42, *order by descending transaction amount*.

```sql
-- Question 6
SELECT Transactions.transNumber, Transactions.amount, 100*(Transactions.amount/Account.balance) AS percentage
FROM Transactions
INNER JOIN Account ON Transactions.accNumber = Account.accNumber
WHERE Account.accNumber = 42
ORDER BY Transactions.amount DESC;
```

#### Query 7

*Customer ID* of customers who have an account at the *New York* branch, who do *not* own an account at the *London* branch and who do *not* own an account with another customer with the same first or last name, *order by customer ID*. The result should not contain duplicate customer IDs.

#### Query 8

*Customer ID, last name, first name* and *income* of customers who have incomes greater than $60,000, if they have the same first and last name as an employee show their salary in a fifth column (which should be NULL for most customers), *order by last name and first name*. You must use an outer join in your solution (which is the easiest way to do it).

```sql
-- Question 8
SELECT c.customerID, c.lastName, c.firstName, c.income, e.salary
FROM Customer c
LEFT OUTER JOIN Employee e ON c.firstName = e.firstName AND c.lastName = e.lastName
WHERE c.income > 60000
ORDER BY c.lastName, c.firstName;
```

#### Query 9

Exactly as question eight, except that your query cannot include any join operation.

```sql
-- Question 9
SELECT c.customerID, c.lastName, c.firstName, c.income, e.salary
FROM Customer c, Employee e
WHERE c.income > 60000 AND e.firstName = c.firstName AND e.lastName = c.lastName
UNION ALL
SELECT c.customerID, c.lastName, c.firstName, c.income, NULL AS salary
FROM Customer c
WHERE c.income > 60000 AND NOT EXISTS (
	SELECT * FROM Employee
	WHERE c.income > 60000 AND firstName = c.firstName AND lastName = c.lastName)
ORDER BY c.lastName, c.firstName;
```

#### Query 10

*Customer ID, last name* and *first name* of customers who own accounts of all types, *order by customer ID*. The result should not contain duplicate customer IDs.

```sql
-- Question 10
SELECT c.customerID, c.lastName, c.firstName
FROM Customer c
INNER JOIN Owns o ON c.customerID = o.customerID
INNER JOIN Account a ON o.accNumber = a.accNumber
WHERE a.type IN ('CHQ', 'SAV', 'BUS')
GROUP BY c.customerID, c.lastName, c.firstName
HAVING COUNT(DISTINCT a.type) = 3
ORDER BY c.customerID;
```

#### Query 11

*Highest salary* (a single number) of a branch manager.

```sql
-- Question 11
SELECT MAX(Employee.salary) AS maxSalary
FROM Branch INNER JOIN Employee ON Branch.managerSIN = Employee.sin
```

#### Query 12

*SIN, first name, last name* and *salary* of the highest paid employee (or employees) of the *London* branch, *order by sin*.

```sql
-- Question 12
SELECT e.sin, e.firstName, e.lastName, e.salary
FROM Employee e
INNER JOIN Branch b ON e.branchNumber = b.branchNumber
WHERE b.branchName = 'London'
AND e.salary = (
	SELECT MAX(salary)
	FROM Employee
	WHERE branchNumber = b.branchNumber
)
ORDER BY e.sin;
```

#### Query 13

*Count* of the number of different first and last names of customers and the number of different first and last names of employees, four - sensibly named - numbers in a single row.

```sql
-- Question 13
SELECT 
	COUNT(DISTINCT c.firstName) AS customerFirstNames,
	COUNT(DISTINCT c.lastName) AS customerLastNames,
	COUNT(DISTINCT e.firstName) AS employeeFirstNames,
	COUNT(DISTINCT e.lastName) AS employeeLastNames
FROM Customer c, Employee e;
```

#### Query 14

*Branch name*, and *minimum, maximum* and *average salary* of the employees at each branch, *order by branch name*.

```sql
-- Question 14
SELECT Branch.branchName, 
	MIN(Employee.salary) AS minSalary, 
	MAX(Employee.salary) AS maxSalary, 
	AVG(Employee.salary) AS avgSalary
FROM Branch INNER JOIN Employee ON Branch.branchNumber = Employee.branchNumber 
GROUP BY Branch.branchName
ORDER BY Branch.branchName;
```

#### Query 15

*Customer ID, last name* and *birth dates* of customers who own accounts at a minimum of three different branches, *order by customer ID*.

```sql
-- Question 15
SELECT Customer.customerID, Customer.lastName, Customer.birthDate
FROM Customer
INNER JOIN Owns ON Customer.customerID = Owns.customerID
INNER JOIN Account ON Owns.accNumber = Account.accNumber
INNER JOIN Branch ON Account.branchNumber = Branch.branchNumber
GROUP BY Customer.customerID, Customer.lastName, Customer.birthDate
HAVING COUNT(DISTINCT Branch.branchNumber) >= 3
ORDER BY Customer.customerID;
```

#### Query 16

The *date of birth* of the oldest customer with an income *over* 90,000, the *date of birth* of the oldest customer with an income *less than or equal to* 90,000 and the difference in days between the two dates of birth; the result must have three named columns, with one row, in one result set (hint: look up T-SQL time and date functions).

#### Query 17

*Customer ID, last name, first name, income,* and *average account balance* of customers who have at least four accounts, and whose *last names* second letter is an *'r'* (e.g. Grey) or whose *first names* have a vowel as the first letter, but do not have a vowel as the last letter (e.g. Amy), *order by customer ID*. Note that this will be much easier if you look up LIKE wildcards in the MSDN T-SQL documentation. Also note - to appear in the result customers must have at least four accounts and satisfy one (or both) of the name conditions.

```sql
-- Question 17
SELECT c.customerID, c.lastName, c.firstName, c.income, AVG(a.balance) AS avgBalance
FROM Customer c
INNER JOIN Owns o ON c.customerID = o.customerID
INNER JOIN Account a ON o.accNumber = a.accNumber
WHERE (c.lastName LIKE '_r%') OR (c.firstName LIKE '[aeiou]%[^aeiou]')
GROUP BY c.customerID, c.lastName, c.firstName, c.income
HAVING COUNT(o.accNumber) >= 4
ORDER BY c.customerID
```

#### Query 18

*Branch number, branch name, budget, sum of account balances* and *sum of account balance / budget* (in a named column) of each branch, *order by branch name*.

```sql
-- Question 18
SELECT b.branchNumber, b.branchName, b.budget, SUM(a.balance) AS totalBalance, 
	SUM(a.balance)/b.budget AS balanceToBudgetRatio
FROM Branch b LEFT JOIN Account a ON b.branchNumber = a.branchNumber
GROUP BY b.branchNumber, b.branchName, b.budget
ORDER BY b.branchName;
```

#### Query 19

*Account number, minimum transaction amount, average transaction amount* and *maximum transaction amount* of accounts with more than ten transactions that are held at a branch which holds accounts of the customer with the second highest income, *order by account number*.

#### Query 20

*Customer ID, last name, account number, type* and *balance* of accounts of customers where the customer's average account balance is less than half of the average overall account balance. For example, if the average account balance (of all accounts) is $10,000 then return customers whose average account balances is less than $5,000. *Order by customer ID, then account number*. Note that all accounts of qualifying customers should be returned even if their balances are more than half the average balance.

```sql
-- Question 20
SELECT c.customerID, c.lastName, a.accNumber, a.type, a.balance
FROM Customer c
INNER JOIN Owns o ON c.customerID = o.customerID
INNER JOIN Account a ON o.accNumber = a.accNumber
WHERE c.customerID IN (
	SELECT o.customerID
	FROM Owns o
	INNER JOIN Account a ON o.accNumber = a.accNumber
	GROUP BY o.customerID
	HAVING AVG(a.balance) < (
		SELECT AVG(a.balance)/2
		FROM Account a))
ORDER BY c.customerID, a.accNumber;
```
