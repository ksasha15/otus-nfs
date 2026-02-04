# otus-nfs

**Задание**
запустить 2 виртуальных машины (сервер NFS и клиента);
на сервере NFS должна быть подготовлена и экспортирована директория; 
в экспортированной директории должна быть поддиректория с именем upload с правами на запись в неё; 
экспортированная директория должна автоматически монтироваться на клиенте при старте виртуальной машины (systemd, autofs или fstab — любым способом);
монтирование и работа NFS на клиенте должна быть организована с использованием NFSv3.

u24@nfss:~$ apt install nfs-kernel-server
E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
u24@nfss:~$ sudo -i
[sudo] password for u24:
root@nfss:~# apt install nfs-kernel-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libnfsidmap1 nfs-common rpcbind
..............

root@nfss:~# ss -tnlpu
Netid      State       Recv-Q      Send-Q           Local Address:Port            Peer Address:Port     Process
udp        UNCONN      0           0                    127.0.0.1:690                  0.0.0.0:*         users:(("rpc.statd",pid=2210,fd=5))
udp        UNCONN      0           0                      0.0.0.0:41675                0.0.0.0:*         users:(("rpc.mountd",pid=2218,fd=8))
udp        UNCONN      0           0                   127.0.0.54:53                   0.0.0.0:*         users:(("systemd-resolve",pid=783,fd=16))
udp        UNCONN      0           0                127.0.0.53%lo:53                   0.0.0.0:*         users:(("systemd-resolve",pid=783,fd=14))
udp        UNCONN      0           0                      0.0.0.0:53330                0.0.0.0:*
udp        UNCONN      0           0                      0.0.0.0:111                  0.0.0.0:*         users:(("rpcbind",pid=1663,fd=5),("systemd",pid=1,fd=88))
udp        UNCONN      0           0                      0.0.0.0:42387                0.0.0.0:*         users:(("rpc.mountd",pid=2218,fd=4))
udp        UNCONN      0           0                      0.0.0.0:40452                0.0.0.0:*         users:(("rpc.statd",pid=2210,fd=8))
udp        UNCONN      0           0                      0.0.0.0:57872                0.0.0.0:*         users:(("rpc.mountd",pid=2218,fd=12))
udp        UNCONN      0           0                         [::]:39502                   [::]:*         users:(("rpc.mountd",pid=2218,fd=10))
udp        UNCONN      0           0                         [::]:34547                   [::]:*         users:(("rpc.statd",pid=2210,fd=10))
udp        UNCONN      0           0                         [::]:111                     [::]:*         users:(("rpcbind",pid=1663,fd=7),("systemd",pid=1,fd=90))
udp        UNCONN      0           0                         [::]:54506                   [::]:*
udp        UNCONN      0           0                         [::]:58680                   [::]:*         users:(("rpc.mountd",pid=2218,fd=6))
udp        UNCONN      0           0                         [::]:47537                   [::]:*         users:(("rpc.mountd",pid=2218,fd=14))
tcp        LISTEN      0           4096             127.0.0.53%lo:53                   0.0.0.0:*         users:(("systemd-resolve",pid=783,fd=15))
tcp        LISTEN      0           4096                   0.0.0.0:51431                0.0.0.0:*         users:(("rpc.mountd",pid=2218,fd=5))
tcp        LISTEN      0           4096                127.0.0.54:53                   0.0.0.0:*         users:(("systemd-resolve",pid=783,fd=17))
tcp        LISTEN      0           4096                   0.0.0.0:111                  0.0.0.0:*         users:(("rpcbind",pid=1663,fd=4),("systemd",pid=1,fd=87))
tcp        LISTEN      0           4096                   0.0.0.0:22                   0.0.0.0:*         users:(("sshd",pid=1320,fd=3),("systemd",pid=1,fd=124))
tcp        LISTEN      0           64                     0.0.0.0:2049                 0.0.0.0:*
tcp        LISTEN      0           4096                   0.0.0.0:36545                0.0.0.0:*         users:(("rpc.mountd",pid=2218,fd=9))
tcp        LISTEN      0           4096                   0.0.0.0:55963                0.0.0.0:*         users:(("rpc.mountd",pid=2218,fd=13))
tcp        LISTEN      0           64                     0.0.0.0:39857                0.0.0.0:*
tcp        LISTEN      0           4096                   0.0.0.0:49981                0.0.0.0:*         users:(("rpc.statd",pid=2210,fd=9))
tcp        LISTEN      0           4096                      [::]:53493                   [::]:*         users:(("rpc.mountd",pid=2218,fd=15))
tcp        LISTEN      0           4096                      [::]:111                     [::]:*         users:(("rpcbind",pid=1663,fd=6),("systemd",pid=1,fd=89))
tcp        LISTEN      0           4096                      [::]:22                      [::]:*         users:(("sshd",pid=1320,fd=4),("systemd",pid=1,fd=128))
tcp        LISTEN      0           64                        [::]:2049                    [::]:*
tcp        LISTEN      0           4096                      [::]:37575                   [::]:*         users:(("rpc.mountd",pid=2218,fd=11))
tcp        LISTEN      0           64                        [::]:41569                   [::]:*
tcp        LISTEN      0           4096                      [::]:53127                   [::]:*         users:(("rpc.statd",pid=2210,fd=11))
tcp        LISTEN      0           4096                      [::]:45925                   [::]:*         users:(("rpc.mountd",pid=2218,fd=7))
root@nfss:~#
root@nfss:~# mkdir -p /srv/share/upload
root@nfss:~# chown -R nobody:nogroup /srv/share
root@nfss:~# chmod 0777 /srv/share/upload/
root@nfss:~# cat << EOF > /etc/exports
> /srv/share/ 192.168.50.11/32(rw,sync,root_squash)
> EOF
root@nfss:~# export
export    exportfs
root@nfss:~# exportfs -r
exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "192.168.50.11/32:/srv/share/".
  Assuming default behaviour ('no_subtree_check').
  NOTE: this default has changed since nfs-utils version 1.0.x

root@nfss:~# exportfs -s
/srv/share  192.168.50.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
root@nfss:~#

**Настраиваем NFS на клиенте**
u24@nfsc:~$ sudo -i
sudo: unable to resolve host nfsc: Name or service not known
[sudo] password for u24:
root@nfsc:~# sudo apt install nfs-common
sudo: unable to resolve host nfsc: Name or service not known
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  keyutils libnfsidmap1 rpcbind
......
root@nfsc:~# echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0" >> /etc/fstab
root@nfsc:~# more /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-528VgFKAoBCZw0yk2Myf1mUOc8IAYPhddaM3izdzJQBBV9DHXSk2K
U71d8mKbjCF / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/28f4a2f6-da12-4aec-bc25-decc4629eed8 /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
192.168.50.10:/srv/share/ /mnt nfs vers=3,noauto,x-systemd.automount 0 0
root@nfsc:~#
root@nfsc:~# systemctl daemon-reload
root@nfsc:~# systemctl restart remote-fs.target
root@nfsc:~#
root@nfsc:~# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=72,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=20890)
root@nfsc:~# ll /mnt
total 12
drwxr-xr-x  3 nobody nogroup 4096 Feb  4 14:04 ./
drwxr-xr-x 23 root   root    4096 Jan 28 14:55 ../
drwxrwxrwx  2 nobody nogroup 4096 Feb  4 14:04 upload/
root@nfsc:~# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=72,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=20890)
192.168.50.10:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=192.168.50.10,mountvers=3,mountport=57872,mountproto=udp,local_lock=none,addr=192.168.50.10)
root@nfsc:~#



