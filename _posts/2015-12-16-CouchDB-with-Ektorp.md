---
published: true
title: CouchDB With Ektorp
layout: post
author: Clark
category: articles
tags:
- CouchDB
- Java
description: A quick set up for Ektorp on CouchDB
comments: true
---
From official [CouchDB wiki](https://wiki.apache.org/couchdb/Getting_started_with_Java), there are a lot of Java CouchDB API, like Ektorp, JRelax, jcouchdb etc. Here I pick the first one to implement it, because I always think the first one will be the most recommended one. Last time, I have already showed the implementation about HibernateOGM on CouchDB, so why we still need this Java API. My idea is that HibernateOGM comes with HibernateORM which means it will keep both the advantage and disadvantage. The initial purpose of Hibernate is to find a way to better implement our SQL database, which makes it giant or in other word **bloated**. In some way, to the contrary. The native CouchDB java API is built only to support CouchDB, which means most time it will be tiny, fast and have better functionality support for native CouchDB specifics.

Now, let's get our hand dirty with this Ektorp. Ektorp use a HttpClient to communicate with our database. After the connection is build, we can specify which document database we are gonna use. During the set up, we can specify the parameter base on your own database design.

{% highlight JAVA%}
HttpClient authenticatedHttpClient = new StdHttpClient.Builder()
                .username("admin")
                .password("password")
                .build();

CouchDbInstance dbInstance = new StdCouchDbInstance(authenticatedHttpClient);
CouchDbConnector db = dbInstance.createConnector("ektorptest", true);
{% endhighlight %}

Once we have our DBConnector, we can directly persist our entity or fetch it from database. While I will not do this. Ektorp provide a repository layer to help us do some simple CRUD management, and if you want to customize some specific database action, you can also extend your repository layer easily. But before implement the repository layer, I will build our entities first. The structure is quite simple, we have a user entity and a dog entity and each user may have many dogs (what a pet lover! lol). Here are our user and dog entities.

User Entity:
{% highlight JAVA%}
public class User extends CouchDbDocument {
    private String name;
    @TypeDiscriminator
    private Integer age;

    @DocumentReferences(backReference = "userId", fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    private Set<Dog> dogs;

    ...getter and setter...

    public void addDog(Dog dog) {
      if (null == this.dogs) {
          this.dogs = new TreeSet<Dog>();
      }
      dog.setUserId(getId());
      this.dogs.add(dog);
    }
  }
{% endhighlight %}


Dog Entity:
{% highlight JAVA%}
public class Dog extends CouchDbDocument implements Comparable<Dog> {
    private String name;
    private String userId;

    ...getter and setter...

  }
{% endhighlight %}


To make our entity is quite simple, we just extend CouchDbDocument which already include the rev and id for each of our document. **@TypeDiscriminator** is used to separate our user document with other document, this annotation will be annotated on the specific property which only our user entity have. Sorry for the bad example, every body know not human being have age. But in my database design only user got age property, so i annotate it. Another thing need to be clarified is how we do our mapping. For now, Ektorp only support OneToOne and OneToMany, but not ManyToMany.

Here we can see our user-dog have a OneToMany relationship. First of all, in our user model, we need to use @DocumentReferences to indicate that our dog entity is referenced to user entity by which property. Secondly, in our dog, we must have the reference property which is userId to map each dog to certain master. During this, we can specify our fetch mode, cascade mode, etc.

Now is the time to write our repository layer. The original CouchDbRepositorySupport already give us some simple CRUD method, like save, update and delete. Here I will override the getAll method, which I will tell the database i only want to list all user documents from database.

UserRepository:
{% highlight JAVA%}
public class UserRepository extends CouchDbRepositorySupport<User> {
    public UserRepository(CouchDbConnector db) {
        super(User.class, db);
        initStandardDesignDocument();
    }

    @GenerateView
    @Override
    public List<User> getAll() {
        ViewQuery q = createQuery("all").includeDocs(true);
        return db.queryView(q, User.class);
    }
  }
{% endhighlight %}


The @GenerateView annotation will update the user design document to specify what we want to list when fetch all user models. The design document is unique for each entity we have from our entities. If any of our entities have special design, this design document will be generated automatically. For example, our user entity have a reference entity dogs, with each fetch mode, this one will be showed in user design document.

Once the repo layer is finished, we can really manipulate our database now. Here I will create a user with two dogs, fetch it from database and list all user also.

{% highlight JAVA%}
      UserRepository repository = new UserRepository(db);

      Dog d1 = new Dog();
      d1.setName("jerry");

      Dog d2 = new Dog();
      d2.setName("tom");

      User u = new User();
      u.setName("clark");
      u.setAge(25);

      //Persist user first, then we can have id
      repository.add(u);

      u.addDog(d1);
      u.addDog(d2);

      repository.update(u);

      User getUser = repository.get(u.getId());

      //list all user
      List<User> userList = repository.getAll();
{% endhighlight %}

So far, we have do some simple user create and update action in our database, you can find more APIS from Ektorp api docs, like delete. Ektorp also support Sping web mvc framework, you can find more information [here](http://ektorp.org/). All the source code showed can be found in my [github](https://github.com/zzyclark/CouchDBWithEktorp).
