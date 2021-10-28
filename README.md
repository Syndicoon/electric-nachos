# How to show icecast song titles on web pages - home server setup

I am parking this here for future reference in hopes that it might help someone else. 
THE SETUP: This is for home use only. If you are looking for a commercial setup this is not for you. My radio station is not shared out publicly so the traffic levels are really low so my server and bandwidth aren't an issue. 

This setup uses Icecast2, Caddy, an external DNS Domain, a script I found on github, and one server. That's it. 

THE SCRIPT: I am using a script called metadata grabber that I found here: https://github.com/cryptiksouls/icecast-shoutcast-metadata-grabber
The directions are rather sparce and I also found out that the scripts included do not actually work. 
I had to grab the one used on the demo site. That one works BUT HAS LIMITS!

THE PROBLEM: The script works great! But for security reasons and, I suspect limits with JS, the script must be run in the same domain as the webpage it is attached to. While on the surface this seems to be easy to overcome, it isn't. Caddy is a web server, so is Icecast. Yes, Icecast has a BUILT-IN WEBSERRVER. To utilize Icecast you have to setup a reverse-proxy which means you are essentially setting up a sub-domain. Doing this means you are forced to deal with two domains in your configuration of Icecast and Caddy which conflicts with the one domain limit of the script. There is way around this and Icecast itself is the answer. 

CONVENTIONS USED IN THIS TUTORIAL: 
    Main site:  myhomesite.com
    radio site: radio.myhomesite.com
    external IP: 67.x.x.x
    intneral IP  192.x.x.x
    
PRE-SETUP: First, to get true https with Caddy, you need to get a domain name. It doesn't matter where you get it, but you will need to set up your DNS records once you get one.  
DNS: 
