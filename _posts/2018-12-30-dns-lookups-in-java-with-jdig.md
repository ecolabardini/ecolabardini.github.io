---
layout: post
title: DNS lookups in Java with JDig
date: '2018-12-30T15:35:00.000-03:00'
author: Eduardo Colabardini
tags:
- dns
- java
- jdig
---

Some time ago (~ Oct 2015) I wrote a DNS library for DNS lookups in Java called [JDig](https://github.com/ecolabardini/jdig){:target="_blank"}. In this post I'll show simples excerpts on how to use it and how it is made.

JDig is open source under the [Apache License](http://www.apache.org/licenses/LICENSE-2.0){:target="_blank"} and freely available on [GitHub](https://github.com/ecolabardini/jdig){:target="_blank"} and it also supports DNS-based Blackhole List (DNSBL) / Real-time Blackhole List (RBL) queries. [What is this?](https://en.wikipedia.org/wiki/DNSBL){:target="_blank"}

<!-- more -->

## How to use

The [README](https://github.com/ecolabardini/jdig){:target="_blank"} itself is full with good examples, but using it is simple as:


#### Add the following dependency to your project

```xml
<dependency>
    <groupId>com.github.ecolabardini</groupId>
    <artifactId>jdig</artifactId>
    <version>0.0.2</version>
</dependency>
```

#### Write your queries

```java
new DnsService("8.8.8.8:53").lookup("google.com", Type.any()).forEach(
    e -> System.out.println(e.getType() + " -> " + e.getName())
);
```

Should print something like this:
```
A -> 192.30.253.112
A -> 192.30.253.113
NS -> ns-1283.awsdns-32.org.
NS -> ns-520.awsdns-01.net.
MX -> ASPMX.L.GOOGLE.COM.
MX -> ALT1.ASPMX.L.GOOGLE.COM.
MX -> ALT2.ASPMX.L.GOOGLE.COM.
TXT -> "v=spf1 ip4:192.30.252.0/22 ip4:208.74.204.0/22 ip4:46.19.168.0/23 include:_spf.google.com include:esp.github.com include:_spf.createsend.com include:mail.zendesk.com include:servers.mcsv.net ~all"
```

In the above example we used the [Google Public DNS Server](https://en.wikipedia.org/wiki/Google_Public_DNS){:target="_blank"} (`8.8.8.8:53`) to query all [GitHub](https://github.com/){:target="_blank"} entries.

If we look carefully we can even notice some of the third party services that [GitHub](https://github.com/){:target="_blank"} is using: AWS, GMail, Zendesk and so on.

#### DNS blacklist queries

This kind of query is a powerful way to check the reliability of some IP adresses. With SORBS lists you can check for things like:


| List  	| Description 	|
|------------------	|-------------	|
| virus.dnsbl.sorbs.net 	| Hosts that have delivered known viruses to the SORBS spamtrap servers in the last 7 days.  The zone has a high overlap with the spam.dnsbl.sorbs.net as viruses that are not instantly recognised initially listed as spam (polymorphic viruses tend to do this.)	|
| new.spam.dnsbl.sorbs.net 	| List of hosts that have been noted as sending spam/UCE/UBE to the admins of SORBS within the last 48 hours.	|
| web.dnsbl.sorbs.net 	| List of web (WWW) servers which have spammer abusable vulnerabilities (e.g. FormMail scripts) Note: This zone now includes non-webserver IP addresses that have abusable vulnerabilities.	|

Let's check if the [GitHub](https://github.com/){:target="_blank"} IP address is spaming or sending virus to anyone:

```java
new DnsService()
    .blacklistLookup("192.30.253.112", "virus.dnsbl.sorbs.net", "spam.dnsbl.sorbs.net")
    .forEach(e -> {
        System.out.println(e.getType() + " -> " + e.getName());
    });
```

Ouch! We receveid an exception. That's because [GitHub](https://github.com/){:target="_blank"} is safe! :)

```bash
javax.naming.NameNotFoundException: DNS name not found [response code 3]
; remaining name '112.253.30.192.virus.dnsbl.sorbs.net'
```

If we try with some naughty address then we'll have an output like this:

```bash
A -> 127.0.0.15
TXT -> "Virus Transmitting Server See: http://www.sorbs.net/lookup.shtml?127.0.0.2"
A -> 127.0.0.6
TXT -> "Spam Received See: http://www.sorbs.net/lookup.shtml?127.0.0.2"
TXT -> "Escalated Listing (Spam or Spam Support) See: http://www.sorbs.net/lookup.shtml?127.0.0.2"
```

To understand SORBS return codes, check [Using SORBS](http://www.sorbs.net/using.shtml){:target="_blank"}. For more DNS blacklists [click here](https://en.wikipedia.org/wiki/Comparison_of_DNS_blacklists){:target="_blank"}.

## Implementation

Almost all the magic happens at the class [DnsService](https://github.com/ecolabardini/jdig/blob/master/src/main/java/com/jdig/service/DnsService.java#L23){:target="_blank"}.

The foundation of everything is, of course, how the query is made. We are using the `com.sun.jndi.dns.DnsContextFactory` implementation of the [`InitialContextFactory`](https://docs.oracle.com/javase/7/docs/api/javax/naming/spi/InitialContextFactory.html){:target="_blank"}.

With this factory we can build the [`InitialDirContext`](https://docs.oracle.com/javase/10/docs/api/javax/naming/directory/InitialDirContext.html){:target="_blank"} and then use the [`getAttributes`](https://docs.oracle.com/javase/10/docs/api/javax/naming/directory/DirContext.html#getAttributes(java.lang.String)){:target="_blank"} method that will return us attributes like `type`, `name` and `weight`.

For more information you can check our implementation here: [`getInitialDirContext()`](https://github.com/ecolabardini/jdig/blob/master/src/main/java/com/jdig/service/DnsService.java#L108){:target="_blank"}.

When creating a new `DnsService`, you can tune your application with several properties:

| Property  	| Description 	|
|------------------	|-------------	|
| dnsProviders 	| Specifies the host name and port of the DNS server(s), e.g.: 8.8.8.8:53	|
| timeout 	| Specifies the number of milliseconds to use as the initial timeout period using the exponential backoff algorithm. If this property has not been set, the default initial timeout is 1000 milliseconds.	|
| retries 	| Specifies the number of times to retry each server using the exponential backoff algorithm. If this property has not been set, the default number of retries is 4.	|
| authoritative 	|  If its value is "true", only authoritative responses are accepted from DNS servers; otherwise, all responses are accepted.	|
| recursion 	| This property is used to specify that recursion is allowed on DNS queries.	|

## References



