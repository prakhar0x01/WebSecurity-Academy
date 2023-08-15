## BROKEN ACCESS CONTROL and IDORS
### [RESOURCES](https://portswigger.net/web-security/access-control)
---
## What is Access Control ??
   - ##### Access control (or authorization) is the application of constraints on who (or what) can perform attempted actions or access resources that they have requested. In the context of web applications, access control is dependent on authentication and session management:

   - ##### Authentication :  identifies the user and confirms that they are who they say they are.

   - ##### Session management : identifies which subsequent HTTP requests are being made by that same user.

   - ##### Access control : determines whether the user is allowed to carry out the action that they are attempting to perform.

## Types of Access Control Vulnerabilities -
   
   1) Vertical access controls
   2) Horizontal access controls
   3) Context-dependent access controls

### Vertical access controls 

1) In this Control different types of users have access to different application functions.

   - For example : An administrator might be able to modify or delete any user's account, while an ordinary user has no access to these actions. 

### Horizontal access controls 

1) Horizontal access controls are mechanisms that restrict access to resources to the users who are specifically allowed to access those resources.

	- For example : A banking application will allow a user to view transactions and make payments from their own accounts, but not the accounts of any other user.
		              
	- You can also take the examples of "Subscription" based functionality.     


### Context-dependent access controls 

1) Context-dependent access controls restrict access to functionality and resources based upon the state of the application or the user's interaction with it.

   Context-dependent access controls prevent a user performing actions in the wrong order. 

	For example : A retail website might prevent users from modifying the contents of their shopping cart after they have made payment.

---

### ############## PRACTICAL TESTING ################ ###

#####
Make two accounts on the application
1 - Hacker
2 - Victim
#####

### 1) Unprotected admin functionality

   1) Always find `/admin` locations on the site
   2) Check `robots.txt`, do directory bruteforcing
   3) Sometimes default OPEN dashboards and maybe easily logged-in by default username:password
---
### 2) Unprotected admin functionality with unpredictable URL

   1) Check the `source-code` of the page.(By view source)
   2) Check `Javascript files` or hidden files, It might be possible that they hide the panel with complex name.
---
### 3) User role controlled by request parameter

   1) Always Check request headers and parameters.
   2) Consider an application verify the users whether he/she is admin or not , if he/she is admin , then redirected to admin dashboard other-wise 404 page.
    
          `GET /admin HTTP/1.1
          Host: subdomain.target.com
		  Cookie: [Admin=false]; session=UmIrCtXy1e2YtCMBUCxLMfSulfKxDV5B
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
	      Connection: close`
   
   3) From the above request, we can easily manipulate `Admin` parameter by changing it from `False` to `True`.
   ---
### 4) User role can be modified in user profile

   1) Sometimes application used to set some specific `role_id,id,user_id` in request parameter.These parameter are somewhere uses in application might be possible in email/password changing functionality.
   2) Consider an application set a rule for `role_id` that if any person have a `role_id: 1` , then it is an `admin` otherwise an ordinary user.
       
          `POST /my-account/change-email HTTP/1.1
          Host: subdomain.target.com
		  Cookie: session=avTFqTxamGFounYZ6MKcirCw32H2cJD3
		  Content-Length: 28
		  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
		  Origin: https://subdomain.target.com
		  Referer: https://subdomain.target.com?id=hacker
          ...
		  {
			 "email":"hacker@hacker.ca",
			"roleid":10                               // It can be manipulate .
          }`

   3) We can easily manipulate the above request by changing the `roleid` to be `1`, and if you don't know the `role_id`, try to bruteforce it with certain limit.
---
### 5) User ID controlled by request parameter

   1) Check each and every parameter and try to manipulate it with others.
   2) Consider an application has a user account dashboard, so everytime when user click on `my account` , there is a request parameter i..e `/myaccount/?name=user_name` , 
   3) An attacker could manipulate the `?name=` or anyother parameter to someone else.  
                  
           `GET /my-account?id=VICTIM_NAME HTTP/1.1
			Host: subdomain.target.com
			Cookie: session=lJ52q5GoqbgcnbZB0epyRJBWKyMdEdOR
			User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
			Referer: https://subdomain.target.com/my-account
			Connection: close`
---
### 6) User ID controlled by request parameter, with unpredictable user IDs

   1) Always check URL parameters, sometimes applications uses encoded string/hash of the name/id or some kind of key,
   2) Try to change it with others
   Eg.Consider an application has a URL parameter like : `https://target.com/myaccount/?userid=aGFja2VyCg==`

      `aGFja2VyCg== : 'hacker'`

   3) So we can easily changed it to victim's.

      `dmljdGltCg== : victim`
---
### 7) User ID controlled by request parameter with data leakage in redirect
 
   1) Check the redirected responses,it might be possible that they leaked some info.
   2) Consider from above case, now application strict the security and instead of response, it redirect to login page on incorrect/invalid input.
   3) Eg. the request look like this : 
         
           `GET /my-account?id=attacker HTTP/1.1
			Host: abc.target.com
			Cookie: session=Yz4XhRIBaFMrHLHfmcNkNBHAKuJwB2QG
			User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
			Connection: close`
      
   4) And from above request if we change the `id` parameter to `victim`. We will be redirected and the response look like this...

           `HTTP/1.1 302 Found
			Location: /login
			Content-Type: text/html; charset=utf-8
			Cache-Control: no-cache
			Connection: close
			Content-Length: 3395	
          ...
          <!DOCTYPE html>
		   <h1>
		   	My Account
		   </h1>
			<div id=account-content>
				<p>Your username is: Victim</p>
			</div> 	
			<div>
				Your API Key is: fazpGfmIjnXEk9W34gDwsXVtuiNNAnei
			</div>`
---
### 8) User ID controlled by request parameter with password disclosure

   1) Again ..!! , Change the request parameter to anyone else and see the difference.
   2) Consider an application has `Disclosed name:password` in `/my_account` page , so if an attacker change the name in request parameter to other's name , it's an full account takeover.
No examples, it's all about changing or manipulate things.
 ---
 ---
## Insecure direct object references (IDOR) 

   - Insecure direct object references (IDOR) are a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly. 

#### IDOR vulnerability with direct reference to database objects 

   - Consider a website that uses the following URL to access the customer account page, by retrieving information from the back-end database:

	`https://insecure-website.com/customer_account?customer_number=132355`

- Here, the customer number is used directly as a record index in queries that are performed on the back-end database. If no other controls are in place, an attacker can simply modify the `customer_number` value, bypassing access controls to view the records of other customers. This is an example of an IDOR vulnerability leading to horizontal privilege escalation.

   Again ..!! it could be anything , `id,key,name,ecoded_String etc`. in place of `customer_number`.

#### IDOR vulnerability with direct reference to static files

   - Consider a website might save transcripts/files to disk using an incrementing filename, and allow users to retrieve these by visiting a URL like the following:

	`https://insecure-website.com/static/12144.txt`

- In this situation, an attacker can simply modify the filename to retrieve a file created by another user and potentially obtain user credentials and other sensitive data.
---
### 1) URL-based access control can be circumvented

   1) Sometimes applications have very strict security rules.
   2) Consider an website has an unauthenticated admin panel at `/admin`, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the `X-Original-URL` header.
   3) So basically we can add `X-Original-URL:` in request header to bypass the security control.

   	       `GET / HTTP/1.1
			Host: abc.target.com
			Cookie: session=yhIYaQ5gNfAu40Pye5tRs1Wnhb5bgGRo
            [X-Original-URL: /admin/delete]
			User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
			Referer: https://abc.target.com/
			Connection: close
			Content-Length: 2`
---
### 2) Method-based access control can be circumvented

   1) Sometimes website use HTTP-methods for validation. So it been a good practice to always check or change the HTTP request methods.
   2) Consider an application has functionality for a subscription, so if a free user cannot upgrade himself in `POST` request method , It might be possible that he get the subscription by changing to the other request methods(`GET,PUT..etc`).
---
### - Referer-based access control

   1) If there is a `Referer` Header in request, like : `https://abc.target.com/`
   2) Change it to --> `Referer: https://abc.target.com/admin`

---
---
