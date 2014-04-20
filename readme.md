# GlusterFS

GlusterFS is a distributed files system that is designed to provide network storage that can be made redundant, 
fault-tolerant and horizontally scalable. It’s particularly well suited to applications that require high-performance 
access to large files. Gluster File System allows you to create a single volume of storage which spans multiple disks, multiple machines and even multiple data centers.

Gluster it's easy to implement and work with, limited only by the system resources you can dedicate to it, and doesn't require you to buy any expensive vendor-specific hardware.

Gluster's storage is build up by what it calls bricks, which are exported directories allocated on the cluster nodes. Cluster nodes are united in trusted pools that together provide storage services and share disk resources.

***
Here's how you can build a Gluster distributed storage system yourself.
## Quick Start Guide

#### Step 1 - Installing GlusterFS

GlusterFS uses a server -> client architecture and in this example we are going to use two machines, both as server and client. This means that if one server fails the other can carry on functioning independently. The two servers are:

```
# Server 1
192.168.0.10     server1.example.com

# Server 2
192.168.0.20     server2.example.com
```

There are a couple reasons to use host names instead of IPs when you configure you node clusters.  The first one is obvious:  If the machine’s IP changes, then you’ll have to update each machine’s configuration manually to reflect the new IP.  When you attach a peer in gluster, it stores either the host name or the IP in the peers configuration file /var/lib/glusterd/peers/<UUID>.  

Second reason, and this is a big one, when a gluster client connects to a particular volume a certain manifest file is downloaded to the client.  

Third reason, and this one is for future planning and scalability:  By using host names in the peer probing process, this allows clients and servers alike to use non-uniform IP accessing to the cluster.  If all clients and all cluster nodes are on the same subnet, by default all traffic will flow ever the same interface.  In a replicated cluster setup, you obviously don’t want the replication traffic riding on the same links as the production traffic.  This will negatively impact read/write operations to your cluster if you’re saturating your network.  Since we’re talking about a scale-out storage system, I’m guessing performance is a big factor for your production traffic, and this should be a no-brainer.

Everything we do in this example needs to be run as a root user, so rather than typing sudo before all commands we can just log in as the root user: `sudo su`

Since a client downloads a gluster manifest file that utilizes host names, the client can resolve those host names to whichever IP the client wishes, either through DNS or the hosts file.  So, on our clusters, we have entries in /etc/hosts as follows:

```
$ vi /etc/hosts
172.1.1.1     gfs1
172.1.1.2     gfs2
```

If you are using servers that will be communicating over the public network you will need to enable these ports 111, 24007, 24008, 24009 (24009 + number of volumes) in your firewall for all servers.

Install Gluster from its own yum repository, which always provides the latest Gluster version. First, download Gluster's repository with the command wget -P /etc/yum.repos.d http://download.gluster.org/pub/gluster/glusterfs/LATEST/EPEL.repo/glusterfs-epel.repo. This command places the repo file in the directory /etc/yum.repos.d/, thus enabling the repo.

Now you can install the packages glusterfs-fuse, the userspace program for managing GlusterFS, and glusterfs-server, the server daemon, with the command yum install glusterfs{-fuse,-server}.

Finally, start Gluster's daemon for the first time with the command service glusterd start. To make it start and stop automatically with the system, run the command chkconfig glusterd on.

Perform this on both of your servers. If you have more than two servers, perform this command on all of the servers required for the volume.

You will now need each of these servers to know about the others.

#### Step 2 - Configure the trusted pool

Gluster unites cluster nodes into a trusted pool, and all nodes inside this pool share disk resources and offer storage services. Acting as one logical entity, a trusted pool provides redundancy, better storage performance, and scalability.

To manage a new trusted pool, and in fact manage all parts of Gluster, use the Gluster console manager at /usr/sbin/gluster. This command-line tool accepts arguments for options; there is no GUI.

A storage pool is created automatically when you install and start Gluster on the first cluster node. From the first node you can add the rest of the nodes after you install and start Gluster on them.

For example, suppose you installed Gluster on a node with the IP address 10.1.1.1 and you want to add to the pool a node with IP 10.1.1.2. To accomplish this use the Gluster console manager with the arguments peer probe [host], as in /usr/sbin/gluster peer probe 10.1.1.2.

Each of the commands should return with Probe successful which means the node is now known to this machine. You will only need to do this on one node of your cluster.

Run `gluster peer status` to check each node in your cluster is aware of the other nodes.

Similarly, you can remove servers from the storage pool with the command /usr/sbin/gluster peer detach [host].

After adding servers to or removing them from the trusted storage pool, you should check the pool's status. Use the command /usr/sbin/gluster peer status to confirm you have the correct number of peers and their statuses.

#### Step 3 - Format and mount the bricks

To comply with best practices, use one disk for the operating system and dedicate one or more others for Gluster's storage only. Also, format the future Gluster storage disk with the XFS filesystem, which is fast, scalable, and reliable for distributed storage.

So far you've installed Gluster on one disk and have just attached another for Gluster storage. The new disk is not formatted and appears under the name /dev/sdb. You can acknowledge the disk with the command fdisk -l, which should show the new disk with valid partitions.

To prepare the disk, first use fdisk to create a primary partition on the new disk. Run the command fdisk /dev/sdb and select "n" for new partition, "p" for primary, "1" for first. Then write the new partition table to the disk by selecting "w." You should now have a new primary partition under the name /dev/sdb1.

Next, format the new partition with the XFS filesystem with the command mkfs.xfs /dev/sdb1. Create a new directory to be used as a mount point for the /dev/sdb1 partition, such as /export/brick1. Brick1 is a good name because this partition will be used for the first brick, which is to say the first exported directory found on the first node.

Finally, make sure the new partition /export/brick1 is automatically mounted during system boot time by adding a new row to /etc/fstab: /dev/sdb1 /export/brick1 xfs defaults 1 2.

Now we need to create the volume where the data will reside. the volume will be called datastore. First of all, we need to identify where on the host this storage is. For this example, it is /mnt/gfs_block on both nodes, but this could be any mount point of storage that you have. If the folder does not exist, it will be silently created so be sure to get the correct path on all nodes:
```
gluster volume create datastore replica 2 transport tcp gfs1:/data/gfs_block gfs2:/data/gfs_block
```
If this has been sucessful, you should see:
```
Creation of volume testvol has been successful. Please start the volume to access data.
```
As the message indicates, we now need to start the volume:
```
gluster volume start datastore
```
Running either of the below commands should indicate that GlusterFS is up and running. The ps command should show the command running with both servers in the argument. netstat should show a connection between both nodes.
```
ps aux | grep gluster
netstat -tap | grep glusterfsd
```
As a final test, to make sure the volume is available, run `gluster volume info`.

***
## Tips and Tricks

#### Extend volume for size - replicated-distributed volume

To extend amount of disk space available on volume www add a brick on server3 and server4 (multiple of the replica count):
```
gluster volume add-brick www gfs3:/data/export/www gfs4:/var/export/www
```

This is now a replicated-distributed volume (2x2) and twice of disk space is available.

#### Remove bricks

In case you created files on this volume you need to know that part of the data on will be lost.  
```
gluster volume remove-brick www gfs3:/data/export/www gfs4:/var/export/www
```

Remove directory /var/export/www on gfs3 and gfs4

#### Extend volume for replica count

Stop volume www: 
```gluster volume stop www```
Delete volume www: 
```gluster volume delete www```
Recreate volume but increase replica count to 4 and define all four bricks:
```
gluster volume create www replica 4 transport tcp server1:/var/export/www server2:/var/export/www server3:/var/export/www server4:/var/export/www
```
Start volume www and set options:
```
gluster volume start www
gluster volume set www auth.allow 192.168.56.*
```
Check volume status:
```gluster volume info www```
Volume data should become available again.

#### Migrate brick
Bricks can be easilly moved between servers. To move brick /var/export/www (volume www) from server4 to server5 execute commands below.  
Start migration of server4:/var/export/www brick to server4:/var/export/www:  
```gluster volume replace-brick server4:/var/export/www server5:/var/export/www start```   
Check status of above migration:  
```gluster volume replace-brick server4:/var/export/www server5:/var/export/www status```   
After the migration is done you need to commit it:  
```gluster volume replace-brick server4:/var/export/www server5:/var/export/www commit```   
Check volume status:  
```gluster volume info www```   

#### How to manage Gluster volumes

A Gluster volume is a logical entity composed of bricks, which are exported directories from servers inside the trusted pool. One pool can support numerous volumes. You can start managing Gluster volumes as soon as you have a trusted storage pool set up.

Gluster supports many advanced options for its volumes. For instance, you can create a distributed, striped, and replicated volume. Such a volume performs well, first because it's distributed – reads and writes happen on multiple servers. Second, the fact that it is striped ensures that large files are striped across multiple bricks and servers and thus accessed faster. High availability is ensured by the fact that the volume and its data is replicated and redundant, so if one node fails another covers for it and there is no interruption in service.

To create a distributed, striped, and replicated volume use the command /usr/sbin/gluster volume create volumename [stripe count] [replica count] host:/brickname. An example of the command would look like: /usr/sbin/gluster volume create testvolume stripe 2 replica 2 10.1.1.1:/export/brick1 10.1.1.2:/export/brick2. This creates a volume called "testvolume" striped across two bricks, replicated twice. The volume consists of two nodes and their bricks – 10.1.1.1:/export/brick1 and 10.1.1.2:/export/brick2.

Once you have a Gluster volume you can easily and seamlessly change its size without interrupting the storage operations. For example, if you want to increase the size of your volume you can add more cluster nodes with more bricks attached to them. First, add each additional node with the command /usr/sbin/gluster peer probe host. Then use the command /usr/sbin/gluster volume add-brick volumename host:/brickname. Finally, rebalance and redistribute the volume evenly across all the bricks with the command /usr/sbin/gluster volume rebalance volumename start.

You need to keep a few concepts in mind for proper volume management and configuration. First, you should use only one brick per server in a given volume so that in case of a crash it can be covered for by its replicating peer. Second, the number of bricks should be a multiple of the replica number – that is, how many times data should be replicated – so that valid replication can be established. For example, if you want data to be replicated twice, you should build your cluster from any number of bricks that's a multiple of two. Last, the number of stripes should be equal to the number of bricks so that you can gain performance by reading from and writing to multiple locations simultaneously.

#### Connect a client to the storage

The best way to connect to Gluster storage is through GlusterFS, which allows clients to take advantage of Gluster's clustering and high reliability features. GlusterFS offers better performance and reliability than non-native communication methods such as NFS or iSCSI, and GlusterFS allows clients to communicate simultaneously with multiple cluster nodes.

GlusterFS is based on filesystem in userspace (FUSE). Prior to installing GlusterFS you have to install the fuse and fuse-libs packages. In CentOS these prerequisite packages can be installed through the official repository with the command yum install fuse fuse-libs. You must run this installation only on the clients from which you will connect to the storage; FUSE is automatically installed with the Gluster server software.

Restart the server after installing the fuse package in order to load the fuse kernel module, or if you want to postpone the restart you can manually load the module using the command modprobe fuse.

Next, to install GlusterFS, you have to add Gluster's repository just as you added it on the cluster nodes, with the command wget -P /etc/yum.repos.d http://download.gluster.org/pub/gluster/glusterfs/LATEST/EPEL.repo/glusterfs-epel.repo, and install glusterfs-fuse with the command yum install glusterfs-fuse.

Once you've installed all the necessary software you can mount a GlusterFS volume. You can use the /bin/mount command with "glusterfs" as parameter for the filesystem (-t option) and a few other GlusterFS-specific parameters; for example, mount -t glusterfs -o backupvolfile-server=10.1.1.2 10.1.1.1:/testvolume /media/gluster.

This example mounts the volume "testvolume" from the Gluster node 10.1.1.1. The option backupvolfile-server instructs the client to work with the replicating Gluster node 10.1.1.2 in case 10.1.1.1 fails.

Once the volume is mounted, Linux treats it just as any other disk resource. Multiple clients can mount the same volume and work with it at the same time.

#### Synchronise to a remote server using geo replication

GlusterFS can be used to synchronise a directory to a remote server on a local network for data redundancy or load balancing to provide a highly scalable and available file system.

The problem is when the storage you would like to replicate to is on a remote network, possibly in a different location, GlusterFS does not work very well. This is because GlusterFS is not designed to work when there is a high latency between replication nodes.

GlusterFS provides a feature called geo replication to perform batch based replication of a local volume to a remote machine over SSH.

The below example will use three servers:

* gfs1.jamescoyle.net is one of the two running GlusterFS volume servers.
* gfs2.jamescoyle.net is the second of the two running GlusterFS volume servers. gfs1 and gfs2 both server a single GlusterFS replicated volume called datastore.
* remote.jamescoyle.net is the remote file server which the GlusterFS volume will be replicated to.
* 
GlusterFS uses an SSH connection to the remote host using SSH keys instead of passwords. We’ll need to create an SSH key using ssh-keygen to use for our connection. Run the below command and press return when asked to enter the passphrase to create a key without a passphrase.

```
ssh-keygen -f /var/lib/glusterd/geo-replication/secret.pem
```

Now you need to copy the public certificate to your remote server in the authorized_keys file. The remote user must be a super user (currently a limitation of GlusterFS) which is root in the below example. If you have multiple GlusterFS volumes in a cluster then you will need to copy the key to all GlusterFS servers.

```
cat /var/lib/glusterd/geo-replication/secret.pem.pub | ssh root@remote.jamescoyle.net "cat >> ~/.ssh/authorized_keys"
```

Make sure the remote server has glusterfs-server installed. Run the below command to install glusterfs-server on remote.jamescoyle.net.

Create a folder on remote.jamescoyle.net which will be used for the remote replication. All data which transferrs to this machine will be stored in this folder.

```
mkdir /gluster
mkdir /gluster/geo-replication
```

Create the geo-replication volume with Gluster and replace the below values with your own:

* [SOURCE_DATASTORE] – is the local Gluster data volume which will be replicated to the remote server.
* [REMOTE_SERVER] – is the remote server to receive all the replication data.
* [REMOATE_PATH] – is the path on the remote server to store the files.

```
gluster volume geo-replication [SOURCE_DATASTORE] [REMOTE_SERVER]:[REMOTE_PATH] start
```

Example:

```
gluster volume geo-replication datastore remote.jamescoyle.net:/gluster/geo-replication/ start
 
Starting geo-replication session between datastore & remote.jamescoyle.net:/gluster/geo-replication/ has been successful
```

Sometimes on the remote machine, gsyncd (part of the GlusterFS package) may be installed in a different location to the local GlusterFS nodes.

Your log file may show a message similar to below:

```
Popen: ssh> bash: /usr/lib/x86_64-linux-gnu/glusterfs/gsyncd: No such file or directory
```

In this scenario you can specify the config command the remote gsyncd location.

```
gluster volume geo-replication datastore remote.jamescoyle.net:/gluster/geo-replication config remote-gsyncd /usr/lib/glusterfs/glusterfs/gsyncd
```

You will then need to run the start command to start the volume synchronisation.

```
gluster volume geo-replication datastore remote.jamescoyle.net:/gluster/geo-replication/ start
```

You can view the status of your replication task by running the status command.

```
gluster volume geo-replication datastore remote.jamescoyle.net:/gluster/geo-replication/ status
```

You can stop your volume replication at any time by running the stop command.

```
gluster volume geo-replication datastore remote.jamescoyle.net:/gluster/geo-replication/ stop
```













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

When you create a new GlusterFS Volume it is publicly available for any server on the network to read. For optimal performance and security you should run the Gluster cluster inside a private and secured network. In such a network you can disable the default CentOS firewall because the network should be accessible only by trusted clients. To do this, use the command `chkconfig iptables off && service iptables stop`.

File servers do not generally have firewalls as they are hosted in a secure zone of a private network. Just because it’s secure doesn’t mean you should leave it wide open for anyone with access to connect to.

Using the auth.allow and auth.reject arguments in GlusterFS we can choose which IP addresses can access the volume. Access is provided at volume level, therefore you will need to alter access permissions on every new volume you create.

Run the below command on each server changing [VOLUME] to match the volume to be accessed and [IP ADDRESS] to be an IP address of the server which can connect to the current server.
```
gluster volume set [VOLUME] auth.allow [IP ADDRESS]
```
[IP ADDRESS] does not have to be a single IP address. You can also use an asterisk [*] as a wildcard, or multiple addresses separated by a comma [,]. The below example allows only servers with an IP address on the 10.1.1.x range, and 10.5.5.1 to access volume datastore.. All other servers will be denied access to the volume.
```
gluster volume set datastore auth.allow 10.1.1.*,10.5.5.1
```

If you can, your storage servers should be in a secure zone in your network removing the need to firewall each machine. Inspecting packets incurs an overhead, not something you need on a high performance file server so you should not run a file server in an insecure zone.

If you cannot provide a private network for the storage cluster, you can use iptables to allow only predefined clients to access Gluster's services. You will need to allow several ports for GlusterFS to communicate with clients and other servers. The following ports are all TCP (please note that brick ports have changed since version 3.4):

* 24007 – Gluster Daemon
* 24008 – Management
* 24009 and greater (GlusterFS versions less than 3.4) OR
* 49152 (GlusterFS versions 3.4 and later) - Each brick for every volume on your host requires it’s own port. For every new brick, one new port will be used starting at 24009 for GlusterFS versions below 3.4 and 49152 for version 3.4 and above. If you have one volume with two bricks, you will need to open 24009 – 24010 (or 49152 – 59153).
* 38465 – 38467 - this is required if you by the Gluster NFS service.
The following ports are TCP and UDP:
* 111 – portmapper

To do so, allow the following with iptables:

RPC incoming connectivity – Gluster's processes communicate using RPC, so RCP's port, TCP and UDP port 111, has to be allowed for incoming connections. Use the commands `iptables -I INPUT -m state --state NEW -m tcp -p tcp -s X.X.X.X/24 --dport 111 -j ACCEPT; iptables -I INPUT -m state --state NEW -m udp -p udp -s X.X.X.X/24 --dport 111 -j ACCEPT;`.

Gluster's own services – For Gluster to access its bricks on each cluster node, you have to allow TCP ports 24007, 24008, and 24009, plus an additional number of ports in sequence equivalent to the number of bricks across all volumes. Thus if you have two bricks in the cluster, you have to allow ports from 24007 to 24011 (24009 plus 2). The command for this is `iptables -A INPUT -m state --state NEW -m tcp -p tcp -s X.X.X.X/24 --dport 24007:24011 -j ACCEPT`.

Other access – Allow other remote connections if you want to use NFS, iSCSI, or other connectivity. You don't have to allow other ports if you plan to mount volumes with the native GlusterFS filesystem.
The above rules allow all cluster nodes and clients from the X.X.X.X C-class network to communicate. You can also apply these rules on a per-host basis by using Y.Y.Y.Y for the IP address of each host instead of X.X.X.X/24 for the network.

Don't forget to save the iptables configuration once you've added the new rules by using the command `service iptables save`.

If you want to enable SSL on GlusterFS (not well documented and supported) [here are some tips](security.md).

***
## Troubleshooting
***
## FAQ
***
## Changes
***
#### Resources
You can check out [GlusterFS's community Website here](http://www.gluster.org/).  
[Documenting the undocumented](http://www.gluster.org/community/documentation/index.php/Documenting_the_undocumented)
***
#### References
http://www.openlogic.com/wazi/bid/284663/Create-distributed-storage-with-Gluster   
http://funwithlinux.net/2013/02/glusterfs-tips-and-tricks-centos/  
http://superrb.com/blog/2011/10/14/high-availability-file-system-for-load-balanced-webservers-with-glusterfs-and-ubuntu
