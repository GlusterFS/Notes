## Networking Performance

Before testing the disk and file system, it’s a good idea to make sure that the network connection between 
the GlusterFS nodes is performing as you would expect. 
Test the network bandwidth between all GlusterFS boxes using Iperf. 
See the [Iperf blog post](http://www.jamescoyle.net/how-to/574-testing-network-speed-with-iperf) for more information on benchmarking network performance. 
Remember to test the performance over a period of several hours to minimise the affect of host and network load. 
If you make any network changes, remember to test between each change to make sure it has had the desired effect.

There are a few tools available to measure network performance metrics such as throughput, packet loss and jitter (sometimes known as delay variation). One of these tools is called iperf.

### iperf
iperf is a tool for active measurements of the maximum achievable bandwidth on IP networks. It supports tuning of various parameters related to timing, protocols, and buffers. For each test it reports the bandwidth, loss, and other parameters.

iperf3 is a new implementation from scratch, with the goal of a smaller, simpler code base, and a library version of the functionality that can be used in other programs. iperf3 also a number of features found in other tools such as nuttcp and netperf, but were missing from the original iperf. These include, for example, a zero-copy mode and optional JSON output.

The basics of using iperf are simple. Install it on a server and a client, then run `iperf -s` on the server and `iperf -c <IP address of server>` on the client. The -s option signifies server, whilst the -c option indicates client. You need to specify the IP address of the server to connect to with the -c option. Running iperf with -h prints out the options. The -i option defines the interval in seconds that iperf reports the metrics. The default time that the iperf client will run is 10 seconds, but this can be changed using the -t option. By default, the iperf server listens on port 5001, but this can be changed if required.

### Installing iperf3

Github repo here: https://github.com/esnet/iperf

To check out the most recent code, do `git clone https://github.com/esnet/iperf.git`
```
./configure
make
make install
```
(Note: If configure fails, try running ./bootstrap.sh first)

Once installed, on the remote host run iperf3 in client mode. If you wish to run the server in daemon mode, add -D to the command. 

```
iperf3 -s -i 1
```

iperf has many configurable options for testing network throughput. For our test, we will use TCP connections to a remote server at <IP address of server>. The test will use 4 threads, each sending data and the test will be performed in both directions.

```
iperf3 -c <IP address of server> -i 1 -t 60
```

## Filesystem IO Performance

Once you have tested the network between all GlusterFS boxes, you should test the local disk speed on each machine. There are several ways to do this, but I find it’s best to keep it simple and use one of two options; DD or bonnie++. You must be sure to turn off any GlusterFS replication as it is just the disks and filesystem which we are trying to test here. [Bonnie++](http://www.coker.com.au/bonnie++/) is a freely available IO benchmarking tool.  DD is a linux command line tool which can replicate data streams and copy files. See [this blog post](http://www.jamescoyle.net/how-to/599-benchmark-disk-io-with-dd-and-bonnie) for information on benchmarking the files system.

## Technology, Tuning and GlusterFS

Once we have made it certain in our minds that disk I/O and network bandwidth are not the issue, or more importantly understood what constraints they give you in your environment, you can tune everything else to maximize performance. In our case, we are trying to maximize GlusterFS replication performance over two nodes.

We can aim to achieve replication speeds nearing the speed of the the slowest performing speed; file system IO and network speeds.

See [this blog post on GlusterFS performance tuning](http://www.jamescoyle.net/how-to/559-glusterfs-performance-tuning%20%E2%80%8E).
