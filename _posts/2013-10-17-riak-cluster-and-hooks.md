---
layout: post
title: "Riak's hooks on multiple nodes"
category: posts
---

I spent a good portion of my day trying to figure out how the node coordination works in Riak with hooks. We were noticing that on our 6-nodes cluster, even if we only pointed our apps at one node, the hooks would trigger randomly on the 6 of them.



It is because when you send a commit to a node, the hook[^hooks] (be it a post-commit or a pre-commit hook) is triggered only on the **coordinating node**, which can be a different node than **the receiving node**, and this is decided by the internal hashing. This coordinating node takes care of the replication, insuring that at least **N nodes** *(configured per bucket)* have the data [^bucketprops].

[Eric Redmond](https://github.com/coderoshi) gave me a good illustration of this :

> Yes, the coordinator is actually the first node in the "preference list". The preference list is the list of nodes that an object will be replicated to. So, say you have nodes A-F, but your N (replication) value is 3, you may be contacting Node A, but it may forward that request to node C. C would be the "coordinator", while D and E are the other primary replica nodes. So your 1/6 is exactly what you'd expect.

So from this I can give the following advice :

* Deploy your hooks to each node in your cluster, otherwise it **will** crash when the coordinator is a node that does not have the file.
* Do not forget to add the `add_path` directive to your `/etc/riak/app.config` on each node
* Try to make sure that each node has "nothing special" and that another node would do just the same thing
* Make sure that your apps are aware of multiple nodes, to avoid having a [single point of failure](http://en.wikipedia.org/wiki/Single_point_of_failure)

### Deploying Riak hooks
As a bonus, here is a script to **deploy your hooks**. It will connect to a node, find all the members of the cluster, and deploy the files there.

{% highlight bash %}
#!/bin/bash
# The cluster will be read from there
firstnode="your-riak-node.com"
# User to connect to each node
user="root"
# Where you want the hooks to be stored
hookpath="~/riakhooks"
hosts=`ssh $user@$firstnode "riak-admin member-status | grep valid | sed -re \"s/^.*'.*@(.*)'.*$/\1/\""`

for host in $hosts
do
	echo "==========================="
	echo "+ Deploying to $host"
	echo "==========================="

	# Check that the node is configured properly
	out=`ssh $user@$host "cat /etc/riak/app.config | grep add_path | grep $hookpath"`
	if [[ $? -ne 0 ]]; then
		echo -e "\n\n!!!! The 'add_path' directive is missing on node $host"
		echo "Add it to /etc/riak/app.config"
		echo "Aborting deploy for $host"
		continue
	fi

	# Rsync .erl files into path
	ssh root@$host "mkdir -p $path && rm /tmp/erlhooks/*"
	rsync --delete-before -azv *.erl $user@$host:/tmp/erlhooks

	# Compile using shipped-in compiler
	path=`ssh $user@$host "dpkg -L riak | grep 'erlc$'"`
	ssh root@$host "cd /tmp/erlhooks && $path *.erl"
	if [[ $? -ne 0 ]]; then
		echo -e "\n\n!!!! Compiling & reloading phase failed on $host"
		continue
	else
		ssh $user@$host "rm $path/*.beam" 2>/dev/null
		ssh $user@$host "mv /tmp/erlhooks/*.beam $path && riak-admin erl-reload"
	fi
done
{% endhighlight %}



[^hooks]: [Commit Hooks](http://docs.basho.com/riak/latest/theory/concepts/#Commit-Hooks)
[^bucketprops]: [Bucket Properties](http://docs.basho.com/riak/latest/dev/references/http/set-bucket-props/)