# Install from RPM with YUM

```
cd /etc/yum.repos.d/
wget http://download.gluster.org/pub/gluster/glusterfs/LATEST/EPEL.repo/glusterfs-epel.repo

yum install glusterfs{-fuse,-server}
service glusterd start
chkconfig glusterd on
```
