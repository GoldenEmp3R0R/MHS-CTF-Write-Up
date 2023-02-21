This is a write up I am covering for the Web challenge from MHS CTF on Valentines Day.

Web solution step by step
* Discover hidden page that prevents web crawlers from checking on
* Discover key written in plain text in Inspect Element
* Flask Session Vulnerability


START OF CAMPAIGN: 

![[Pasted image 20230220225800.png]]

This is the main page of the website, the first thing to notice is there are 4 links on the same page javascript that move your scroll bar to different types of chocolates offered.

On this page, I went to see if the developers left any comments in the page source that they forgot to remove or hide.


In the page source we discover there is a hyperlink reference to a hidden directory on the website called /hidden-page


![[Pasted image 20230220230130.png]]






Following this, I then plugged /hidden-page at the end of chocolates-mhsctf.0xmmalik.repl.co and arrived at a new page. 


![[Pasted image 20230220231106.png]]

It warns you that you need a key and the text is clickable, however it only redirects you back to the original page -> chocolates-mhsctf.0xmmalik.repl.co.

I had no leads as to what the key may be, I assumed I needed to insert a payload that the server would acknowledge, and so I went to Inspect Element to see if I could find any interesting scripts or files that may deliver a hint as to what the key or payload may be. 





I went to the sources for chcolates-mhsctf.0xmmalik.repl.co and found there was one folder called *static*, I viewed inside static a file called style.css which revealed a written key with a query string. 

![[Pasted image 20230220231459.png]]


Copying the whole key to hidden-page led to a new page called now "Cindy's Chocolates" instead of "Taylors Chocolates"



![[Pasted image 20230220231848.png]]




On this page, there is a link to click to verify you are an admin, however when you click the link, it has an image accessing a resource with a 404 error. 

![[Pasted image 20230220232119.png]]



When I attempted to modify the URL with "thedarkestchocolate" the string needed to pass authorization on the hidden page, I got this page :


![[Pasted image 20230220232643.png]]


When you click the button it redirects you to the main page once again, so this was the beginning of a rabbit hole.


I went back to https://chocolates-mhsctf.0xmmalik.repl.co/hidden-page?key=thedarkestchocolate
to inspect the page further to see if there was a way I can manipulate the authorization of the user, and looked for cookies on the page. 


I inspected the cookie and came across something very interesting was not only a **session** header, but the cookie data contained a **JWT format**, indicating this was a web token and not a cookie passing verification before clicking the link. 


![[Pasted image 20230220233611.png]]


The format of JWT token has a **header**, a **payload** and a **signature** signed broken into three sections, and so I pasted the JWT in jwt.io to decode the web token so I can understand its values to possibly pass it for authorization. 


![[Pasted image 20230220234535.png]]



The format of the cookie was an HMACSHA256 which was even more interesting as HMACSHA256's signatures are signed with a secret key, so I knew I had to decode and discover the key used to sign the JWT token so I can then resign it with a payload that allowed the admin to be set to true for the session .

At first I wanted to try to just edit the admin and set it to true to test if the website would not check for the secret key to pass verification in the JWT token.


![[Pasted image 20230221001326.png]]
 I got an internal server error from doing this which told me that the webserver cannot handle or read my request which meant it has to verify the session as the secret key is being checked for verification in the payload and that required me to design the JWT token. 



I used a tool called flask-unsign I found from pypi.org that had a particular interesting function that suited my exactly purposes and allowed me to bruteforce the cookie I had by unsigning the cookie.


![[Pasted image 20230220234439.png]]

I passed my JWT token-> *eyJhZG1pbiI6ImZhbHNlIiwidmlzaXRfdGltZSI6IjIwMjMtMDItMjEgMDQ6MTE6MzAuNTgwNjUyIn0.Y_RJ2g.Ej_YDEZeliHU8sUgKnFLpC54CSI*   into a file as one of the switches required an input file.

Command breakdown:
**flask-unsign** : command/tool
**--unsign** : switch to unsign the cookie or JWT
**--cookie** : switch to pass a cookie/jwt value
< : indicates stdin and reads input from a file  "cookie.txt" with the cookie/jwt data. 
**--wordlist** : switch that allows to pass in a wordlist file
The word list file using is the default rockyou.txt that comes built in kali.
**--no-literal-eval** : switch allows to use a wordlist / secret keys without having to wrap each line in quotes as Python strings request. This is done to prevent the data being presented in an arbitrary binary format that can differ results


![[Pasted image 20230220235833.png]]

After running the command we find find the value of BATMAN wrapped in quotes to be our secret key.


![[Pasted image 20230221000040.png]]

After signing the cookie now with the secret key 'BATMAN' , and the existing cookie values from the JWT, I now hopefully would be able to get an valid  session that would allow me to set the session  admin to true and pass verification.


![[Pasted image 20230221000438.png]]

I opened up burpsuite and intercepted web traffic after having clicked the "Click here to verify that you are an admin." After this I changed the session= parameter in the Cookie header to the value of the new JWT token I generated in flask-unsign and forwarded the request to the web page.


![[Pasted image 20230221000641.png]]

The flag was found on this page after passing verification successfully, and the JWT token payload was successfully passed and solved the challenge. 

flag: valentine{1ts_jus7_100%_cacao}


Mistakes in my




