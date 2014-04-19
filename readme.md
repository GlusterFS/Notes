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
## Tips and Tricks

####Problem: Attaching a Peer If you have cloned a virtual machine

If you thought you would save some time configuring each system by just performing all the necessary installation steps on a vm, and then cloning it, you’re in trouble! You will be able to attach your peer, but you won’t be able to create any sort of volume using the two nodes.

When glusterfs-server package is first installed, it creates a node UUID file at /var/lib/glusterd/glusterd.info.
So, when gluster resolves the hostname to a UUID, it creates a conflict. As a result of the VM cloning, both nodes have the same UUID.  Since gluster seems to perform it’s operations based on peer UUIDs, it’s impossible to remove using the gluster command.

**Solution:** Stop the gluster service on the nodes.  On node2, rm the glusterd.info file.  Next, we have to manually hack the peer out of the configs.  Simply and remove the file that shares the UUID in the folder /var/lib/glusterd/peers/. Upon restarting of the service, gluster will create a new UUID for the system, and the problematic ‘node2 is localhost’ issue will be resolved for good.

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
***
#### References
http://funwithlinux.net/2013/02/glusterfs-tips-and-tricks-centos/  
http://superrb.com/blog/2011/10/14/high-availability-file-system-for-load-balanced-webservers-with-glusterfs-and-ubuntu
