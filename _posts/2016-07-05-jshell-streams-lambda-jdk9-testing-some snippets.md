---
layout: post
title: JShell, Streams, Lambda, JDK9 - Testing some snippets!
date: '2016-07-05T10:00:00.005-03:00'
author: Eduardo Colabardini
tags:
- java
- jshell
- jdk9
- streams
- lambda
---

Java 9 introduces JShell and a Read-Eval-Print Loop ([REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop){:target="_blank"}) for the Java Programming Language. [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop){:target="_blank"} allows you to evaluate code snippets such as declarations, statements, expressions.

<!-- more -->

I created a [docker image](https://github.com/ecolabardini/docker-jshell){:target="_blank"} with JDK9 so we can execute some examples:

~~~ bash
sudo docker run -it ecolabardini/jdk9 bash
jshell
~~~

Removing duplicated items from a list:

~~~ java
Arrays.asList("item1", "item2", "item2", "item3", "item3")
  .stream()
  .distinct()
  .forEach(System.out::println)
~~~

Killing processes by name:

~~~ java
ProcessHandle
  .allProcesses()
  .filter(p -> p.info().command().get().contains("jshell"))
  .findFirst().get()
  .destroy()
~~~

(ouch! we killed ourselves!)

A simple GET request to my blog:

~~~ java
import java.net.http.*;
HttpRequest
  .create(new URI("https://ecolabardini.github.io/"))
  .GET()
  .response()
  .body(HttpResponse.asString())
~~~

#### Processing and filtering content

The file [transactions.csv](http://ecolabardini.github.io/assets/transactions.csv){:target="_blank"} has some real transactions from Sacramento (California, EUA). Let’s download it and do some basic statistics:

~~~ bash
sudo docker run -it ecolabardini/jdk9 bash
wget http://ecolabardini.github.io/assets/transactions.csv
jshell
~~~

Here is the first three lines of the file (the first one is the CSV header):

~~~ bash
street,city,zip,state,beds,baths,sq__ft,type,sale_date,price,latitude,longitude
3526 HIGH ST,SACRAMENTO,95838,CA,2,1,836,Residential,Wed May 21 00:00:00 EDT 2008,59222,38.631913,-121.434879
7340 HAMDEN PL,SACRAMENTO,95842,CA,2,2,1134,Condo,Wed May 21 00:00:00 EDT 2008,110700,38.700051,-121.351278
(...)
~~~

Filtering by “Condo” and counting (the seventh column):

~~~ java
import java.nio.file.*
Files
  .lines(Paths.get("transactions.csv"))
  .filter(t -> t.split(",")[7].equals("Condo"))
  .count()
~~~

The average price of places with 3 beds (the fourth column):

~~~ java
Files
  .lines(Paths.get("transactions.csv"))
  .filter(t -> t.split(",")[4].equals("3"))
  .mapToInt(t -> new Integer(t.split(",")[9]))
  .average()
  .getAsDouble()
~~~

Of course there are simpler and quicker ways to solve these examples with just ``/bin/bash`` (using grep, sort, awk etc), but the purpose here is that your code can be tested as you create it, and way before you are done with your whole project.

### References

* [JShell and REPL in Java 9](https://blogs.oracle.com/java/jshell-and-repl-in-java-9){:target="_blank"}
* [Java™ Platform, Standard Edition 9 API Specification](http://download.java.net/java/jdk9/docs/api/index.html){:target="_blank"}
* [Functional Programming in Java: Beyond The Hype (in portuguese)](https://www.infoq.com/br/presentations/codigo-funcional-em-java-superando-o-hype){:target="_blank"}
