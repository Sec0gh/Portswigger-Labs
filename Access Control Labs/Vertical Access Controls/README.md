# Vertical Access Controls

## Summary
- ### [Unprotected functionality](#unprotected-functionality-1)
	- [Lab1: Unprotected admin functionality](#lab1-unprotected-admin-functionality)
	- [Lab2: Unprotected admin functionality with unpredictable URL](#lab2-unprotected-admin-functionality-with-unpredictable-url)
- ### [Parameter-based access control methods](#parameter-based-access-control-methods-1)
	- [Lab3: User role controlled by request parameter](#lab3-user-role-controlled-by-request-parameter)
	- [Lab4: User role can be modified in user profile](#lab4-user-role-can-be-modified-in-user-profile)
- ### [Broken access control resulting from platform misconfiguration](#broken-access-control-resulting-from-platform-misconfiguration-1)
	- [Lab5: URL-based access control can be circumvented](#lab5-url-based-access-control-can-be-circumvented)
	- [Lab6: Method-based access control can be circumvented](#lab6-method-based-access-control-can-be-circumvented)

## Unprotected functionality
### Lab1: Unprotected admin functionality
- Go to the `robots.txt` file.

![Lab1_RobotsFile.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab1_RobotsFile.png)
- By accessing the `administrator-panel` page , you will access the admin panel.
-----
### Lab2: Unprotected admin functionality with unpredictable URL
- I just viewed the page source, and I found the JS code discloses an unpredictable page name for the admin panel.
- So it is very important and sometimes useful to view the page source and discover it.
![Lab2_unpredictableURLdisclosed.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab2_unpredictableURLdisclosed.png)
- When I added the `/admin-odwyec` page to the URL, I accessed the admin panel.
----
## Parameter-based access control methods
### Lab3: User role controlled by request parameter
- Log in with your credentials of `wiener:peter`.
- And then you will see there is an `admin` parameter in the cookie, it identifies the admin from the value that is set there.

![Lab3_AdminParameterCookie.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab3_AdminParameterCookie.png)
- We can change the value to `true`, and if you did that you will find the `admin panel` appeared and can be accessed.
-----
### Lab4: User role can be modified in user profile
- Log in with your credentials of `wiener:peter`.
- In the `/my-account` page, you can update your email.
- Try to update it and send the request to see what will happen.

![Lab4_myaccount.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab4_myaccount.png)
- The web app sends the new email that we update in the `JSON` format, and also the server responds with JSON data where one of the keys is to identify the role for the users in `roleid`.

![Lab4_UpdateEmail.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab4_UpdateEmail.png)
- Try to modify the value for the `roleid` to 2 and send it with the JSON data in the body request, you will notice the users just with `roleid` is 2 will access the admin panel.
```json
{
  "email":"test@gmail.com",
  "roleid":2
}
```
------
## Broken access control resulting from platform misconfiguration
### Lab5: URL-based access control can be circumvented
- If you sent any normal request to access the admin panel, you will get an error message `"Access denied"`.
- ==Some applications support non-standard headers such as X-Original-URL or X-Rewrite-URL in order to allow overriding the target URL in requests with the one specified in the header value.==
- Check out [owasp.org](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema)
- We can exploit these headers to access the specific URLs which we can not access.
- To detect the support for the header `X-Original-URL`, you will find the backend responded with a `Not Found` message, it is meaning the server deal with the URL which I added to the header.

![Lab5_donotExist.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab5_donotExist.png)
- Go to add the `/admin` to the header, it will let you access the admin panel.

![Lab5_AccessAdminPanel.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab5_AccessAdminPanel.png)
- To delete the `carlos` user, you need to set the path of deleting users `/admin/delete` in the header, and then in the real `GET` URL add the `username` parameter and its value.

![Lab5_DeleteCarlos.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab5_DeleteCarlos.png)

-----
### Lab6: Method-based access control can be circumvented
- In this lab, you have two account credentials.
- At first, log in with your credentials of `administrator:admin`,  then go to the admin panel and check the functionality of changing the user level from `NORMAL` or `ADMIN` options where you can `Upgrade` or `Downgrade` the user.

![Lab6_Admin_panel.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab6_Admin_panel.png)
- In the `/admin-roles` request body:

![Lab6_AdminReqBody.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab6_AdminReqBody.png)
> `The aim`:  do we can use the same functionality and execute it with a low-level user?
- So now we can move to check that and go to log in with credentials of `wiener:peter`.
- I tried to update the email for `wiener` to send any request from the wiener session:

![Lab6_UpdateEmailWiener.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab6_UpdateEmailWiener.png)
- Then I changed the body request with the parameters of the admin request and also the target URL `/admin-roles`:
```
username=wiener&action=upgrade
```

![Lab6_wienerReq.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab6_wienerReq.png)
- But it will give you an error message, it's an `Unauthorized` session.
- By changing the HTTP request method to `GET`, it will execute the same functionality and access it to change the user level from `NORMAL` to `ADMIN`.

![Lab6_Foothold.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab6_Foothold.png)
