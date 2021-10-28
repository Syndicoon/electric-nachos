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
Caddy is a web server, so is Icecast. Yes, Icecast has a BUILT-IN WEBSERRVER. 
To utilize Icecast you have to setup a reverse-proxy which means you are essentially setting up a sub-domain. 
Doing this means you are forced to deal with two domains in your configuration of Icecast and Caddy which conflicts with the one domain limit of the script. 
There is way around this and Icecast itself is the answer. 

CONVENTIONS USED IN THIS TUTORIAL: 
    Main site:  myhomesite.com
    radio site: radio.myhomesite.com
    external IP: 67.x.x.x
    intneral IP  192.x.x.x
    
PRE-SETUP: First, to get true https with Caddy, you need to get a domain name. 
It doesn't matter where you get it, but you will need to set up your DNS records once you get one. 

DNS CONFIGURATION: 
    A          @           67.X.X.X
    CNAME       radio       myhomesite.com

Next... download and configure  Caddy. 
Caddy also lacks clear documentation. It makes a lot of assumptions so for complete newbies, it can be a bit confusing. 
This is why I have included a sample Caddyfile. Change it to fit your needs. 
# --------------------------------------------------------------------------
/home/syndi/TEST/Caddyfile-sample.txt
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
# --------------------------------------------------------------------------------------------------------------------------------
Webpage Sample save as radio-jazz.html

!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8"/>
 <!-- CSS
  ================================================== -->
    <link rel="stylesheet" href="stylesheets/base.css">
    
<title>Jazz Radio</title>

</head>
<body>
    <center>
    <img class="logo" src="images/logo.png" width="450" height="200" alt="logo">
    <p></p>
<audio preload="none" id="aud" controls src="https://radio.myhomesite.com/jazz" type="audio/mpeg">
</audio>
<div><h4></h4></div>
<p></p>
<a href="https://myhomesite.com/index.html"><h4>Radio Landing Page</h4></a>
</center>
<script>
if ('serviceWorker' in navigator) {
  var iceworker = navigator.serviceWorker.register('metadataworker.js')
    .then(function(reg) {
      console.log('Icecast service worker registered');
    }).catch(function(error) {
    console.warn('Error ' + error);
  });
}
var delay = 8000;
navigator.serviceWorker.addEventListener('message', event => {
  if(event.origin != 'https://radio.myhomesite.com'){
    return;
  }
  setTimeout(function(){
    document.querySelector('h4').innerText = event.data.msg.substring(event.data.msg.indexOf("'") + 1,event.data.msg.lastIndexOf("'"));
  },delay);

  console.log(event.data.msg);
});
  document.querySelector('audio').addEventListener('pause', event => {
  navigator.serviceWorker.controller.postMessage('message');
  document.querySelector('audio').src = document.querySelector('audio').src;
})
  function addItem(text) {
  var node = document.createElement("li");
  var textnode = document.createTextNode(text);
  node.appendChild(textnode);
  document.querySelector('ul').appendChild(node);
}
</script>
</body>
</html>

    #-------------------------------------------------------------------------------------------------------------
    
    The metadataworker.js script
    
    let fetchController;
let fetchSignal = null;
self.addEventListener('fetch', event => {
console.log(event);
  if (event.request.destination != 'audio') {
return
}
    let Heads = new Headers({"Icy-Metadata": "1"});
    let stream = new ReadableStream({
      start(controller) {
        let songs = Array();
        let fetchController = new AbortController();
        let signal = fetchController.signal;
        let decoder = new TextDecoder();
        let startFetch = fetch(event.request.url, {signal,headers: Heads});
        function pushStream(response) {
          let metaint = Number(response.headers.get("icy-metaint"));
          let stream = response.body
            let reader = stream.getReader();
          return reader.read().then(function process(result) {
            if (result.done) return;
            let chunk = result.value;
            for (let i = 0; i < chunk.length; i++) {
              songs.push(chunk[i]); 
              if(songs.length > (metaint + 4080)){
                let musicData = Uint8Array.from(songs.splice(0,metaint));
                let metalength = songs.shift() * 16;
                if (metalength > 0){
                  let songtitle = decoder.decode(Uint8Array.from(songs.splice(0,metalength)));
                  self.clients.matchAll().then(function (clients){
                    clients.forEach(function(client){
                      client.postMessage({
                        msg: songtitle
                      });
                    });
                  });
                }
                if(fetchSignal == 1)
                {fetchController.abort();}
                controller.enqueue(musicData);
              }
            }
            return reader.read().then(process);
          });
        }
        startFetch
          .then(response => pushStream(response))
          .then(() => controller.close())
          .catch(function(e) {
            console.log('Connection to stream cancelled ' + e);
            fetchSignal = 0;
            sendMsg('Dropped connection');
          });
      }
    });
    event.respondWith(new Response(stream, {
      headers: {'Content-Type': 'audio/mpeg'}
    }));
});
self.addEventListener('install', event => {
  self.skipWaiting();
});
self.addEventListener('activate', event => {
  clients.claim();
});
self.addEventListener('message', event => {
  if(event.origin != 'https://radio.myhomesite.com'){
    return;
  }
  fetchSignal = 1;
});
function sendMsg(msg){
  self.clients.matchAll().then(function (clients){
    clients.forEach(function(client){
      client.postMessage({
        msg: msg
      });
    });
  });
}
#--------------------------------------------------------------------------------------------------------------------
On the landing page, place a link to your radio pages....

 <a href="https://radio.myhomesite.com:8000/jazz.html">Jazz Radio</a>
 <a href="https://radio.myhomesite.com:8000/rock.html">Rock Radio</a>
 
 Now you should be able to access all your radio stations online AND have it show what is playing. 
 For the jazz station you could use this link....
 
 https://radio.myhomesite.com/radio-jazz.html
 
 And that is all! Have fun. 
