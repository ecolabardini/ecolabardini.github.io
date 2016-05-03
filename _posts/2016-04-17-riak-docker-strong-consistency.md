---
layout: post
title: Riak, Docker & Strong Consistency
date: '2016-04-17T19:49:00.005-03:00'
author: Eduardo Colabardini
tags: 
- riak
- strong consistency
- nosql
- eventual consistency
---

Riak is a distributed NoSQL key-value data store that offers high availability, fault tolerance, operational simplicity, and scalability. Riak implements the principles from [Amazon's Dynamo paper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) with heavy influence from the [CAP](https://en.wikipedia.org/wiki/CAP_Theorem) theorem. ([Wikipedia](https://en.wikipedia.org/wiki/Riak))

Riak allows you to create buckets that provide [strong consistency](http://docs.basho.com/riak/kv/2.1.4/developing/app-guide/strong-consistency/) guarantees for the data stored within them, enabling you to use Riak as a CP system (consistent plus partition tolerant) for all of the data in that bucket. You can store just some of your data in strongly consistent buckets or all of your data, depending on your use case. Strong consistency was added to complement Riak’s standard [eventually consistent](https://en.wikipedia.org/wiki/Eventual_consistency), high availability mode. ([Basho](http://docs.basho.com/riak/kv/2.1.4/developing/app-guide/strong-consistency/))
