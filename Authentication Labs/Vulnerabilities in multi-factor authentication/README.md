# Vulnerabilities in multi-factor authentication
## Summary
- [Lab7: 2FA simple bypass](#lab7-2fa-simple-bypass)
- [Lab8: 2FA broken logic](#lab8-2fa-broken-logic)

### Lab7: 2FA simple bypass
##### **Understanding the behavior of the web application**
- In first, we need to try to understand the behavior of web application, and we will do that through our own account.
- We will log in with `wiener:peter` credentials.
- After logging in, it will redirect you to the `/login2` page to enter your security code.

![securityCode.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/securityCode.png)
- When we get the `security code` from the email server and enter that, it will redirect us to `/my-account` page.
##### **How do we exploit that?**
- Now I logged out from my account and then I will access the account of the victim and the credentials are `carlos:montoy`.
- After logging in with these credentials try to access `/my-account` page directly without entering the security code.
- You will find yourself in a `logged in` state and, you accessed the `carlos` account before entering the verification code.
---
### Lab8: 2FA broken logic
##### **Understanding the behavior of the web application**
- In first, during logging in with the credentials of `wiener:peter`,  the web app will set cookies in the `verify` parameter with the same username which I used to log in.

![Lab8_login.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab8_login.png)
- Then it will redirect me to the `/login2` page to enter my verification code with my cookie of `wiener` which is in the `verify` parameter where the web application identifies and verifies from the user intended with its cookie.

![Lab8_login2.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab8_login2.png)
##### **How do we exploit that?**
- I will change the `verify` parameter with `carlos` value who is the victim I need to get his security code. 
- When I change this value, I will send this request to make sure the `2FA` code of `carlos` user is created.

![Lab8_carlosCookie.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab8_carlosCookie.png)
- Go again to the `/login2` page and enter any invalid code and capture the request and change the cookie value of the `verify` parameter to `carlos` then send it to the burp intruder to brute force the `mfa-code` parameter.

![Lab8_intruder.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab8_intruder.png)

![Lab8_verificationCode.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab8_verificationCode.png)
- Rearrange the length of responses, and you will find a response with `302` status code, which indicates redirection that happened to any other page.
- Take the valid security code and send it with `carlos` cookie.

![Lab8_bypass.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/Lab8_bypass.png)
- Show response in the browser, you will find yourself in a `logged in` state, and the lab is solved.
