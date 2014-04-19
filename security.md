## GlusterFS Security

So you want to enable SSL on glusterfs and you are lost? Well you are not alone - SSL mode is not documented and you can find little info about it on mailing lists. I will try to help you out with this small tips... 

So here it goes, first you need to generate SSL certificates using following commands:
```
openssl genrsa -out glusterfs.key 1024
openssl req -new -x509 -key gluster.key -subj /CN=Anyone -out glusterfs.pem
```
Now you need to move that files into proper location, gluster have that hardcoded, so until you don't want to mess with sources put them into /etc/ssl/. Next step is to create glusterfs.ca file - you do that by simply copy glusterfs.pem into glusterfs.ca. You should end-up with this files in /etc/ssl/:
```
glusterfs.ca
glusterfs.key
glusterfs.pem
```
Now, let's finally enable SSL mode on the volume. Do it by setting following parameters on volume:
```
gluster volume set gv0 client.ssl on
gluster volume set gv0 server.ssl on
```
Verify:
```
gluster volume info gv0

Volume Name: gv0
Type: Replicate
Volume ID: c9205800-11e7-491d-be9b-d695098beddc
Status: Started
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: mx-1:/export/brick1
Brick2: mx-2:/export/brick1
Options Reconfigured:
server.ssl: on
client.ssl: on
```

stop gv0, restart glusterd, start gv0 and to be sure that SSL is working, checkout glustershd.log log, it should read:
```
[socket.c:3480:socket_init] 0-gv0-client-0: SSL support is ENABLED
```
repeat that procedure on all nodes, that's all!
