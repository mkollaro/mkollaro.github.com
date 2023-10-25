---
layout: post
title: "OpenStack Swift 101"
published: false
---

Swift is an object storage service similar to [Amazon
S3](http://aws.amazon.com/s3/).  It mainly focuses on being extremely reliable,
not losing any of your data and horizontal scalability. 
It expects some of your disks and computers to fail
occasionally and tries not to lose any data. Your users won't even notice
when some of your servers stop working.

## Usage

As a user, you can create a _container_ (equivalent of Amazon's bucket)
and then upload files to it. That is pretty much the whole thing. An _object_ is
simply some file or any piece of data with metadata glued to it.

There is no hierarchy as in a file system, but you can 
[simulate one](http://docs.openstack.org/trunk/openstack-object-storage/developer/content/pseudo-hierarchical-folders-directories.html)
by putting
`/` into the file names and then listing only files with a certain prefix.

Use cases:
* backup service
* static page hosting (see
    [Jekyll](http://truongtx.me/2013/01/16/jekyll-bootstrap-blogging-platform-for-geeks/))
* VM snapshots backup
* photo gallery
* video storage
* storing [Glance](http://docs.openstack.org/developer/glance/) images

It tries to specialize on a few things and do them very well, therefore it
does not provide block storage. If you need that, look into
[Cinder](https://wiki.openstack.org/wiki/Cinder),
[Gluster](http://www.gluster.org/)
or [Ceph](http://ceph.com/).


## Architecture
Please keep in mind that I'm simplifying here a lot and trying to avoid jargon.
For more detailed info, look into the [Swift architecture
docs](http://docs.openstack.org/developer/swift/overview_architecture.html).

The basic idea is to keep 3 copies of each file, as far away from each other as
possible, so that a flood in a single server room does not lose all 3 copies.
Even when some data servers die, Swift will restore the copies on some other
available servers, so that there will be 3 copies available again. (The number
of copies is configurable, but I'm assuming it to be 3 for simplicity.)
  
There are two main types of Swift servers. The first one is the _proxy server_,
which in this case means "the server that knows where my data should be", not
an HTTP proxy. The other type is the _data server_, which contains the disks,
watches the data for corruption and copies them if a disk dies. You can
combine them together or even do a single node installation, but in practice
these two tend to be separated.

             +-------------------+
             |   Proxy Server    |
             +---------+---------+
                       |
              +--------+---------+
    +---------+----+     +-------+------+
    | Data Server  |     | Data Server  |
    |              |     |              |
    |+-+ +-+ +-+   |     |+-+ +-+ +-+   |
    || | | | | |   |     || | | | | |   |
    || | | | | |   |     || | | | | |   |
    |+-+ +-+ +-+   |     |+-+ +-+ +-+   |
    +--------------+     +--------------+

### The Proxy Servers
When the proxy server gets a request, it doesn't really know by itself if the
data exist in the datacenter. There is no central database that keeps track of
this and therefore no single point of failure. All it does is computing a hash
of it and decides using a [hash
ring](http://en.wikipedia.org/wiki/Consistent_hashing) where the file should
be. Then it asks the appropriate data servers if they have the file.

When you upload a file, the proxy server decides where it should go using the
hash ring. Since we want to have 3 copies of the file that are as far from each
other as possible, it will ask 3 data servers to upload it. Even if you only
have one server, it will at least put the file on different disks. If you have
set up _zones_, it will put them to different ones (I'll get to that). More
than half of the writes have to succeed, only then does the user get back a
success return code. In this case, it means two writes have to go well.

### The Data Servers
A data server is simply a Linux box with some hard drives formatted
ideally with 
[XFS](http://en.wikipedia.org/wiki/XFS).
RAID is not recommended, it isn't that helpful in this case.

There are three kinds of data servers - _account_, _container_ and _object_,
but they are usually running together on one machine. However, you can run them
separately or use some disks only for a certain service. For example, you could
make the account and container services use SSDs and leave the standard hard
disks for objects.  Account and container servers generally require higher
[IOPS](http://en.wikipedia.org/wiki/IOPS) and can take good advantage of
caching, so it makes sense to deploy them separately from object servers in
some deployments.

Swift itself doesn't manage user accounts, this is done by some authentication
service like [Keystone](http://docs.openstack.org/developer/keystone/). For
Swift, an account is just metadata and a list of container names that belong to
that user. It handles them similarly as objects, i.e., keeps three copies of
them.

A container is just metadata and a list of object names. Again, it gets handled
similarly as objects (i.e. keeps three copies of each).

### Zones
Imagine you have around 10 data servers. Half of them are on one side of the
building, the other half on the other. You want to make sure there is always at
least one copy of an object (or container, or account) on each side of the
building, so that if either part burns down in a fire, no data loss or outage
will occur. To do this, put servers from one group to the first _zone_, the rest
of them to the other _zone_. This way, when Swift tries to put the copies as
far from each other as possible, it will know that the zones are somewhat
independent and will put a copy into each.

Of course, there should be at least one proxy server on each side of the
building, nothing will work without one. However, the proxies don't really
"belong" to zones, they all have the same information. When a user makes a
request, there is no preference about which proxy will handle it, making this
an unsuitable solution for servers that are hundreds of kilometers away from
each other. 

### Regions
This is a new feature in the [Grizzly
release](http://www.openstack.org/software/grizzly/) that allows you to have
globally distributed clusters. One of your _regions_ may be in Berlin, the
other in London. Regions are location aware - this means that a user in the
UK will get served by servers in London.

Swift will keep a copy of all data in both regions, but not immediately after
writing it, since this would be too slow. At first, it will only write it
to the closest region and later move a copy to the remote region.

## Sources
* [Swift Architecture docs](http://docs.openstack.org/developer/swift/overview_architecture.html)
* [Detailed Swift Architecture by SwiftStack](http://swiftstack.com/openstack-swift/architecture/)
* [Data Placement in Swift](http://swiftstack.com/blog/2013/02/25/data-placement-in-swift/)
* [How Swift Handles Failures (video)](http://swiftstack.com/blog/2012/09/13/how-openstack-swift-handles-hardware-failures/)
* [How the Ring Works in OpenStack Swift (video)](http://swiftstack.com/blog/2012/11/21/how-the-ring-works-in-openstack-swift/)
* [A Globally Distributed OpenStack Swift Cluster](http://swiftstack.com/blog/2012/09/16/globally-distributed-openstack-swift-cluster/)
