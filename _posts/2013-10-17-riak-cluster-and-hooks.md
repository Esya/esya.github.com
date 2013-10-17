---
layout: post
title: "Riak's hooks on multiple nodes"
category: posts
---

I spent a good portion of my day trying to figure out how the node coordination works in Riak with hooks[^hooks]. We were noticing that on our 6-nodes cluster, even if we only pointed our apps at one node, the hooks would trigger randomly on the 6 of them.



It is because when you send a commit to a node, the hook (be it a post-commit or a pre-commit hook) is triggered only on the **coordinating node**, which can be a different node than **the receiving node**, and this is decided by the internal hashing. This coordinating node takes care of the replication, insuring that at least **N nodes** *(configured per bucket)* have the data [^bucketprops].

[Eric Redmond](https://github.com/coderoshi) gave me a good illustration of this :

> Yes, the coordinator is actually the first node in the "preference list". The preference list is the list of nodes that an object will be replicated to. So, say you have nodes A-F, but your N (replication) value is 3, you may be contacting Node A, but it may forward that request to node C. C would be the "coordinator", while D and E are the other primary replica nodes. So your 1/6 is exactly what you'd expect.

So from this I can give the following advice :

* Deploy your hooks to each node in your cluster, otherwise it **will** crash when the coordinator is a node that does not have the file.
* Do not forget to add the `add_path` directive to your `/etc/riak/app.config` on each node
* Try to make sure that each node has "nothing special" and that another node would do just the same thing
* Make sure that your apps are aware of multiple nodes, to avoid having a [single point of failure](http://en.wikipedia.org/wiki/Single_point_of_failure)



[^hooks]: http://docs.basho.com/riak/latest/theory/concepts/#Commit-Hooks
[^bucketprops]: http://docs.basho.com/riak/latest/dev/references/http/set-bucket-props/