---
published: true
title: Lambda Expression in Java
layout: post
author: Clark
category: articles
tags:
- Java
description: a research on Java Lambda Expression implementation
comments: true
---

It has been a long time never update my blog since my [first post](http://superclark.co/articles/2015/04/30/Angularjs-Spring-Security-CSRF-Configuration.html). From April to November, our team devote all of our time to this [new product](http://www.cloardkit.com/). Good news is that it just launched last Friday, so i could have some to continue writing this blog.

Just like before, this product is base on Model-View-Controller architecture design. While during the development time, we met some troubles to maintain this MVC structure. We have persistence layer which is Model, a Service layer to handle all the data processing, and lastly, the controller layer to render the data to front view. We try to follow a simple role, each layer can only use the function from the deeper layer. For example, Service can only use the function from Dao, while controller can only use the function from Service. For most time, this rule is easy to follow, however some times we must handle some data filtering and handling into model and some other place where we don't want to inject any layer object. So in order to achieve such filtering and data handling function, some times we may pass the function as a parameter.

How's performance, so far so good. While, our code looks like a little bit dirty. Can you imagine such code like this:

{% highlight java %}

findAdultUser(userList, new CheckAdultUserService());

{% endhighlight %}

I pass a service object to this function. We can't this approach is wrong, but can we have any better approach to let our code looks more elegant and clean. In Java 8, we have Lambda Expression enable us to do this, it treats functionality as method argument, or code as data.

First of all let's solve some problem base on my old approach. Scenario is that we have a group of people, and we only need to print out the name of all the **adult people**

For the service, we will have a **UserCheckService** to **test()** out all valid user, a **UserNameMapService** to **apply()** (grab) all the user's name and lastly a **NamePrintService** to **accept()** (print out) all these names. If we want to create these all by ourselves, we need at least three interface and three class implementation to do this.

* UserCheckService
* UserCheckServiceImpl
* UserNameMapService
* UserNameMapServiceImpl
* NamePrintService
* NamePrintServiceImpl

In Java 8, all these interface are already pre-defined, so we don't need to define the interface again. All the implementation will like this. (Class name may vary)

UserCheckServiceImpl:
{% highlight java %}

public class CheckAdultUserPredicator implements Predicate<User> {
    @Override
    public boolean test(User user) {
        return user.getAge() > 18;
    }
}

{% endhighlight %}

UserNameMapServiceImpl:
{% highlight java %}

public class UserNameMapper implements Function<User, String>{
    @Override
    public String apply(User user) {
        return user.getName();
    }
}

{% endhighlight %}

NamePrintServiceImpl:
{% highlight java %}

public class NameDataConsumer implements Consumer<String> {
    @Override
    public void accept(String s) {
        System.out.println(s);
    }
}

{% endhighlight %}

Then we will create a function to combine all these service together to print all adult user name.

{% highlight java %}

private static void findAdultUserName(List<User> users,
  Predicate<User> predicator,
  Function<User, String> mapper,
  Consumer<String> consumer) {
      for (User u : users) {
          if (predicator.test(u)) {
              String s = mapper.apply(u);
              consumer.accept(s);
          }
      }
  }

{% endhighlight %}

In our main function, we will call like this:
{% highlight java %}

findAdultUserName(userList,
                new CheckAdultUserPredicator(),
                new UserNameMapper(),
                new NameDataConsumer());

{% endhighlight %}

Finally, we did it. Seems a lot of work have been done, and our code size increased a lot. You will not like this, now let's do this with Lambda Expression.

{% highlight java %}

findAdultUserName(userList,
                u -> u.getAge() > 18, //same as the predicator
                u -> u.getName(),     //same as the mapper
                name -> System.out.println(name));  //same as the consumer

{% endhighlight %}

Just like this, we don't need to do any other interface implementation. The first approach is like we give the laundry house our dirty clothes and a note which tell them how to wash our clothe. With Lambda Expression, We only pass our dirty clothes and orally tell them how to wash our clothes. We don't need to write note any more, wonderful. To emphasize here, you can only user a Lambda Expression for a functional interface, like (Predicate<T>, Consumer<T>, Function<T,R>, etc). Because a functional interface contains only one abstract method, we can omit the name of the method name when we use Lambda Expression to implement it.

To explain function findAdultUserName() step by step:

1. **Obtain source data from userList, it will be an object of type Iterable.**
2. **Filter objects that match the predicator object tester.**
3. **mapper each filtered object to a value.**
4. **Perform an action that consume our mapped value.**

If you even don't want to create this findAdultUserName() function, we also can do this with Aggregate Operation in our main function, which i will explain in future. Let's see how simple the code is firstly.

{% highlight java %}

userList.stream()
                .filter(u -> u.getAge() > 18)
                .map(u -> u.getName())
                .forEach(name -> System.out.println(name));

{% endhighlight %}

For the syntax of Lambda Expression, you can reference [oracle doc](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html#syntax). You can also check the my source code on [github](https://github.com/zzyclark/Lambda-Expression-in-Java).
