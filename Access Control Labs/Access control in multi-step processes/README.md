### Lab12: Multi-step process with no access control on one step
- In this lab, you have two account credentials.
- At first, log in with your credentials of `administrator:admin`, then go to the admin panel and check the functionality of changing the user level from `NORMAL` or `ADMIN` options where you can `Upgrade` or `Downgrade` the user.

![Lab6_Admin_panel.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab6_Admin_panel.png)
- A confirmation message Will appear to confirm the changing user role process.

![Lab12_ConfirmationMessage.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab12_ConfirmationMessage.png)
- In burp proxy the confirmation process seems like that:

![Lab12_AdminConfirmProcess.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab12_AdminConfirmProcess.png)
- Log in with wiener credentials `wiener:peter`.
- From `wiener` account, you can try to make the same confirmation process to upgrade your level.
- Capture any request from the `wiener` account and modify the request with the `/admin-roles` URL and add the parameters of the admin confirmation process in the request body:
```
action=upgrade&confirmed=true&username=wiener
```
- Then change the HTTP method request to `POST` method and send it.

![Lab12_UpgradeWiener.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Access%20Control%20Labs/images/Lab12_UpgradeWiener.png)
- It will upgrade you to admin.