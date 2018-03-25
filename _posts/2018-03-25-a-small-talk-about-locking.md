---
layout: post
title: A Small Talk About Locking
date: '2018-03-25T18:00:00.000-03:00'
author: Eduardo Colabardini
tags:
- locking
- redis
- database
- mysql
- zookeeper
- nosql
- crdt
- eventual consistency
- riak
---

I've been recently working on a new project that needs to maintain a strong consistency of values between concurrent accesses. In order to better understand this problem you can think something as:

- Keeping your bank account balance always right
- Handling correctly the number of products in a shelf
- The number of likes or views of a video or article

<!-- more -->

You should be wondering what's the complexity in managing these problems. For this, let me present you *The Lost Update Problem*.

<!---
https://bramp.github.io/js-sequence-diagrams/#syntax

Title: The Lost Update Problem
Transaction #1 -> Database:how many products in stock?
Transaction #2 -> Database:how many products in stock?
Database -> Transaction #1: 10
Database -> Transaction #2: 10
Note right of Transaction #1: sell 3 units
Note left of Transaction #2: sell 5 units
Transaction #1 -> Database:update units in stock as 7
Transaction #2 -> Database:update units in stock as 5
-->
<!-- <center>The Lost Update Problem</center> -->
![The Lost Update Problem](/assets/the_lost_update_problem_diagram.svg)

This problem is fairly known in the computer science field and is about, just like the name says, losing an update due to concurrent access. As you can see the final number of items available in stock will be 5 where it should be 2.

To solve this issue we need to think about locking, better defined as

> "A mechanism to synchronize access by multiple users to the same piece of data at the same time".

We can easily overcome this problem with database locking but it gets a little more tricky when we're talking about distributed systems.

## Database locking

### A note about Isolation Levels

Isolation determines how transaction integrity is visible to other users and systems. It's very important to say that this can't solve the issue described in this article.

| Isolation level  	| Dirty Reads 	| Non-repeatable reads 	| Phantom Reads 	|
|------------------	|-------------	|----------------------	|---------------	|
| Read Uncommitted 	| YES         	| YES                  	| YES           	|
| Read Committed    	| NOPE        	| YES                  	| YES           	|
| **Repeatable Read**  	| NOPE        	| NOPE                 	| YES           	|
| Serializable     	| NOPE        	| NOPE                 	| NOPE          	|

Changing the default from `Repeatable Read` can just make more difficult to track a problem and will introduce you to a whole new world of inconsistencies.

Even if you allow dirty reads there will be a moment where your data access won't be synchronized amongst concurrent transactions. `Tip: Don't change this!`

### Pessimistic and optimistic locking

The easiest way to solve _The Lost Update Problem_ is just to lock for writing the row you want to update before actually updating it (*pessimist locking*). Most databases support the following statement:

```sql
SELECT * FROM products WHERE id = 1 FOR UPDATE;
```

That means that other sessions can read the row, but cannot modify it until your transaction commits. So we can easily solve the concurrent access by... not having concurrent access! Databases can solve most of the issues from small-to-big companies and can work under a heavy load of transactions, but there are sometimes that this can lead to performance issues.

The *optimistic locking* assumes that although conflicts are possible, they will be very rare. It allows fast performance and high concurrency, at the cost of occasionally refusing to write data that was initially accepted but was found at the last second to conflict with another user's changes.

As we know, the `UPDATE` statement already locks the row for writing, so it's just one update at a time. This way we can just add a check for the old value while updating it:

```sql
UPDATE products SET units=5 WHERE units=10;
```

Other concurrent transactions will not satisfy the `WHERE` clause, updating zero rows. If that happens, you'll need to do a rollback and inform the user that the `UPDATE` failed, or you can just do the entire transaction again.

## Distributed Systems

Let me introduce to you some new problems:

- We're not using a database that natively supports locking or transactions
- We're working with distributed systems that need to lock a access to a shared resource

First of all, let me remind you of the first law of distributed objects: **don't distribute your objects** (thanks [Martin Fowler](https://martinfowler.com/){:target="_blank"}). If you can avoid going into distributed systems then do it. This will require solving a lot more problems than the ones listed above, such as: failure handling (distributed rollback), data consistency and replication, fault tolerance, resiliency, throttling/rate limit, retries and so on...

### External locking

If you're going into it, then you'll need external locking. Here are some articles and tools that can help you in this new adventure:

**[Apache Zookeeper](https://zookeeper.apache.org/){:target="_blank"}**

Softwares like [Apache Hadoop](http://hadoop.apache.org/){:target="_blank"} (distributed processing of data), [Apache Kafka](https://kafka.apache.org/){:target="_blank"} (distributed streaming platform) and [Apache Mesos](http://mesos.apache.org/){:target="_blank"} (distributed systems kernel) are using Zookeeper for distributed coordination and synchronization.

**[Distributed locks with Redis](https://redis.io/topics/distlock){:target="_blank"}**

Redis is an in-memory database with lots ready-to-use data structures and algorithms. This article  explains how to use an algorithm called **Redlock** that implements a DLM (Distributed Lock Manager). You can also use a single-node Redis with the [SETNX command](https://redis.io/commands/setnx){:target="_blank"} (discouraged in favor of the first one).

**[How to coordinate distributed work with MySQL's GET_LOCK](https://www.xaprb.com/blog/2006/07/26/how-to-coordinate-distributed-work-with-mysqls-get_lock/){:target="_blank"}**

A very good article by [Baron Schwartz](https://www.xaprb.com/){:target="_blank"}, co-author of [High Performance MySQL](http://shop.oreilly.com/product/0636920022343.do){:target="_blank"} (published by O'Reilly Media) talking about a MySQL function called [GET_LOCK](https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html#function_get-lock){:target="_blank"} capable of locking a user defined string. Of course you should not use this to lock rows (MySQL already does that very well) but probably to lock external resources.

**[How to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html){:target="_blank"}**

This is a very well written article by [Martin Kleppmann](https://martin.kleppmann.com/){:target="_blank"}, the author of [Designing Data-Intensive Applications](http://dataintensive.net/){:target="_blank"} (also published by O'Reilly Media) talking about the usefulness of locking, some considerations about the **Redlock** algorithm and **fencing tokens**.

### Distributed data stores

#### CAP Theorem

As discussed above, there are several ways of doing locking and avoiding inconsistencies. [RDBMSs](https://en.wikipedia.org/wiki/Relational_database_management_system){:target="_blank"} are already known as great tools for this job and they are what is called a **CP** system, i.e., they provide an environment with **C**onsistency plus **P**artition tolerance.

The [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem){:target="_blank"} states that in the presence of a network partition, one has to choose between consistency and availability:

|**Consistency**|Every read receives the most recent write or an error|
|**Availability**|Every request receives a (non-error) response – without guarantee that it contains the most recent write|
|**Partition tolerance**|The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes

Distributed data stores are more focused in delivering availability over consistency and have some features that can help us dealing with concurrent access in a different way.

#### Conflict resolution and eventual consistency

Performance and availability comes with a cost. You can have multiple updates in the same resource and in different nodes of your cluster. When that happens you can end up with two different versions of the data (also known as *siblings*), that can (or cannot) eventually converge.

Some ways of resolving conflicts are:

- timestamps (not very reliable in distributed systems)
- last write wins (dropping some writes in the case of concurrent updates)
- [vector clocks](https://en.wikipedia.org/wiki/Vector_clock){:target="_blank"}
- [dotted version vectors](https://en.wikipedia.org/wiki/Version_vector){:target="_blank"}

These last two items are mechanisms for tracking object updates in terms of logical time rather than chronological time (as with timestamps), enabling then a better conflict resolution. But if a conflict can't be solved then it's the application’s responsibility to deal with it.

Proposed at the beginning of this article, the problem `The number of likes or views of a video or article` can be easily solved by [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type){:target="_blank"}. They are data structures that can be updated independently and concurrently without coordination, and where it is always mathematically possible to resolve inconsistencies which might result. [Riak Data Types](http://docs.basho.com/riak/kv/2.2.3/learn/concepts/crdts/){:target="_blank"} are a good examples about this.

The point here is:

> You can design your system to understand eventual consistency and conflict resolution. Some problems can be easily modeled and resolved without explicit locking.

## Conclusion

Understand your needs. Relational databases are a long standing technology and very accurate way of handling all these problems. On the other site, probably with a higher cost but higher performance, availability and so on, are distributed systems and distributed data stores, handling problems in a very different way. `Keep studying!`

## References
* [MySQL 5.7 Reference Manual - Locking Reads](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html){:target="_blank"}
* [IBM Knowledge Center - Pessimistic vs optimistic concurrency control](https://www.ibm.com/support/knowledgecenter/SSPK3V_6.5.0/com.ibm.swg.im.soliddb.sql.doc/doc/pessimistic.vs.optimistic.concurrency.control.html){:target="_blank"}
* [Microservices and the First Law of Distributed Objects, Martin Fowler](https://martinfowler.com/articles/distributed-objects-microservices.html){:target="_blank"}
* [Conflict Resolution, Basho Docs - Riak KV](http://docs.basho.com/riak/kv/2.2.3/developing/usage/conflict-resolution/){:target="_blank"}
* [How to do distributed locking, Martin Kleppmann
](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html){:target="_blank"}
* [Riak Data Types and CRDTs, Basho Docs - Riak KV](http://docs.basho.com/riak/kv/2.2.3/learn/concepts/crdts/){:target="_blank"}
