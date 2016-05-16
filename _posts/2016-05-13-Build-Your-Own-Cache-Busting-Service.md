---
published: true
title: Build Your Own Cache Busting Service
layout: post
author: Clark
category: articles
tags:
- Nodejs
- Javascript
description: Customized Cache Busting Plugin for Expressjs
comments: true
---
Every time we release our new version web application, we always have a caching problem. We may change some code in our static file, for example, javascript, css, image file. When user try to use our new release, they may refer the static file cached in their browser, since we haven't change the static file name. We can solve this problem by changing cache expiration in the **Cache-Control** header, but this is not the best solution. User may request the same file again and again, even there is no new version release. These requests will cause much useless bandwidth cost and seriously reduce Page loading performance.

Another solution is **Cache-Buster**. Cache-Buster is a unique piece of code that prevents a browser from reusing files that cached, or saved to a temporary memory file. Cache-Buster doesn't stop a browser from caching the file, but just prevents is from reusing it.

Through this article, I will show how to build our own cache busting service. We use Node.js to build our web server. Express framework will be used to accelerate our development. Express has already provided a mature dev process for web application in Node.js. In this article, lots of express technologies will be used. I expect you have some knowledge about express, or you can refer to the [express tutorial](http://expressjs.com/) first which will only cost you 5 minutes. Our extension will all bind with Express, but you can customize it to any other framework or technologies, like Java.

Main algorithm of our extension is to give a different request url for all the static files in each new release. New url may refer to our release version number or deployment time. Here we use both to rename.

**Original file:**
{% highlight HTML%}
<script src="/angular/sha256.js" type="text/javascript"></script>
{% endhighlight %}

**Renamed file:**
{% highlight HTML%}
<script src="/angular/sha256.min.js?v=0.1.27-1463110454476" type="text/javascript"></script>
{% endhighlight %}

The renamed file includes version information **_0.1.27_** and a unix timestamp **_1463110454476_** which indicate our deployment time. In each of our new release, all the static file will have a new url link, this will force our user to request the static file from the server again, but not from browser cache.

By using express, we can customize some express function which can be called from jade directly. Inside our extension **omni-cache-busted.js**, we can declare our function handler as below. We can use **cacheBust** function in our jade template now.

{% highlight JAVASCRIPT%}
cacheBust.handler = function handler (app, options) {
	app.locals.cacheBust = cacheBust(options);
};
{% endhighlight %}

In the cacheBust function, we can write our own logic after get the information of each static file name. You can refer more in my [github](https://github.com/zzyclark/omni-cache-busted).

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


All the code you can find in my [github](https://github.com/zzyclark/omni-cache-busted). If you also Node.js and Express, you can directly use my npm package [omni-cache-busted](https://www.npmjs.com/package/omni-cache-busted). To use this as npm package, please add this in your **package.json** file.

{% highlight JAVASCRIPT%}
...
"dependencies": {
  ...
  "omni-cache-busted": "~1.0.2"
  ...
}
...
{% endhighlight %}
