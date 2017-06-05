Lately I’ve been giving a lot of talks about Progressive Web Apps. Towards the end of the talks there is normally a Q & A section where the audience asks questions or proposes ideas. Often during these Q & A sessions, I get asked really useful questions that are worth sharing with a wider audience.
 
In the article, I’ve put together a list of some of the questions I regularly get asked about Progressive Web Apps and Service Workers and tried to include the most accurate answers I could. Some of these questions might seem obvious, some not so obvious - but overall I hope that you find useful. 
 
 ![Big list of Progressive Web App tips & tricks](https://307a6ed092846b809be7-9cfa4cf7c673a59966ad8296f4c88804.ssl.cf3.rackcdn.com/pwa-tips-tricks/list-of-progressive-web-apps-tips.jpg)

So here goes...in no particular order, a list of helpful tips, tricks and gotchas that could help you when you build your next Progressive Web App!
 
## How do I tell how many users are using the Add to Homescreen (A2HS) functionality on my site?
 
When the A2HS banner is shown, you can tap into the *beforeinstallprompt* event to determine the choice that the user made when they were presented with the banner.
 
The code below shows this in action:

    window.addEventListener('beforeinstallprompt', function(event) {
	  event.userChoice.then(function(result) {						
 
    if(result.outcome == 'dismissed') {							
      // They dismissed, send to analytics
    }
    else {
      // User accepted! Send to analytics
    }
  });
});
 
Using the code above, you can determine if the user dismissed the banner or if they decided to add your web app to their home screen. Using a web analytics package, you’ll be able to track their choice and hopefully determine if this functionality is beneficial to your users.
 
Another sneaky technique is to set the start url in your manifest.json file to include a querystring indicating that it was opened via the home screen of a user’s device. 
 
For example, you could update the start_url property on the manifest.json:
 
{
	name: ‘Progressive Beer’,
	short_name: ‘beer’
	**start_url: ‘index.html?start=a2hs’**
}
 
This updated start URL including querystring would allow your web analytics tools to track usage and determine how many users are arriving on your PWA via the icon on the homescreen of their device.
 
## The Add to Homescreen banner doesn’t make sense for my website. How do I disable this functionality? I want to hide it!
 
Using a sneaky bit of code, you can actually override the default functionality and cause the browser to ignore the Add to Homescreen (A2HS) banner. By adding the following code to your page, you’ll be able to tap into the banner prompt event and override the default behaviour.
 
	window.addEventListener('beforeinstallprompt', function(e) {
	  e.preventDefault();
	  return false;
	});
 
Depending on the type of your web app it might not make sense to show this prompt; perhaps your site covers sensitive topics or even a short lived event that might be more annoying to the user than helpful.
 
## My Add to Homescreen (A2HS) functionality doesn’t seem to be working - help!?
 
Okay, so you’ve correctly added a manifest.json file to your website and referenced it in the head tag of your HTML like so:
 
	<link rel="manifest" href="manifest.json"> 
 
However, if for some reason you still aren’t seeing the Add to Homescreen banner appear at the bottom of the page, there are a few things you might want to check. Firstly, for the A2HS banner to appear, a few criteria need to be met:

 - Your site needs to be running over HTTPS, have a valid manifest file (with a start url and icon) and have an active service worker file.
 - The user also has to have visited your site at least twice within the last five minutes. The reason for this is that if the banner appeared too many times it could be very spammy for the user. Those “install our native app” banners are bad enough on some websites already!
 
## I’m adding resources into cache with code in my service worker, but the cache isn’t updating when I change the file. I can still see the older version of my files even after I refresh the page - why?
 
Start by checking the Developer Tools to determine what files are actually being cached. If you open up Chrome Dev Tools and head over to the **Application** tab, you’ll be able to determine what files are actually in the cache.
 
![Service Worker cache](https://307a6ed092846b809be7-9cfa4cf7c673a59966ad8296f4c88804.ssl.cf3.rackcdn.com/pwa-tips-tricks/service-worker-cache.jpg)
 
If you need to ensure that files are always updated when you make changes, you might want to consider versioning your files and renaming them. This way you can ensure that each file change is guaranteed to be cached correctly. For example, using [file versioning](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#invalidating_and_updating_cached_responses) we might reference a JavaScript file in our HTML like so:
 
	<script src=”/js/main-v2.js”>
 
Each time the file changes, you bump the version which results in a fresh download.
 
Another technique to ensure that you always get fresh code is to actually delete the current cached entries when the service worker activates after updating. By tapping into the activate event during the service worker lifecycle, you can clear the cache. I recommend checking out [this link](https://googlechrome.github.io/samples/service-worker/custom-offline-page/) as a guidance.
 
Depending on how your PWA has been built you might want to choose the best strategy to suit your needs.
 
## How do I unit test my Service Worker code?
 
Testing your Service Worker code can be tricky, but fear not, Matt Gaunt wrote an excellent article about the ins and outs of testing Service Workers. I recommend reading [this article](https://medium.com/dev-channel/testing-service-workers-318d7b016b19) on Medium for more information. 
 
## I’m not sure if my manifest.json file is working correctly - how do I test it?
 
One of my favourite tools for validating manifest files is [manifest-validator.appspot.com](http://manifest-validator.appspot.com). The web app manifest validator checks the file and uses the W3C specification in order to determine if it is valid. If you are having trouble understanding why your web app manifest doesn’t seem right, the tool will provide you with feedback about which character caused an issue and also different reasons that could be causing the issue.
 
![Manifest Validator](https://307a6ed092846b809be7-9cfa4cf7c673a59966ad8296f4c88804.ssl.cf3.rackcdn.com/pwa-tips-tricks/validator.png)
 
However, if you do struggle with creating these files and find that you make mistakes here and there, I would recommend using a Manifest file generator. This [Github repo](https://brucelawson.github.io/manifest/) created by Bruce Lawson has a handy tool that you simply input your details and it will spit out a fully created web manifest file for you.
 
## How often does the Service Worker file update?
 
Every time you navigate to a new page that's under a service worker's scope, Chrome will make a standard HTTP request for the JavaScript resource that was passed into the *navigator.serviceWorker.register()* call. By default, this HTTP request will obey standard HTTP cache directives, but if the service worker file is older than 24 hours, it will always go to the network and fetch a fresh version of your service worker file. This is to ensure that developers don't accidentally roll out a "broken" or buggy service worker file that gets stuck in the browser forever. Think of it as a safety switch for your service worker file.
 
For more information, I recommend reading the following [article on Stack Overflow](https://stackoverflow.com/questions/38843970/service-worker-javascript-update-frequency-every-24-hours/38854905#38854905) where Google’s Jeff Posnick goes into more detail.
  
## My Service Worker file is throwing an error, but I’m not sure what’s wrong. How do I debug it?
 
Without a doubt, the easiest way to debug your Service Worker code is to use the Developer Tools in your browser. If you open the Developer Tools in Google Chrome and head to the Scripts tab, you can set a breakpoint to help you debug the error.

![Service Worker Debugging](https://307a6ed092846b809be7-9cfa4cf7c673a59966ad8296f4c88804.ssl.cf3.rackcdn.com/pwa-tips-tricks/debugging-service-worker.jpg)
 
With the breakpoint set in the Developer Tools, your code will pause when it reaches this point and allow you to see exactly how your code logic is executing. Mastering the Dev Tools is a great step forward in becoming a better developer. While many of the browser vendors will offer tutorials for their developer tools, my personal favourite is Chrome’s Developer Tools. If you’d like to learn more, I recommend checking out the following link.
 
## I’ve tried everything, but for some crazy reason my Service Worker logic never seems to execute - help!?
 
It’s worth double checking your Developer Tools to see if you may have enabled a setting incorrectly. For example, if you enable ‘Bypass for network’, your Service Worker logic will be ignored and instead fetch resources via the network instead of cache. 
 
![Chrome Developer Tools -Bypass for network](https://307a6ed092846b809be7-9cfa4cf7c673a59966ad8296f4c88804.ssl.cf3.rackcdn.com/pwa-tips-tricks/bypass-for-network.jpg)
 
In the same way, you might want to check that you don’t have the other settings enabled when you don’t need them. For example, Offline and Update on reload - I have been left scratching my head many times trying to figure out why my code wasn’t working, only to find out that I’d forgotten to disable one of these settings - d’oh!
 
## If a user has installed my web app to their home screen, but they clear their cache in Chrome, does my sites cached resources get cleared too?
 
Yes, because the Progressive Web App experience is powered by Chrome, the storage is currently shared. If a user clears their Chrome cache, your PWA will clear it’s storage too.
 
If you’d like to learn more about the improved A2HS functionality in Chrome, I highly recommend reading the [following article](https://developers.google.com/web/updates/2017/02/improved-add-to-home-screen#will_my_installed_sites_storage_be_cleared_if_the_user_clears_chromes_cache).
 
## How much memory can my Progressive Web App use on a user’s device?
 
The honest answer is that it really depends on your device and storage conditions. Like all browser storage, the browser is free to throw it away if the device comes under storage pressure. 
 
If you’d like to determine how much storage you have and how much you’ve used up, the following code might help.
 
	navigator.storageQuota.queryInfo("temporary").then(function(info) { 
	console.log(info.quota); // The total amount in bytes 
	console.log(info.usage); // How much data you’ve used so far in bytes 
	});
 
The above code might not work on all browsers, but will definitely point you in the right direction. There is a great [answer on Stack Overflow](https://stackoverflow.com/questions/35242869/what-is-the-storage-limit-for-a-service-worker) that explains this in more detail.
 
## My cached resources seem to expire every so often, how do I ensure that they stay cached permanently?
 
When storage space on a device is running low, the browser will automatically clear storage to make more available space. While this ensures that your users device is always runs smoothly, it can make building a truly offline experience for the web a little tougher.
 
Fear not! There is a way. If you’d like to make cache storage more persistent, you can ask for it explicitly using a bit of code.
 
	if (navigator.storage && navigator.storage.persist)
	  navigator.storage.persist().then(granted => {
	    if (granted)
	      alert("Storage will persist and not be cleared");
	    else
	      alert("Storage won’t persist and may be cleared");
	  });
 
There a few criteria that need to be met before persistent storage is granted, but if you’d like to learn more about this great feature, I recommend reading [this article](https://developers.google.com/web/updates/2016/06/persistent-storage) for more information.
 
## I’ve added code to handle push notifications in my service worker file, but how can I test them quickly without writing server side code?
 
If you are looking for a quick way to simulate push events within your web app, the Developer Tools provide a quick and easy way to simulate them in action.
 
![Web push notifications](https://307a6ed092846b809be7-9cfa4cf7c673a59966ad8296f4c88804.ssl.cf3.rackcdn.com/pwa-tips-tricks/simluate-push.jpg)
 
## I’ve built an offline web app but now I can’t see how users are using my it. How do I track usage?
 
Without a doubt, one of coolest libraries to appear lately has to be the **Offline Google Analytics** package. Using a bit of clever service worker magic, the library will queue up any analytics requests while the user is offline, and as soon as you regain a connection, it will then send the queued requests through to the analytics server.
 
In order to start using the library, you simply need to include it in your service worker file using the code below.
 
	importScripts('../build/offline-google-analytics-import.js');					
	 
	goog.offlineGoogleAnalytics.initialize();							
	 
	self.addEventListener('install', () => self.skipWaiting());					
	self.addEventListener('activate', () => self.clients.claim());
 
By including this code in your service worker file, the library will track any actions made by the user while offline, queue them, and then send them in order once the user regains connectivity. Very cool stuff!
 
## How do I deal with query string parameters and caching?
 
When a Service Worker checks for a cached response, it uses a request URL as the key. By default, the request URL must exactly match the URL used to store the cached response, including any query parameters in the search portion of the URL.
 
For example, if you make a request for a URL with a querystring and it may previously matched, you might find that it misses the next time because the query string differs slightly. To ignore query strings when you check the cache, use **ignoreSearch** attribute and set the value to true.
 
The code below gives you an idea of this in action:
 
	self.addEventListener('fetch', function(event) {
	  event.respondWith(
	    caches.match(event.request, {
	      **ignoreSearch: true**
	    }).then(function(response) {
	      return response || fetch(event.request);
	    })
	  );
	});
 
## Summary
There have been many moments when I’ve found myself pulling my hair out trying to figure out the different nuances to service workers, only to find that the solution was simpler than it seemed - I hope that you found this document helpful. 
 
In order to keep knowledge sharing I’ve created a Github repo with all of these questions. If you have any of your own tips that you’d like to add, or something doesn’t seem accurate, please send a PR and together we can grow this document - you never know how it could help others!
