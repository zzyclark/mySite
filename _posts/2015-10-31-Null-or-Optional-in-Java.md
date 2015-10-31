---
published: true
title: Null or Optional in Java
layout: post
author: Clark
category: articles
tags:
- Java
description: a research on Optional in Java, a Null replacement
comments: true
---
NullPointerException is the most annoying Exception which I hate the most. However, it's also the Exception that has most appearance. For almost all the function parameter passed in, i need to check whether this Object is Null or not. If the Object is null, all the following code will fail. Unfortunately, it's a runtime Exception, until the app is really running, we can't find the error location. Sometimes, even we found the null object cause this error, we still need long time to find the reason which cause this object is null.

To avoid this, we usually do null like this:

{% highlight java linenos %}
 if ( null != objCheck) {
 // your code after null check
 }
{% endhighlight %}

So in our code, we can thousand of code check like above in order to avoid worse condition will happen. While this is not the only way to check this. In Java 8, we have Optional to do the same thing and even better.

>Optional is a container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.

Optional can be initialized by all the Object, include your own created Object.

To initialize an Optional, we have three method, and mostly you will only use two of them, as I think.

{% highlight java linenos %}
 //empty option
 Optional<String> emptyOption = Optional.empty();

 // This will throw NullPointerException, we must confirm the object we pass is a non-null object
 Optional<Document> docNonNullOption = Optional.of(nullDocument);

 //This allow null object passed in
 Optional<Document> docNullableOption = Optional.ofNullable(docWithNullProperty);
{% endhighlight %}

You can pass your object to *of()* and *ofNullable()*, no matter it's a null or non-null object. *of()* will throw NullPointerException, while *ofNullable()* will create an empty Optional.

We simply can use *get()* to get the object from Optional. If the optional is an empty optional, it will throw us a NoSuchElementException. While this is still not perfect, we only want to do something if the Optional really contains something. So we need to check whether the object inside the Optional is a null or a non-null object. To achieve this, we have two approach, *isPresent()* and *ifPresent()*. The first one returns a Boolean and second one can direct run certain code if the optional is not empty.

{% highlight java linenos %}
 //use isPresent to check condition
 if (docNullableOption.isPresent()) {
 // content will be null, since we didn't check whether inside document all the property is non-null value.
    printVal(docNullableOption.get());
 }

 //user ifPresent to consume some certain logic
 docNullableOption.ifPresent(d -> printVal(d));
{% endhighlight %}

If the Optional is really an empty optional, we don't just throw a exception, but want to execute some default method. We can do this with *orElse()* and *orElseGet()*

{% highlight java linenos %}
 //if option is empty return something else
 String documentTitle = docNullableOption.map(Document::getTitle).orElse("Empty Title");

 //if option is empty do something else
 String documentContent = docNullableOption.flatMap(Document::getContentOption).orElseGet(() -> defaultContent());
{% endhighlight %}

Sometimes check whether object itself is null is not enough. We also want to ensure the inner property is also not null once we consume it. To achieve this, we can use *map()* and *flatMap()*

{% highlight java linenos %}
 //get document title option
 Optional<String> docTitleOption = docNullableOption.map(Document::getTitle);

 //difference between this one and map is that getContentOption directly return option, while getTitle just return normal Object
 Optional<String> docContentOption = docNullableOption.flatMap(Document::getContentOption);
{% endhighlight %}

Base on the method provide by Optional, we check whether the object is null or not, and also its property. Is it the best approach to avoid null in your code, hard to say. At least, we have an alternative way to do the null check. Be honest, I think null is the worst design in software history, 80% of my debugging time is used on how to solve a NullPointerException problem.

The good thing is not only Java has a solution, other language also have more or less same method to avoid null ([The worst mistake of computer science](https://www.lucidchart.com/techblog/2015/08/31/the-worst-mistake-of-computer-science/)). Some of them even don't have null at all. If you are using other language, i think you can have other solution to avoid this ugly null in your code.

If you are confused to the code showed above, you can check my original code on [my github](https://github.com/zzyclark/JavaOptionalTest).
