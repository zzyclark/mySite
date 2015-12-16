---
published: true
title: CouchDB With HibernateOGM
layout: post
author: Clark
category: articles
tags:
- CouchDB
- Java
description: A quick set up for HibernateOGM on CouchDB
comments: true
---
Base on an article I have read recently, I try to start get my hand dirty with various NoSQL database. This [article](http://kkovacs.eu/cassandra-vs-mongodb-vs-couchdb-vs-redis) compares among lots of NoSQL databases famous today. I should thank the author who give me a general idea about the difference between these NoSQL databases. These NoSQL databases can be separated by store technologies. Like CouchDB, Hbase, they use classic document and BigTable stores, Neo4j use Graph databases etc.

I start with CouchDB because I want to build a demo CMS system which will accumulation and occasionally changing data. What's more with the replication feature provided by CouchDB, my database can easily grows big and I can try to build a database cluster base on this.

The protocol for CouchDB is HTTP/REST. Each time, we can query database by a REST call from our server side. Here I will initialize our database.I assume you already have your CouchDB installed on your machine, we can use both CURL from terminal or the graphical interface (The default graphical interface locate at http://127.0.0.1:5984/_utils/) to manage or database.

To crate database couchdbwithhibernateogm:

{% highlight BASH%}
curl -X PUT http://127.0.0.1:5984/couchdbwithhibernateogm
{% endhighlight %}

and you will receive:

{% highlight JSON%}
{"ok":true}
{% endhighlight %}

The database is setup ready, now let's get our hand dirty to connect the data with HibernateOGM. We also have other API option to call your database, like Ektorp which I will explain how to set it up in future article.

HibernateOGM use JPA like HibernateORM, I can't see it's good or not, since JPA was first invented to serve our ORM database better. However, if you come from HibernateORM, you may feel some configuration and API usability are quite familiar which will save our learning time for a new API.

Before start our project, let's set up our project dependencies:

{% highlight XML%}
        <!--JPA-->
        <dependency>
            <groupId>org.hibernate.javax.persistence</groupId>
            <artifactId>hibernate-jpa-2.1-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.jboss.spec.javax.transaction</groupId>
            <artifactId>jboss-transaction-api_1.2_spec</artifactId>
        </dependency>

        <!--JBoss transaction-->
        <dependency>
            <groupId>org.jboss.jbossts</groupId>
            <artifactId>jbossjta</artifactId>
        </dependency>

        <!--CouchDB-->
        <dependency>
            <groupId>org.hibernate.ogm</groupId>
            <artifactId>hibernate-ogm-couchdb</artifactId>
            <version>4.2.0.Final</version>
        </dependency>
{% endhighlight %}

In our JPA configuration: **persistence.xml**, we will set our transaction server provider, the database connection info etc.

{% highlight XML%}
<persistence-unit name="couchdb-hibernateogm" transaction-type="JTA">
      <!-- Use Hibernate OGM provider: configuration will be transparent -->
      <provider>org.hibernate.ogm.jpa.HibernateOgmPersistence</provider>
      <properties>
          <!-- defines which JTA Transaction we plan to use -->
          <property name="hibernate.ogm.datastore.provider" value="COUCHDB_EXPERIMENTAL"/>
          <property name="hibernate.ogm.datastore.host" value="localhost"/>
          <property name="hibernate.ogm.datastore.port" value="5984"/>
          <property name="hibernate.ogm.datastore.database" value="couchdbwithhibernateogm"/>
          <!-- If you didn't set up your database in CouchDB, this will help you create the database -->
          <!-- <property name="hibernate.ogm.datastore.create_database" value="true"/> -->
          <property name="hibernate.ogm.datastore.username" value="username"/>
          <property name="hibernate.ogm.datastore.password" value="password"/>
      </properties>
  </persistence-unit>
{% endhighlight %}

You can change your host name and port base on your setting, moreover if you have set admin of your database, please set the username and password base on your setting. HibernateOGM has provide lots of datastore connection driver, here we should also change our datastore provider to this special COUCHDB_EXPERIMENTAL.

We have two entities, they also have a **OneToOne** relationship. Here I will only show one of it. For more information, you can jump to the bottom to see my code on github.

{% highlight JAVA%}
package co.superclark.entity;

import org.hibernate.annotations.*;
import org.hibernate.ogm.datastore.document.options.AssociationStorage;
import org.hibernate.ogm.datastore.document.options.AssociationStorageType;

import javax.persistence.*;
import javax.persistence.CascadeType;
import javax.persistence.Entity;

/**
 * @Author clark
 * @Date 30-Nov-2015
 */
@Entity
@AssociationStorage(AssociationStorageType.ASSOCIATION_DOCUMENT)
public class Dog {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)
    private Long id;

    @Version
    @Generated(value = GenerationTime.ALWAYS)
    @Column(name = "_rev")
    private String revision;
    private String name;
    @OneToOne(cascade = CascadeType.PERSIST)
    private Breed breed;


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getRevision() {
        return revision;
    }

    public void setRevision(String revision) {
        this.revision = revision;
    }

    public Breed getBreed() {
        return breed;
    }

    public void setBreed(Breed breed) {
        this.breed = breed;
    }
}
{% endhighlight %}

The syntax is quite similar with HibernateORM, while we have some new thing can be configured. Since we are using CouchDB, we shoud specify our entity storage Type.

{% highlight JAVA%}
...
@Entity
@AssociationStorage(AssociationStorageType.ASSOCIATION_DOCUMENT)
public class Dog {
 ...
{% endhighlight %}

And for each of our entity, we must have the revision property, since this is needed for our CouchDB. Revision is compulsory for all of our entity, that's one of the key feature for our CouchDB to implement MVVM database structure. Your can also check [this](http://guide.couchdb.org/draft/consistency.html) to see how did CouchDB implement it.

{% highlight JAVA%}
...
@Version
@Generated(value = GenerationTime.ALWAYS)
@Column(name = "_rev")
private String revision;
...
{% endhighlight %}

Lastly, we can also set our ID generation strategies. In CouchDB, they default use _UUID as the document id, while sometimes, we have our own requirement. HibernateOGM provide multiple generation strategies, like **GenerationType.TABLE** or **GenerationType.SEQUENCE**. In this demo, I use **GenerationType.TABLE**.

{% highlight JAVA%}
...
@Id
@GeneratedValue(strategy = GenerationType.TABLE)
private Long id;
...
{% endhighlight %}

Finally, let's see how to handle our data. We initialize transaction manager and entity manager first.

{% highlight JAVA%}
...
TransactionManager tm = com.arjuna.ats.jta.TransactionManager.transactionManager();

//build the EntityManagerFactory as you would build in in Hibernate Core
EntityManagerFactory emf = Persistence.createEntityManagerFactory( "couchdb-hibernateogm" );

EntityManager em = emf.createEntityManager();
...
{% endhighlight %}

I will create a new Dog entity. Here I have already set the CascadeType for Breed, so create Dog will automatically create related Breed.

{% highlight JAVA%}
...
Dog dina = new Dog();
dina.setName( "Nancy" );
Breed breed = new Breed();
breed.setName("Ding");
dina.setBreed(breed);

tm.begin();
em.persist(dina)
em.flush();
em.close();
tm.commit();
...
{% endhighlight %}

To read the dog:
{% highlight JAVA%}
...
tm.begin();
Dog dinaFind = em.find(Dog.class, dina.getId());
em.flush();
em.close();
tm.commit();
...
{% endhighlight %}

To remove the dog:

{% highlight JAVA%}
...
tm.begin();
em.remove(dinaFind);
em.flush();
em.close();
tm.commit();
...
{% endhighlight %}

Update process is almost same as create process, besides, you should grab your entity first and then persist it. Except plain JPA, you can also use HibernateOGM native API to do the same function. HibernateOGM native API is almost same the HibernateORM's, it will be easy to pick this up. To initialize HibernateOGM session, we wrap it from JPA, or bootstrap directly from Hibernate ORM native APIs. In my example, I have show how to wrap it from JPA, you can find more information from my [github](https://github.com/zzyclark/CouchDBWithHIbernateOGM).
