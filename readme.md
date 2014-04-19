# GlusterFS

GlusterFS is a file system that is designed to provide network storage that can be made redundant, 
fault-tolerant and scalable. It’s particularly well suited to applications that require high-performance 
access to large files.

You can check out [GlusterFS's community Website here](http://www.gluster.org/).

***

## Quick Start Guide

#### Step 1 – Have at least two nodes

#### Step 2 - Format and mount the bricks

#### Step 3 - Installing GlusterFS

#### Step 4 - Configure SELinux and iptables

#### Step 5 - Configure the trusted pool

#### Step 6 - Set up a GlusterFS volume

#### Step 7 - Testing the GlusterFS volume

***
## Performance

We are using GlusterFS to replicate storage between two physical servers for two reasons; load balancing and data redundancy. With GlusterFS we also have several ways to improve performance but before we look into those, we need to be sure that is it the GlusterFS layer which is causing the problem. For example, if your disks or network is slow, what chance does GlusterFS have of giving you good performance? You also need to understand how the individual components work under the load of your expected environment. The disks may work perfectly well when you use dd to create a huge file, but what about when lots of users create lots of files all at the same time? You can break down performance into three key areas:

* Networking – the network between each GlusterFS instance.
* Filesystem IO performance - the file system local to each GlusterFS instance.
* GlusterFS – the actual GlusterFS process.

[More about GlusterFS performance](performance.md).

***
## Security
So you want to enable SSL on GlusterFS and you are lost? Well you are not alone - SSL mode is not documented and you can find little info about it on mailing lists. I will try to help you out [with these small tips](security.md).
***
## Troubleshooting
***
## FAQ
***
## Changes
***
#### Resources
[Documenting the undocumented](http://www.gluster.org/community/documentation/index.php/Documenting_the_undocumented)
