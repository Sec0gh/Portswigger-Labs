# Business Logic Vulnerabilities - Excessive trust in client-side controls
## Summary
- [Lab1: Excessive trust in client-side controls](#lab1-excessive-trust-in-client-side-controls)
- [Lab2: 2FA broken logic](#lab2-2fa-broken-logic)

### Lab1: Excessive trust in client-side controls
- In first, we will log in with our credentials which we have, then go to the  `Lightweight "l33t" Leather Jacket` item and add it to the cart. 
- Run the burp suite and capture the request, and change the price as you like but don't more than `$100.00` which is available in the credit store, and also change the quantity as you need, it's so simple lab.

![Lab1_ChangePrice&quantity.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab1_ChangePrice%26quantity.png)
- Then place the order to complete your purchase workflow.

![Lab1_price.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab1_price.png)

-----
### Lab2: 2FA broken logic
- You can check out the solution for this lab from multi-factor authentication vulnerabilities labs.
- Check out here: [2FA broken login](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Authentication%20Labs/Vulnerabilities%20in%20multi-factor%20authentication#lab8-2fa-broken-logic)

