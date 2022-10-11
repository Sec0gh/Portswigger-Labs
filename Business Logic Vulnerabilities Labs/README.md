# Business Logic Vulnerabilities
## Summary
- #### [Excessive trust in client-side controls](#excessive-trust-in-client-side-controls-1)
	- [Lab1: Excessive trust in client-side controls](#lab1-excessive-trust-in-client-side-controls)
	- [Lab2: 2FA broken logic](#lab2-2fa-broken-logic)
- #### [Failing to handle unconventional input](#failing-to-handle-unconventional-input-1)
	- [Lab3: High-level logic vulnerability](#lab3-high-level-logic-vulnerability)
	- [Lab4: Low-level logic flaw](#lab4-low-level-logic-flaw)
	- [Lab5: Inconsistent handling of exceptional input](#lab5-inconsistent-handling-of-exceptional-input)


## Excessive trust in client-side controls
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
----
## Failing to handle unconventional input
### Lab3: High-level logic vulnerability
- In first, we will log in with our credentials which we have, then go to the  `Lightweight "l33t" Leather Jacket` item and add it to the cart. 
- Go to any item of the products and add it to the cart and run burp suite then capture the request.
- If you tried to enter a negative numeric number in the `quantity` parameter, you will notice it deals with the negative value as a conventional value, and you will receive succeeded response with a `302` status code.

![Lab3_negativenumber.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab3_negativenumber.png)
- Follow the redirect, and show the response in the browser.
- Try to submit and complete the order, but it won't work because The total price with a negative number and is less than zero.

![lab3_cart.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/lab3_cart.png)
- So we will try to make the total price with a positive number, but it must be in the available money which we have in the credit store.

![Lab3_completeOrder.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab3_completeOrder.png)
- Now we can complete the order.

![Lab3_completedOrder.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab3_completedOrder.png)

-----
### Lab4: Low-level logic flaw
- In first, we will log in with our credentials which we have, then go to the  `Lightweight "l33t" Leather Jacket` item and add it to the cart.
- We can select any item, and add it to the cart, and capture this request then send it to repeater to test the `quantity` parameter.
- if you tried to add any value you bigger than `99`, it will response with error message:

![Lab4_ErrorMessage.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab4_ErrorMessage.png)
- So this parameter doesn't take any number unless it is 2-digits.
- You can send the requested item and brute force with null payloads because we just need to send the maximum value for the parameter that can be taken and repeat this process.
- Don't add any positions in the intruder before starting the attack.

![Lab4_intruder.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab4_intruder.png)
- If you tried to send a big number of quantities of an item, you will find the total price reaches the maximum count for numbers in the backend programming language then it will start counting from the negative number with also the big number in the total price.

![Lab4_cart.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab4_cart.png)
- Now we will try to add big suitable quantities from any item to make the total price with a positive number, but it must be in the available money which we have in the credit store.
- You can do that easily with burp suite, and I think using some maths to calculate that will help you.

![Lab4_cart2.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab4_cart2.png)
- The order has been completed successfully.

![Lab4_CompletedOrder.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab4_CompletedOrder.png)

----
### Lab5: Inconsistent handling of exceptional input
- The email address for us from the web security academy will be like that where we will register with this email : 
> attacker@exploit-0a18009203351792c05c09e3015f00d9.exploit-server.net 
- The most important thing is testing and understanding the behavior and action of the application.
- Now I will register:

![Lab5_RegistrationProcess.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab5_RegistrationProcess.png)
- It will send me a link to confirm and complete the registration.  

![Lab5_ConfirmRegistration.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab5_ConfirmRegistration.png)
- When I will log in, it will redirect me to the `/my-account` page and Indicates to I have been registered successfully.

![Lab5_MyAccount.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab5_MyAccount.png)
- When I try to register with `@dontwannacry.com` domain, the email client doesn't send me a link to confirm the registration.

![Lab5_ dontwannacry.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab5_%20dontwannacry.png)
- Ok, now we will use a technique of truncation attack, it works when it truncates the plaintext which reaches and exceeds its max length of it.
- I will repeat the string to set a long string in the email address field to know what is the max length for the email address.
```
sec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0ghsec0gh@exploit-0a3f008004d13054c0f94b1c01be0092.exploit-server.net
```
- When I registered with this email address I noticed it was truncated to be like this:

![Lab5_knowThelength.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab5_knowThelength.png)
- If we tried to know the length of the email address after truncating, you will find the length is `255` characters.

![lab5_length.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/lab5_length.png)
- We need to register with the specific domain `@dontwannacry.com` of the company to access the admin panel, so the length of this domain is `17` characters.
- We can set an email with a long string of `238` characters plus this specific domain to make the total length is `255` characters.
- But if we sent this email, also it won't send any link to confirm the registration because it is not a valid email.
- Now we can add our email id of the web security academy at the end of the email just to receive the confirmation link of registration.

> Look at this equation:
> Email = `Long string` + `@dontwannacry` + `web security academy id`
- We will set `dontwannacry` as a subdomain for the `web security academy id`, and then after receiving the confirmation link, the web application will truncate the excess part of the email address.
- The final email address form will be like that:
```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@dontwannacry.com.exploit-0a3f008004d13054c0f94b1c01be0092.exploit-server.net
```

![Lab5_attacker.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/Lab5_attacker.png)
- After successfully registering, you can log in with your username and password, and finally, you will find you can access the admin panel.

![lab5_AttackSucceeded.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Business%20Logic%20Vulnerabilities%20Labs/images/lab5_AttackSucceeded.png)
- Go to the admin panel and delete the `Carlos` user.
