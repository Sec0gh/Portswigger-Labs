# Horizontal Access Controls
## Summary
- [Lab7: User ID controlled by request parameter](#lab7-user-id-controlled-by-request-parameter)
- [Lab8: User ID controlled by request parameter, with unpredictable user IDs](#lab8-user-id-controlled-by-request-parameter-with-unpredictable-user-ids)
- [Lab9: User ID controlled by request parameter with data leakage in redirect](#lab9-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
- ### Horizontal to vertical privilege escalation
	- [Lab10: User ID controlled by request parameter with password disclosure](#lab10-user-id-controlled-by-request-parameter-with-password-disclosure)
- [Lab11: Insecure direct object references](#lab11-insecure-direct-object-references)

### Lab7: User ID controlled by request parameter
- Log in with credentials of `wiener:peter`.
- And just access the `/my-account?id=wiener` URL, then if you changed the value for the `id` parameter to `carlos`, you will access the `carlos` account and find `API key` of `carlos`.
---
### Lab8: User ID controlled by request parameter, with unpredictable user IDs
- Log in with credentials of `wiener:peter`.
- You can not access any user because you can not predict any user `id` and it looks like that:
```
id=3a3d3c87-4081-436e-97df-f44e3816e0eb
```
- So during browsing posts, you will find a post with `carlos` name.

![Lab8_carlosPost.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab8_carlosPost.png)
- Check this name, observe the URL, and notice the `userId` parameter discloses `carlos` id like that:
```
/blogs?userId=cb4b96e6-0424-4cf7-a52d-4b7f7b0529d1
```
- Go to `/my-account?id=...` and change your id to `carlos` id.
----
### Lab9: User ID controlled by request parameter with data leakage in redirect
- Log in with credentials of `wiener:peter`.
- Try to change the value in the `id` parameter with `carlos` name, you will find the web app redirects you to the `/login` page again.
- But check always the request with the proxy.
- So the situation in the burp suite seems different, repeat this action and change the value in the `id` parameter with `carlos` name and observe what happens:

![Lab9_ LeakageInRedirect.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab9_%20LeakageInRedirect.png)
- During the web app redirecting you to the `/login` page, it discloses information in the traffic about `carlos` and his API key.
----
## Horizontal to vertical privilege escalation
### Lab10: User ID controlled by request parameter with password disclosure
- Log in with credentials of `wiener:peter`.
- The `/my-account` page contains functionality to update the password.

![Lab10_UpdatePassword.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab10_UpdatePassword.png)
- You can change the value for the `id` parameter and see the page for the user who you set his name.
- Change the parameter value with the `administrator`, observe the admin's password from the proxy or inspect the element of the password from the page source, and go to log in with admin credentials to delete the `carlos` user.

![Lab10_AdminPassword.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab10_AdminPassword.png)

----
### Lab11: Insecure direct object references
- You will find a live chat, try to send some messages, and view the transcript in the burp proxy.

![Lab11_LiveChat.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab11_LiveChat.png)
- When you try to view the transcript, the transcript will be downloaded into a text file, and each file has a name with a different number.

![Lab11_TextFile.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab11_TextFile.png)
- Go to change the number of the file name to `1.txt` and see the transcript in the response, you will find the password of the `carlos`.

![Lab11_password.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab11_password.png)
- Take this password and log in with it as a `carlos` user.
