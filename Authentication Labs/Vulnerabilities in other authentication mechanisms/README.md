# Vulnerabilities in other authentication mechanisms
## Summary
- [Lab10: Brute-forcing a stay-logged-in cookie](#lab10-brute-forcing-a-stay-logged-in-cookie)
- [Lab12: Password reset broken logic](#lab12-password-reset-broken-logic)
- [Lab14: Password brute-force via password change](#lab14-password-brute-force-via-password-change)

### Lab10: Brute-forcing a stay-logged-in cookie
- You will log in with these credentials: `wiener:peter`.
- And if any user selected the option of `Stay logged in`, it will set a cookie for the user to make him logged in even after the user closes the browser because the web application when generates this token, the web application uses it to identify the user and skip the step of the login for this specific user, this feature a lot of web applications use it.

![Lab10_login.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab10_login.png)
- After logging in, if you checked your session, you will find there is a parameter called `stay-logged-in` has been added.

![Lab10_cookies.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab10_cookies.png)
- If we tried to decode it, you will notice it is a base64 encoding, you can detect that with any tool you appropriate
- You can know that with something like [CyberChef](https://gchq.github.io/CyberChef/).

![Lab10_decodeCookie.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab10_decodeCookie.png)
- When I decoded that I found the username and concatenated with `:` and then `md5` hash(32-character).
- If you tried to crack the md5 hash, you will find it is a password for the user.

![Lab10_CrackTheHash.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab10_CrackTheHash.png)
- So the cookie is: `base64(username:md5(password))`.
- We can exploit that by brute force the cookie for the `carlos` user.
- I wrote a python script to guess the password of the victim.
- Check out my script from here: [Lab10_BruteForcingcookie.py](https://github.com/Sec0gh/python-scripts/blob/main/Authentication%20scripts/Lab10_BruteForcingCookie.py)
- The output will be like that:

![Lab10_password&cookie.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab10_password%26cookie.png)

----
### Lab12: Password reset broken logic
- This lab needs us to reset the password for the victim, and this web application uses a method for resetting passwords by sending a unique URL to users that takes them to a password reset page.

![Lab12_login.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab12_login.png)
- When we select `Forgot password`, it will take us to `/forgot-password` page to enter the username or email to send the unique URL for him and access the password reset page.

![Lab12_enterUsername.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab12_enterUsername.png)
- Check out the `Email client`, you will find the unique URL is about a unique token for the `wiener` user, it will look like that:
```
https://0a7b00ea03fc3d92c06df30f00a5001d.web-security-academy.net/forgot-password?temp-forgot-password-token=GUe8lP0Z5uqEmWp9X8NdWbIm8hJdHpgp
```
- When you access the token and go to the password reset page, and capture the request, you will notice there is a parameter called `username` which is sent also in the request body.
- If you tried to change the value in the `username` parameter to `carlos`, the password will be reset successfully, and it happen cause there is an issue where the web app doesn't check the `temp-forgot-password-token` in the back-end for the intended user, if you tried to remove this parameter and the token value, it will not impact and the web app doesn't notice we did suspicious action.

![Lab12_resetPassword.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab12_resetPassword.png)
- After that go to the log-in page and use the new password for the `carlos` user.
---
### Lab14: Password brute-force via password change
- You will log in with these credentials: `wiener:peter`.
- Then you will find the password change page, after some tests you will notice some messages appear for you.
- If you entered the correct current password and a new password in the 2 parameters `new-password-1` and `new-password-2` matching themself, the password will change successfully.

![Lab14_changedSuccessfully.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab14_changedSuccessfully.png)
- If you entered the wrong current password and two new passwords match, the account will lock and redirect you to the log-in page again.
- But if you entered the wrong current password and two different new passwords, this error message will appear to tell you the current password is incorrect.

![Lab14_passwordIncorrect.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab14_passwordIncorrect.png)
- The final case, if you entered the correct current password but two different new passwords, the web app will tell you the new passwords don't match themself.

![Lab14_doesn'tmatch.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab14_doesn'tmatch.png)
- So We will brute force the correct password for the victim which will be known when the web app checks the current password and find it's the correct current password and the 2 new passwords don't match.
- I wrote a python script to automate this task.
- check out here: [Lab14_BruteForcer.py](https://github.com/Sec0gh/python-scripts/blob/main/Authentication%20scripts/Lab14_BruteForcer.py)
- The output will be like that:

![Lab14_thePassword.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab14_thePassword.png)
