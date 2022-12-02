### Lab13: Referer-based access control
- In this lab, you have two account credentials.
- At first, log in with your credentials of `administrator:admin`, then go to the admin panel and check the functionality of changing the user level from `NORMAL` or `ADMIN` options where you can `Upgrade` or `Downgrade` the user.

![Lab6_Admin_panel.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab6_Admin_panel.png)
- Log in with wiener credentials `wiener:peter`.
- From the `wiener` account, capture any request like `my-account?id=wiener`:

![Lab13_wiener.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab13_wiener.png)
- And then change the `GET` request with the `GET` request of admin and its parameters, try to send that but the server will respond with an `Unauthorized` error message.

![Lab13_Unauthorized.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab13_Unauthorized.png)
- Go to the admin request, and you will find a header called `Referer`.

![Lab13_Referer.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab13_Referer.png)
> The `Referer` header is added to the request by the browser to indicate the page from which a request was initiated.

- Take the `Referer` header and its value from the admin request and add it in the wiener request with the headers and send it to the server to identify you came from the `/admin` destination.
