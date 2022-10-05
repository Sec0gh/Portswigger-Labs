# Vulnerabilities in password-based login
## Summary
- [Lab1: Username enumeration via different responses](#lab1-username-enumeration-via-different-responses)
- [Lab2: Username enumeration via subtly different responses](#lab2-username-enumeration-via-subtly-different-responses)
- [Lab3: Username enumeration via response timing](#lab3-username-enumeration-via-response-timing)
- [Lab4: Broken brute-force protection, IP block](#lab4-broken-brute-force-protection-ip-block)


> `Please note: All labs can be solved with burb intruder but somtimes we will solve it by automate the lab with python.`

### Lab1: Username enumeration via different responses
- In first, if you tested manually by entering any username and password, you will notice a message that appears saying `Invalid username`.
- We can use it to look for a valid username to find a different response through brute force.
- After finding the correct username, if we entered any wrong password, also you will notice a message saying `Incorrect password`.
- We will do the same thing to brute-force the password with our valid user.
- I have written a python script to do that automatically.
- Check out my script from here: [Lab1_username&password_auth.py](https://github.com/Sec0gh/python-scripts/blob/main/Authentication%20scripts/Lab1_username%26password_auth.py)
- The output will look like that:

![Authentication Labs/images/lab1.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab1.png)

------------

### Lab2: Username enumeration via subtly different responses
- In brief, we can say there is a difference when you enter the wrong username, the error message will be like that `Invalid username or password.`.
- But during brute-forcing you will notice a difference in the error message, it is without a full stop mark.

![lab2_username.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab2_username.png)

- Check out my script: [Lab2_username&password_auth.py](https://github.com/Sec0gh/python-scripts/blob/main/Authentication%20scripts/Lab2_username%26password_auth.py)

![lab2.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab2.png)

------------

### Lab3: Username enumeration via response timing
- Many applications perform some processing on the client’s IP address to carry out functions such as logging, access control, or user geolocation. 
- So we will add the `X-Forwarded-For` header to our request to spoof our IP address and change each request through brute force with a new value for each request we send.
- Applications may use the IP address specified in the `X-Forwarded-For` request header if it is present.
- Check out this article is very helpful: [geeksforgeeks](https://www.geeksforgeeks.org/http-headers-x-forwarded-for/)
- In the first, we will brute force the username and value of the `X-Forwarded-For` header, and then we will set the password and repeat it a lot to trigger a time delay in response.

![lab3_intruder_username.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab3_intruder_username.png)

![lab3_username.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab3_username.png)
- After we noticed the time delay in the response revieved, we got the right username then we will use it to brute force the password.  

![lab3_intruder_password.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab3_intruder_password.png)

- During trying to filter the responses to catch the password we will see there is a response with a status code of `302`, it is a redirection that happened.

![lab3_password.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab3_password.png)
- Here, we used Burp suite to solve the lab but **You can see my script which I wrote to perform this task:** [Lab3_ResponseTiming](https://github.com/Sec0gh/python-scripts/blob/main/Authentication%20scripts/Lab3_ResponseTiming.py)
-----------

### Lab4: Broken brute-force protection, IP block
- Here, we will exploit a flaw in the brute-force attack protection, where it will block your IP address if you tried to login in after many failed login attempts.

![lab4_block.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab4_block.png)
- So we can log in with our account to reset the counter of the number of failed login attempts.
- We can log in with the right credentials of our account and send it to reset the counter after every failed attempt, then it will let me complete my brute-force attack.
- I wrote my script to do this task check out it from here: [Lab4_BypassBlockingIP](https://github.com/Sec0gh/python-scripts/blob/main/Authentication%20scripts/Lab4_BypassBlockingIP.py)
- The output will appear like that:

![lab4.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Authentication%20Labs/images/lab4.png)
