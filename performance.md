## Networking Performance

Before testing the disk and file system, it’s a good idea to make sure that the network connection between 
the GlusterFS nodes is performing as you would expect. 
Test the network bandwidth between all GlusterFS boxes using Iperf. 
See the [Iperf blog post](http://www.jamescoyle.net/how-to/574-testing-network-speed-with-iperf) for more information on benchmarking network performance. 
Remember to test the performance over a period of several hours to minimise the affect of host and network load. 
If you make any network changes, remember to test between each change to make sure it has had the desired effect.

## Filesystem IO Performance

Once you have tested the network between all GlusterFS boxes, you should test the local disk speed on each machine. There are several ways to do this, but I find it’s best to keep it simple and use one of two options; DD or bonnie++. You must be sure to turn off any GlusterFS replication as it is just the disks and filesystem which we are trying to test here. [Bonnie++](http://www.coker.com.au/bonnie++/) is a freely available IO benchmarking tool.  DD is a linux command line tool which can replicate data streams and copy files. See [this blog post](http://www.jamescoyle.net/how-to/599-benchmark-disk-io-with-dd-and-bonnie) for information on benchmarking the files system.

## Technology, Tuning and GlusterFS

Once we have made it certain in our minds that disk I/O and network bandwidth are not the issue, or more importantly understood what constraints they give you in your environment, you can tune everything else to maximize performance. In our case, we are trying to maximize GlusterFS replication performance over two nodes.

We can aim to achieve replication speeds nearing the speed of the the slowest performing speed; file system IO and network speeds.

See [this blog post on GlusterFS performance tuning](http://www.jamescoyle.net/how-to/559-glusterfs-performance-tuning%20%E2%80%8E).
