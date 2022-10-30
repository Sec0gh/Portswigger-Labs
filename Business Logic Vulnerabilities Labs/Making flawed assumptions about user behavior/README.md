# Making flawed assumptions about user behavior

## Summary 
- [Lab6: Inconsistent security controls](#lab6-inconsistent-security-controls)
- [Lab7: Weak isolation on dual-use endpoint](#lab7-weak-isolation-on-dual-use-endpoint)
- [Lab8: Password reset broken logic](#lab8-password-reset-broken-logic)
- [Lab9: 2FA simple bypass](#lab9-2fa-simple-bypass)
- [Lab10: Insufficient workflow validation](#lab10-insufficient-workflow-validation)
- [Lab11: Authentication bypass via flawed state machine](#lab11-authentication-bypass-via-flawed-state-machine)

### Lab6: Inconsistent security controls
- At first, we need to register with our email but we can not use the domain of `@dontwannacry.com` in the registration, because we won't receive the confirmation link to complete the registration and we don't work for DontWannaCry.

![Lab6_Register.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab6_Register.png)
- After registration, we can confirm the registration from the `Email client` and log in with the account we created.
- when you log in, you will find an option to update your email, since you can exploit that to your advantage, and update your email with this domain `@dontwannacry.com` .

![Lab6_UpdateEmail.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab6_UpdateEmail.png)
- The web app doesn't check for the email adress, so the account updated successfully, and you can access the admin panel.

![Lab6_UpdateSuccessfully.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab6_UpdateSuccessfully.png)

---
### Lab7: Weak isolation on dual-use endpoint 
- At first, We will log in with this credentials: `wiener:peter`.
- This time you have option to change the password for your account.

![Lab7_ChangingPassword.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab7_ChangingPassword.png)
- Run burp suite and capture the request for the changing password process.
- After some tests and observing the behavior of this process, you will find the app checks the `username` parameter if it is a valid user or not, and there is another issue when you remove the `current-password` parameter, the app behaves normally and appears this message ` Password changed successfully!`.

![Lab7_RequestBody.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab7_RequestBody.png)
- We can exploit that to change the password for the `administrator` and removing the `current-password` parameter doesn't impact in the behavior of the application when we don't set the current password for the admin.

![Lab7_ChangingAdminPassword.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab7_ChangingAdminPassword.png)
- The password was changed successfully for the administrator, you can log in with the new credentials for the admin and access the admin panel.
---
### Lab8: Password reset broken logic
- You can check out the solution for this lab from `vulnerabilities in other authentication mechanisms` labs.
- Check out here: [Password reset broken logic](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/Vulnerabilities%20in%20other%20authentication%20mechanisms/README.md#lab12-password-reset-broken-logic)
---
### Lab9: 2FA simple bypass
- You can check out the solution for this lab from multi-factor authentication vulnerabilities labs.
- check out here: [2FA simple bypass](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Authentication%20Labs/Vulnerabilities%20in%20multi-factor%20authentication#lab7-2fa-simple-bypass)
---
### Lab10: Insufficient workflow validation
- You will log in first with the credentials: `wiener:peter`.
- You can try to add any item to the cart and observe how the process of placing an order happens.
- When you place the order it redirects you from `/cart/checkout` page to `/cart/order-confirmation?order-confirmed=true`.
- Notice the `order-confirmed` with `True` value, it just confirmed the order through this value that you bought the item and you have the price of it.

![Lab10_addHighItemPrice.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab10_addHighItemPrice.png)
- But if you tried to buy an item like this, and you don't have its price, the web app will redirect you to this page:
```
https://web-security-academy.net/cart?err=INSUFFICIENT_FUNDS
```
- And you will get this error message:

![Lab10_NotEnough.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab10_NotEnough.png)
- So you can just modify the URL with  `/cart/order-confirmation?order-confirmed=true` to confirm the process and exploit the flaw in the sequence of purchasing workflow.
```
https://web-security-academy.net/cart/order-confirmation?order-confirmed=true
```
---
### Lab11: Authentication bypass via flawed state machine
- As usual first you need to understand the sequence and steps in which the application works. 
- You will log in with your credentials `wiener:peter`, and you will find the `/role-selector` page which asks you to select your role to let you complete the login process.

![Lab11_RoleSelector.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab11_RoleSelector.png)
- Pretty, But what will happen if we didn't load this page and bypassed this step from the process?
- Just run the burp or any proxy then perform the log-in and forward the request of the `/login` page and then drop the `role-selector` page.

![Lab11_DropRequest.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab11_DropRequest.png)
- Then go to the home page, you will be the admin and you can access the admin panel, it has happened cause you bypassed the identify the role, and the web app identified your role as the admin by default.
