# Information Disclosure
## Summary
- [Lab1: Information disclosure in error messages](#lab1-information-disclosure-in-error-messages)
- [Lab2: Information disclosure on debug page](#lab2-information-disclosure-on-debug-page)
- [Lab3: Source code disclosure via backup files](#lab3-source-code-disclosure-via-backup-files)
- [Lab4: Authentication bypass via information disclosure](#lab4-authentication-bypass-via-information-disclosure)
- [Lab5: Information disclosure in version control history](#lab5-information-disclosure-in-version-control-history)

### Lab1: Information disclosure in error messages
- I have just tried to causing an error in the `product Id` parameter by adding single quote.
- Choose any product and causing in any error by setting any random values as you like, therefore it will be an unexpected data type to the parameter.
```
/product?productId=2'
```
- It caused a long error message and disclose the version of `Apache Struts` web app framework:

![Lab1_DisclosureErrorMessage.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab1_DisclosureErrorMessage.png)

----
### Lab2: Information disclosure on debug page
- View the page source at the bottom, you will find a comment that disclose path for the `phpinfo.php` file.

![Lab2_comment.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab2_comment.png)
- Access the file.
```
https://web-security-academy.net/cgi-bin/phpinfo.php
```
- You will find some information about the environment and the `SECRET_KEY`:

![Lab2_secretKey.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab2_secretKey.png)


---
### Lab3: Source code disclosure via backup files
- If you tried to access the `robots.txt` file, you will find the `/backup` directory.

![Lab3_RobotsFile.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab3_RobotsFile.png)
- Go to access the `/backup` directory, you will find a backup file called `ProductTemplate.java.bak` containing the source code.

![Lab3_backup.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab3_backup.png)

```
https://web-security-academy.net/backup/ProductTemplate.java.bak
```
- If you showed the source code, you will find a hard-coded password for the PostgreSQL database.

![Lab3_DatabasePassword.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab3_DatabasePassword.png)

----
### Lab4: Authentication bypass via information disclosure
- In the first step, go to log in with this credentials: `wiener:peter`.
- This lab talk about there is an insecure configuration that happened where the `TRACE` method is enabled.
- The HTTP TRACE method is designed for diagnostic purposes.
- **If `TRACE` method enabled, the web server will respond to requests that use the TRACE method by echoing in the response the exact request that was received.**
- Here, in the response when I used the `TRACE` method in the request, the server disclosed some information about my request and you will find there is a header called `X-Custom-IP-Authorization` that takes a value of my IP address.

![Lab4_TRACE.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab4_TRACE.png)
- We can add this header and modify its value to the localhost IP and send it.

![Lab4_AdminPanel.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab4_AdminPanel.png)
- Now, you can access the admin panel and delete `carlos`.
```
curl -X GET https://web-security-academy.net/admin/delete?username=carlos -H "X-Custom-IP-Authorization: 127.0.0.1" 
```
---
### Lab5: Information disclosure in version control history
- From the lab name it talks about the version control systems and its histories such as `Git & GitHub` which is being in the `/.git` directory.

> Version control system: keeps track of the changes we make to our files, and we can make edits to multiple files and treat a collection of edits as a single change, which is commenly known as a `commit`.

```
$ dirb https://web-security-academy.net/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Nov  3 14:58:03 2022
URL_BASE: https://web-security-academy.net/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: https://web-security-academy.net/ ----
+ https://web-security-academy.net/.git/HEAD (CODE:200|SIZE:23)                                                    
```
- Download the `./git` directory:
```
$ wget -r  https://0afd007603249cf1c132178000f500ea.web-security-academy.net/.git/
```
- We will try to see the logs and committed changes.

- `git log`: show all commits in the current branch's history.
- `git show` with a commit ID(SHA): shows the changes made in a particular commit. 
- `git diff ID1..ID2`: To see the changes _between_ two commits.

![LAb5_commit.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/LAb5_commit.png)

![Lab5_commit2.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab5_commit2.png)
- This is the difference that happened between those 2 commits:

![Lab5_diff.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Information%20Disclosure/images/Lab5_diff.png)
- You will find the administrator password, and you can log in with it.
