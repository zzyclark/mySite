---
published: true
title: Build Your Own Cache Busting Extension
layout: post
author: Clark
category: articles
tags:
- Nodejs
- Javascript
description: Customized Cache Busting Plugin for Expressjs
comments: true
---
Every time we release our new version web application, we always have a caching problem. We may change some code in our Javascript file, while the name of the will not change. When user try to use our new release, they may refer the Javascript cached in their browser, since the new one share the same name with the one cached. We can solve this by changing cache expiration in the **Cache-Control** header, but this is not the best solution. User may request the same file again and again, even there is no new version release. These requests will cause much useless bandwidth cost and seriously reduce Page loading performance.

To solve this, we may use **Cache-Buster**. A Cache-Buster is a unique piece of code that prevents a browser from reusing files that cached, or saved to a temporary memory file. Cache-Buster doesn't stop a browser from caching the file, but just prevents is from reusing it.

Thought this article, I will show how to build our own cache busting extension. We use Node.js to build our server. Express framework will be used in fasten our development, it has already provided a mature dev process for web application in Node.js. During this article, lots of express technologies will be used. I expect you have some knowledge about express, or you can refer to the [express tutorial](http://expressjs.com/) first which will only cost you 5 minutes. Our extension will all bing with Express, but you can customize it to any other framework or technologies, like Java.

Main logic of our extension is to give a different name for all the static files in each new release. New name can refer to our release number or deployment time. Here we use both to rename.

**Original file:**
{% highlight HTML%}
<script src="/angular/sha256.js" type="text/javascript"></script>
{% endhighlight %}

**Renamed file:**
{% highlight HTML%}
<script src="/angular/sha256.min.js?v=0.1.27-1463110454476" type="text/javascript"></script>
{% endhighlight %}

New renamed file includes version information **_0.1.27_** and unix timestamp **_1463110454476_** which indicate our deployment time. Each of our new release, all the static file will have a url link, this will force our user to request the static file from the server again, but not from browser cache.

By using Express, we can customize some express function which can be called from jade directly. Inside our extension **omni-cache-busted.js**, we can declare our function handler as below. We can use **cacheBust** function in our jade template now.

{% highlight JAVASCRIPT%}
cacheBust.handler = function handler (app, options) {
	app.locals.cacheBust = cacheBust(options);
};
{% endhighlight %}

In the cacheBust function, we can write our own logic after get the information of each static file name. You can refer more in my github.

In our express **app.js**, we will initialize our extension in this way.
{% highlight JAVASCRIPT%}
...
var cacheBust = require('omni-cache-busted');
var app = express();
...

...
cacheBust.handler(app);
...
{% endhighlight %}

Finally, we only need to use cacheBust function in jade template to request our static files.

{% highlight JAVASCRIPT%}
...
!= cacheBust('/angular/sha256.min.js')
...
{% endhighlight %}

Static files will be request in the following way:

![Chrome Console]({{ site.url }}/images/Cache-Busting.png)
