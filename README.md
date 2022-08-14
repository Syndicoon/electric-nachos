# How to show icecast song titles on web pages - home server setup

I am parking this here for future reference in hopes that it might help someone else. 

THE SETUP: This is for home use only. If you are looking for a commercial setup this is not for you. 
My radio station is not shared out publicly so the traffic levels are really low so my server and bandwidth aren't an issue. 

This setup uses Icecast2, Caddy, an external DNS Domain, a script I found on github, and one server. That's it. 

THE SCRIPT: I am using a script called metadata grabber that I found here: https://github.com/cryptiksouls/icecast-shoutcast-metadata-grabber
The directions are rather sparce and I also found out that the scripts included do not actually work. 
I had to grab the one used on the demo site. That one works BUT HAS LIMITS!

THE PROBLEM: The script works great! But for security reasons and, I suspect limits with JS, the script must be run in the 
same domain as the webpage it is attached to. While on the surface this seems to be easy to overcome, it isn't. 
Caddy is a web server, so is Icecast. Yes, Icecast has a BUILT-IN WEBSERVER. 
To utilize Icecast you have to setup a reverse-proxy which means you are essentially setting up a sub-domain. 
Doing this means you are forced to deal with two domains in your configuration of Icecast and Caddy which conflicts with the one domain limit of the script. 
There is way around this and Icecast itself is the answer. 

CONVENTIONS USED IN THIS TUTORIAL: 
    Main site:  myhomesite.com
    radio site: radio.myhomesite.com
    external IP: 67.x.x.x
    internal IP  192.x.x.x
    
PRE-SETUP: First, to get true https with Caddy, you need to get a domain name. 
It doesn't matter where you get it, but you will need to set up your DNS records once you get one. 

DNS CONFIGURATION: 
    A          @           67.X.X.X
    CNAME       radio       myhomesite.com

Next... download and configure  Caddy. 
Caddy also lacks clear documentation. It makes a lot of assumptions so for complete newbies, it can be a bit confusing. 
This is why I have included a sample Caddyfile. Change it to fit your needs. 
# --------------------------------------------------------------------------
[Caddyfile-sample.txt](https://github.com/Syndicoon/electric-nachos/files/7433973/Caddyfile-sample.txt)

# ---------------------------------------------------------------------------

I am sure some have noticed that there are two domains: the main myhomesite.com and the sub-domain radio.myhomesite.com don't worry, this will work.

WEBSITE SETUP: Create your homesite and set up and index.html page. This will be a landing page that will point to all your radio stations. 

ICECAST SETUP: download and configure Icecast2 and set up your radio stations. For this tutorial, I will use the following radio stations: jazz and rock and use port 8000.  
After you setup your icecast server you should be able to access these two stations using the following links: 

http://192.x.x.x:8000/jazz      https://radio.myhomesite.com/jazz
http://192.x.x.x:8000/rock      https://radio.myhomesite.com/rock

Now comes the solution. Icecast has a built-in web server and we are going to use it to serve up the pages for our radio stations. 
In this example the pages will be called jazz.html and rock.html. 
On UBUNTU SERVER, Icecast serves up web pages from /usr/share/icecast2/web directory so anything put there will be served up by Icecast. Neat huh?
So place the two html files in this directory. 
Place the metadataworker.js script in that directory.
Place this script into the two pages, be sure to change the radio.myhomesite.com link to point to your icecast server. 
# --------------------------------------------------------------
[radio-jazz.html.txt](https://github.com/Syndicoon/electric-nachos/files/7433999/radio-jazz.html.txt)


#---------------------------------------------------------------
    
    The metadataworker.js script - github forced me to save it with the extra .txt so be sure rename it to metadataworker.js before actualy using it.
    
[metadataworker.js.txt](https://github.com/Syndicoon/electric-nachos/files/7434016/metadataworker.js.txt)

#--------------------------------------------------------------------------------------------------------------------
On the landing page, place a link to your radio pages....

 <a href="https://radio.myhomesite.com/jazz.html">Jazz Radio</a>
 <a href="https://radio.myhomesite.com/rock.html">Rock Radio</a>
 
 Now you should be able to access all your radio stations online AND have it show what is playing. 
 For the jazz station you could use this link....
 
 https://radio.myhomesite.com/radio-jazz.html
 
 And that is all! Have fun. 
