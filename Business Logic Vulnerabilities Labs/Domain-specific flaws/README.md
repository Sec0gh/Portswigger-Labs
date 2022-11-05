# Domain-specific flaws
## Summary
- [Lab12: Flawed enforcement of business rules](#lab12-flawed-enforcement-of-business-rules)
- [Lab13: Infinite money logic flaw](#lab13-infinite-money-logic-flaw)

### Lab12: Flawed enforcement of business rules
- You will log in with your credentials `wiener:peter`.
- After logging in with your account, you will find a coupon code you can apply to take a discount on the total price when you place your order.

![Lab12_coupon.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab12_coupon.png)
- But if you scroll and go down, you will find you can sign up for the newsletter.

![Lab12_newsletter.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab12_newsletter.png)
- If you signed up, you will get another coupon code.

![Lab12_newcoupon.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab12_newcoupon.png)
- I selected the jacket item then I added the first coupon code I have gotten, but if you tried adding the same coupon code once again, the coupon code won't apply to make the discount.

![Lab12_errorMessage.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab12_errorMessage.png)
- But if you tried to alternate with two coupons, there is an issue happening in the web app that you can use the coupon again but when you apply another coupon.
- I repeated this process to alternate between the two coupons until I reached to make the total price of zero.
- Now I can place my order.

![Lab12_placeOrder.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab12_placeOrder.png)

----
### Lab13: Infinite money logic flaw
- In this lab, I didn't use the burp, I have gone to automate my script.
- You can check it out here: [Infinite money logic flaw exploit](https://github.com/Sec0gh/python-scripts/tree/main/Infinite%20money%20logic%20flaw)
### `Note:` 
- At first, You must login and capture any request with any proxy to capture the csrf value to use it as argument in the script or you can get the CSRF value from the source code.
 
![Lab13_CSRF.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab13_CSRF.png)
- The automated script will work like that and you will find there is an increase in the credit store:

![Lab13_ScriptOuptut.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab13_ScriptOuptut.png)

![Lab13_StoreCredit.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab13_StoreCredit.png)
- Now we can place the order.
