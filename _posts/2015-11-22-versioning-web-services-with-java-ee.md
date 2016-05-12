---
layout: post
title: Versioning web-services with Java EE
date: '2015-11-22T22:52:00.000-02:00'
author: Eduardo Colabardini
tags:
- jaxrs
- javaee
- rest
- archetype
- api
- maven
- restful
- wildfly
---

So, how do you plan on versioning your web-services?

This is crucial today, when you business logic is changing all the time and you have to add new fields, parameters, validations and so on. But how to do this without breaking products that already depend upon your API?

There are some simple ways to accomplish this:

* Add the version on you URL path: <http://www.example.com/api/v2/user>
* Add a custom request header: ``api-version: 2``
* Modify the accept header: ``Accept: application/com.example.v2+json``
* Add a query parameter: <http://www.example.com/api/user?version=v2>

In this tutorial I will cover the first method with Java EE, since it is more evident to customers and easy to implement because of the way JAX-RS handles paths (I think it might break the RESTful philosophy because the resource path changes...).

Let’s start with a Maven Archetype (I’m using Apache Maven 3.3.3). The command below generates a Java EE 7 webapp project for Wildfly. The project is an EAR, with an EJB-JAR and WAR.

~~~bash
mvn -B archetype:generate \
-DgroupId=com.github.ecolabardini \
-DartifactId=example \
-Dversion=0.1-SNAPSHOT  \
-DarchetypeArtifactId=wildfly-javaee7-webapp-ear-blank-archetype \
-DarchetypeGroupId=org.wildfly.archetype \
-DarchetypeVersion=8.2.0.Final \
-DarchetypeRepository=http://repository.jboss.org/nexus/content/groups/public
~~~

Download Wildfly 8.2.1 Final, extract and run it on standalone mode:

~~~bash
wget http://download.jboss.org/wildfly/8.2.1.Final/wildfly-8.2.1.Final.tar.gz
tar xvfz wildfly-8.2.1.Final.tar.gz
./wildfly-8.2.1.Final/bin/standalone.sh
~~~

I created two Java classes (``rest.v1.UserRest`` and ``rest.v2.UserRest``) on the ``example-web`` project, they are exactly the same except for the package name. They are mapped on the ``/user`` REST path, and I added a GET method ``/user/123`` that prints ``rest.v1: Hi, I'm user with ID 123``.

~~~java
package rest.v1;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("user")
public class UserRest {
    @GET
    @Path("/{id}")
    @Produces(MediaType.TEXT_PLAIN)
    public String getUser(@PathParam("id") String id) {
        return this.getClass().getPackage().getName() + 
        	": Hi, I'm user with ID " + id;
    }
}
~~~

We need to add the base path to these two implementations (``/api/v1/user/123`` and ``/api/v2/user/123``), so we implement the ``javax.ws.rs.core.Application`` defining the ``@ApplicationPath``. So, again, I created two Java classes (``rest.v1.Application`` and ``rest.v2.Application``) on the ``example-web`` project, they are responsible for identifying which class works on each path, they are exactly the same except for the package name, ``@ApplicationPath`` and the UserRest import (``v1`` or ``v2``).

~~~java
package rest.v1;

import java.util.HashSet;
import java.util.Set;

import javax.ws.rs.ApplicationPath;

@ApplicationPath("/api/v1")
public class Application extends javax.ws.rs.core.Application {
	@Override
	public Set<Class<?>> getClasses() {
	    final Set<Class<?>> classes = new HashSet<>();
	    classes.add(UserRest.class);
	    return classes;
	}
}
~~~

To deploy the application, you can download the source code [here](https://s3-us-west-2.amazonaws.com/ecolabardini/versioning-webservices.zip) and execute the following commands:

~~~bash
mvn clean install
cp example-ear/target/example-ear.ear wildfly-8.2.1.Final/standalone/deployments/
~~~

Then you can access:

* <http://localhost:8080/example-web/api/v1/user/1>
* <http://localhost:8080/example-web/api/v2/user/1>

As you can see, we have two running versions of the ``UserRest``, and your client/customer is free to decide when it’s time to update his logic to the next version.

Note that on ``rest.v2.Application`` class you can still import classes from ``rest.v1`` package, exposing a new version of ``UserRest`` and maintaining an old version of other REST classes.

And if you are already providing a web-service to your customers, you can omit the first version ``v1``, so everything continues working!

[Download source code](https://s3-us-west-2.amazonaws.com/ecolabardini/versioning-webservices.zip)

### References
* [Creating a RESTful Root Resource Class, The Java EE Tutorial](https://docs.oracle.com/javaee/7/tutorial/jaxrs002.htm){:target="_blank"}
* [Your API versioning is wrong, Troy Hunt](http://www.troyhunt.com/2014/02/your-api-versioning-is-wrong-which-is.html){:target="_blank"}