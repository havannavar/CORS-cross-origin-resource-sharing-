# CORS-cross-origin-resource-sharing
Setting up of Apache cross origin resource in virtual-host.conf

Once in a life time of your product development need to make a cross-domain request from Javascript, this is something the browser very much dislikes. 
Not a expert of CROS, tried searching in google, but really unsuccessful.

I faced multiple issues before finding a complete solution.

Here is a brief solution:

    Set up was in DigitalOcean  - Ubuntu-14.10. and Apache-2.4.10

I started off with just adding the Access-Control-Allow-Origin header in my Apache configuration, 
thinking that it’ll solve my problems. I also decided to set it on wildcard, 
allowing anything to request resources.
There is no 
    .htaccess file in Apache-2.4.10, So add the below configurations only in VirtualHost.conf


# In my virtualhost config
    Header set Access-Control-Allow-Origin "*" 

Since header module was missing so have to install 
    sudo a2enmod headers

and restarted the server, still no success, got the same error “OPTIONS Method Not Allowed” 
Then changed it to specific URL 
    Header set Access-Control-Allow-Origin "http://test.example.com"

Thought to add allow OPTIONS request in the python code, but that is really bad practice.

Digged more and understood that this is a “pre-flight” request, basically asking the server for the CORS headers, but without data.
Found that for every OPTION call we should send success response

so added 

    RewriteEngine On
    RewriteCond %{REQUEST_METHOD} OPTIONS
    RewriteRule ^(.*)$ $1 [R=200,L]

again rewrite module was missing, so installed that also

    sudo a2enmod rewrite. and restarted the server.

This time it was new error saying - credintials missing and added

    Header always set Access-Control-Allow-Credentials "true"

Hurry !! This time was successful.

So my final setup of headers and rewrite engine is :

				# Always set these headers.
				Header always set Access-Control-Allow-Origin "http://test.example.com"
				Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS, DELETE, PUT"
				Header always set Access-Control-Max-Age "1000"
				Header always set Access-Control-Allow-Credentials "true"
				Header always set Access-Control-Allow-Headers "x-requested-with, Content-Type, origin, authorization, accept, client-security-token"
				 
				# Added a rewrite to respond with a 200 SUCCESS on every OPTIONS request.
				RewriteEngine On
				RewriteCond %{REQUEST_METHOD} OPTIONS
				RewriteRule ^(.*)$ $1 [R=200,L]



Thumb rule dont use this for Production as allowing Allow credenttials is security threat.


