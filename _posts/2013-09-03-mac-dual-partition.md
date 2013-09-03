---
layout: post
title: "Dualboot on a Mac with FusionDrive"
category: posts
---

It was hard for me to find some info about this, since the CoreStorage commands are **not documented** in `man diskutil`, so I thought I'd share all this.

This is how I split my main volume in order to have a Windows Dual Boot.



First of all, list your CoreStorage logical volumes using 

{% highlight bash %}
diskutil cs list
{% endhighlight %}

Then, find the Logical Volume you want to shrink, the line should be something like :

`Logical Volume XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX`

Get that identifier, and simply do 

{% highlight bash %}
sudo diskutil cs resizeStack XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX 215g NTFS-3G WINDOWS 35g
{% endhighlight %}

Where : 

* `XXXXX....` is the identifier for the volume to shrink
* `215g` is the size of that volume after being shrinked
* `NTFS-3G` is the file system of the new volume being created
* `35g` is the size you wish to allocate to your new volume

You're now good to go !

## References
**resizeStack hidden documentation** :

{% highlight text %}
Resize both a logical volume and its underlying physical volume in a single
operation. The setup must be simple: Exactly one logical volume and one
related physical volume can, and must, exist.
If this is a shrink operation, you can optionally request that new partitions
be created in the newly-formed free space gap.
{% endhighlight %}

[This awesome blog post : "Undocumented corestorage commands"](http://blog.fosketts.net/2011/08/05/undocumented-corestorage-commands/)

[This gist : "OS X Lion diskutil commands (documented and hidden).sh"](https://gist.github.com/mralexgray/2979512)