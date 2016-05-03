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

Riak is a distributed NoSQL key-value data store that offers high availability, fault tolerance, operational simplicity, and scalability. Riak implements the principles from [Amazon's Dynamo paper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf){:target="_blank"} with heavy influence from the [CAP](https://en.wikipedia.org/wiki/CAP_Theorem){:target="_blank"} theorem. ([Wikipedia](https://en.wikipedia.org/wiki/Riak){:target="_blank"})

Riak allows you to create buckets that provide [strong consistency](http://docs.basho.com/riak/kv/2.1.4/developing/app-guide/strong-consistency/){:target="_blank"} guarantees for the data stored within them, enabling you to use Riak as a CP system (consistent plus partition tolerant) for all of the data in that bucket. You can store just some of your data in strongly consistent buckets or all of your data, depending on your use case. Strong consistency was added to complement Riak's standard [eventually consistent](https://en.wikipedia.org/wiki/Eventual_consistency){:target="_blank"}, high availability mode. ([Basho](http://docs.basho.com/riak/kv/2.1.4/developing/app-guide/strong-consistency/){:target="_blank"})

In this blog post I’ll show you how to quickly run five Riak nodes using Docker with strong consistency enabled (I’m using Ubuntu 14.04.4 LTS).

If you don’t have Docker installed, please follow this tutorial: [Install Docker](https://docs.docker.com/linux/step_one/){:target="_blank"}

Building the Riak Docker image (thanks Hector Castro):

~~~ bash
git clone https://github.com/hectcastro/docker-riak.git
cd docker-riak && make build
~~~

Running a 5 node cluster with strong consistency enabled:

~~~ bash
export DOCKER_HOST="unix:///var/run/docker.sock"
DOCKER_RIAK_AUTOMATIC_CLUSTERING=1 DOCKER_RIAK_CLUSTER_SIZE=5 DOCKER_RIAK_STRONG_CONSISTENCY=on make start-cluster
~~~

You can get the nodes IP addresses with the following command:

~~~ bash
make test-cluster | egrep -A6 "ring_members"
~~~

Connect to a Riak Docker instance to create a strongly consistent bucket type:

~~~ bash
docker exec -it riak01 bash
riak-admin bucket-type create strongly_consistent \ 
  '{"props":{"consistent":true}}'
riak-admin bucket-type activate strongly_consistent
~~~

From now on the cluster is ready to be used. Let’s perform some CRUD operations!
Insert operation with key "hello" and value "world":

~~~ bash
curl -v 172.17.0.2:8098/types/strongly_consistent/buckets/test/keys/hello \
  -X PUT \
  -H "Content-type: text/plain" \
  -d "world"

< HTTP/1.1 204 No Content
~~~

Fetching the key "hello" from another Riak node: 

~~~ bash
curl -v 172.17.0.3:8098/types/strongly_consistent/buckets/test/keys/hello

< HTTP/1.1 200 OK 
< X-Riak-Vclock: a85hYGBgzGBKYWBJLU4tzGDKY2XgZwQKAendfh0X+LIA
world
~~~

Strongly consistent buckets cannot allow siblings by definition, and so all writes to existing keys must include a context with the object. If you attempt a write to a non-empty key without including causal context, you will receive the following error: 

~~~ bash
curl -v 172.17.0.2:8098/types/strongly_consistent/buckets/test/keys/hello \
  -X PUT \
  -H "Content-type: text/plain" \
  -d "hell"

< HTTP/1.1 412 Precondition Failed 
~~~

Update using context (X-Riak-Vclock header):

~~~ bash
curl -v 172.17.0.4:8098/types/strongly_consistent/buckets/test/keys/hello \
  -X PUT \
  -H "Content-type: text/plain" \
  -H "X-Riak-Vclock: a85hYGBgzGBKYWBJLU4tzGDKY2XgZwQKAendfh0X+LIA" \
  -d "darling"

< HTTP/1.1 204 No Content
~~~

Finally, to stop the cluster execute the following:

~~~ bash
make stop cluster
~~~

### References

* [Bringing Consistency to Riak - Joseph Blomstedt, RICON2012](https://vimeo.com/51973001){:target="_blank"}
* [Strong Consistency](http://docs.basho.com/riak/kv/2.1.4/developing/app-guide/strong-consistency/){:target="_blank"}
* [Riak Quick Start with Docker](http://basho.com/posts/technical/riak-quick-start-with-docker){:target="_blank"}
* [A Docker project to bring up a local Riak cluster](https://github.com/hectcastro/docker-riak){:target="_blank"}
