# `File Upload`
### Lab1: Remote code execution via web shell upload
- In the first, When I tested what kind of file can I upload other than the image and I uploaded a file with the extension `.php`, the server accepted it.
- when I uploaded my file, this path for my uploaded file appeared but I noted it is not the full path for the file because when I uploaded an image and saw the path I found the path like that `/files/avatars/image.png`.

(https://github.com/Sec0gh/Portswigger-Labs/blob/main/File%20Upload%20Labs/images/path.png)
- Then i accessed the full right path.
 
![[path_lab1.png]]
- I will run my PHP script and reload the previous path to display the file's content.

![[lab1.png]]
- Here we find the restrictions in the server are very low because it does not validate and sanitize the name, type, content, or size of the file you uploaded.
-----------------------------------------------------------------------

### Lab2: Web shell upload via `Content-Type` restriction bypass
- Here I checked some locations:
	1. I changed the extension of the image to `.php` and sent the request and didn't happen any error message.
	2. I changed the content of the image and replaced it with the malicious script, send and didn't happen anything strange.
	3. In the last, I tried to change the `Content-Type` header but here caused this message:
		   
	![[message lab2.png]]
- Here we conclude that the server set the restriction to validate just this header to be an image type of png or jpeg, but we can bypass it easily by setting our malicious script other than the content of the image file. 
##### `NOTE: `
**Content-Type header**: Tells the server the MIME type of the data that was submitted using this input.

![[lab2.png]]

-----------------------------------------------------------------------
### Lab3: Web shell upload via path traversal
- Some servers if you bypassed the validation and sanitization, it prevent executing any scripts in the same directory of files which uploaded in, and if you tried to access the file to execute it will appear for you as plaintext in the browser like this image:
 
![[plaingtext_lab3.png]]

- So the solution for it is to try to upload this script in another directory other than the default directory for the uploaded files. 
- We will try to use the `directory traversal` vulnerability if it is found to change the directory to another one to execute the script.

![[path traversal_lab3.png]]
- I will use URL encoding to bypass the path.

![[bypassing_lab3.png]]
- Here from the response, we find the file has been uploaded with the path traversal which I did.
- Now I can access the file with this path `/files/rce.php`.
-----------------------------------------------------------------------
### Lab4: Web shell upload via extension blacklist bypass
- Some configured servers add the dangerous file extensions to the blacklist to prevent the end-user from uploading any malicious script.
- When I tried to upload my malicious script to the server, it doesn't allow to me to upload the `.PHP` extension.

![[error_message_lab4.png]]
- But we can bypass it by using some of the lesser-known alternative file extensions to try to upload our file, I tried some extensions from [Hacktricks.](https://book.hacktricks.xyz/pentesting-web/file-upload#file-upload-general-methodology)
- I tried this extension `.phtml` and it has worked.

![[response_lab4.png]]

-----------------------------------------------------------------------
### Lab5: Web shell upload via obfuscated file extension
- There are some techniques it is called **obfuscation techniques** to bypass the blacklists for the extension.
- There are many techniques to use:
	1. Adding multiple extensions: `EX:` exploit.php.jpg  
	2. Adding trailing characters like dot or whitespace: `EX:` exploit.php.
	3. Use the URL encoding or URL double encoding for the dot of the extension.
	4. We can use semicolons or URL encoded Null byte:
	`EX:` ==> exploit.php;.png                  ==> exploit.php%00.jpg 
- There are more techniques you can use to obfuscate the file extension.

- Here I used a null byte to bypass the validation for the filename.

![[bypass_lab5.png]]

-----------------------------------------------------------------------
### Lab6: RCE via polyglot web shell upload
-To increase the security level, The servers make a validation for the content of the file through the file signatures or it is called the **magic numbers(Magic bytes)**.
Check out: [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)
Check out: [File Magic Numbers](https://gist.github.com/leommoore/f9e57ba2aa4bf197ebc5)
#### The first solution:
- At first, I uploaded a normal `png` image, Then I injected my malicious code within the content of the image.
![[puttingMaliciousCode_lab6 1.png]]
- And changed the extension of the image to `.php` to run my script which I put into the content of the image file.

![[changeTheExtension_lab6.png]]
- My image has been uploaded successfully and the server didn't detect my PHP code within the content of the image.
![[response_lab6 1.png]]

#### Another solution:
- We will use the ExifTool to add a comment within the image file it is a PHP code and we will rename the image to a file with the extension of `.php`.

![[exiftool_lab6.png]]
- After uploading the file, which I created, I accessed the uploaded file, and I found the key.
 
![[thekey_lab6.png]]

-----------------------------------------------------------------------
### Lab7: Web shell upload via race condition
- A race condition is a flaw that produces an unexpected result or behavior when timing of actions impact other actions.
- Race conditions may occur when a process is critically or unexpectedly dependent on the sequence or timings of other events. In a web application environment, where multiple requests can be processed at a given time, developers may leave concurrency to be handled by the framework, server, or programming language. 
- This vulnerability arises for a brief period of time under certain specific circumstances. Because the vulnerability exists only for a short time, an attacker “races” to exploit it before the application closes it again. 
- But how does this vulnerability arises here with file upload?
	- We can say some servers don't upload the files to the primary destination to store them in the main filesystem but the server takes them to a temporary directory or is called **sandbox**, it is an environment to test the files before taking them to the main location in the system.
	- The sandbox does some tasks, it makes randomizing for the name of files to avoid that there is a file with the same name on the server where it will cause overwriting with an existing file, then the sandbox makes validation for the file, if it's a safe file, it will be transferred to its destination, if not, the server will delete the file Immediately.
	- Sometimes the websites upload the files to the main filesystem to make its validations and randomizing, and don't use the sandbox to do that so if the file is not safe and didn't pass the validations, the server will remove it. Still, the attacker here can access this malicious script to run it before removing it, so here the race between the server and attacker arose. 

- We will use an extension in Burp Suite called `Turbo Intruder`, we will set the `POST` request of uploading the file in the **noPayload** variable.
![[extension_lab7.png]]
- And then we added our payload with the `GET` request with the path for accessing the file from the server before removing it.
![[payload_lab7.png]]
- Here the first request has succeeded, and it takes a lower time to access the file from the server before it is removed.
- And the second request took more time to make its validations.
- So congrats, we won the race!!
![[attack_done_lab7.png]]















