---
title: "How to Repair File System Provided From Kubernetes Ceph Persistent Volume"
date: 2023-05-06T09:29:38+08:00
draft: true
tags: ["Kubernetes", "Ingress", "CSI"]
---
# How to repair file system provided from Kubernetes ceph persistent volume 
某日由於底層VM出現硬體故障，整座自建的Kubernetes接受影響，
部署於該環境的應用MySQL無法掛載pv，從pod出現的錯誤訊息如下:
```bash
MountVolume.SetUp failed for volume "pvc-78cf22fa-d776-43a4-98d7-d594f02ea018" : mount command failed, status: Failure, reason: failed to mount volume /dev/rbd13 [xfs] to /var/lib/kubelet/plugins/ceph.rook.io/rook-ceph/mounts/pvc-78cf22fa-d776-43a4-98d7-d594f02ea018, error mount failed: exit status 32 Mounting command: systemd-run Mounting arguments: --description=Kubernetes transient mount for /var/lib/kubelet/plugins/ceph.rook.io/rook-ceph/mounts/pvc-78cf22fa-d776-43a4-98d7-d594f02ea018 --scope -- mount -t xfs -o rw,defaults /dev/rbd13 /var/lib/kubelet/plugins/ceph.rook.io/rook-ceph/mounts/pvc-78cf22fa-d776-43a4-98d7-d594f02ea018 Output: Running scope as unit run-r2594d6c82152421c8891bfa8761e8c05.scope. mount: mount /dev/rbd13 on /var/lib/kubelet/plugins/ceph.rook.io/rook-ceph/mounts/pvc-78cf22fa-d776-43a4-98d7-d594f02ea018 failed: Structure needs cleaning
```
處理方式:
透過ceph tool重新掛載該pv的image，試著使用fsck修復
## 詳細處理流程:
```bash
$ kubectl exec -it -n rook-ceph rook-ceph-tools-69f996589-ps4k7 bash
[root@twtpedev013 /]# rbd map --pool=replicapool pvc-78cf22fa-d776-43a4-98d7-d594f02ea018
[root@twtpedev013 /]# lsblk
NAME                                                                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                                                                   8:0    0   16G  0 disk
|-sda1                                                                8:1    0  731M  0 part
|-sda2                                                                8:2    0    1K  0 part
`-sda5                                                                8:5    0 15.3G  0 part
  |-ubuntu--vg-root                                                 252:0    0 14.3G  0 lvm  /etc/hosts
  `-ubuntu--vg-swap_1                                               252:1    0  976M  0 lvm
sdb                                                                   8:16   0 1000G  0 disk
`-ceph--4b83cf83--445d--42cc--8eeb--f7febf99fcab-osd--data--ee3fbf0e--4521--46fd--9879--e73048f7a697
                                                                    252:2    0  999G  0 lvm
sr0                                                                  11:0    1 1024M  0 rom
rbd0                                                                251:0    0   10G  0 disk
[root@twtpedev013 /]# mkdir test-mysql
[root@twtpedev013 /]# mount /dev/rbd0 test-mysql/
mount: mount /dev/rbd0 on /test-mysql failed: Structure needs cleaning
[root@twtpedev013 /]# fsck /dev/rbd0
fsck from util-linux 2.23.2
If you wish to check the consistency of an XFS filesystem or
repair a damaged filesystem, see xfs_repair(8).
[root@twtpedev013 /]# xfs_repair -L /dev/rbd0
Phase 1 - find and verify superblock...
- reporting progress in intervals of 15 minutes
Phase 2 - using internal log
- zero log...
ALERT: The filesystem has valuable metadata changes in a log which is being
destroyed because the -L option was used.
- scan filesystem freespace and inode maps...
agi unlinked bucket 35 is 8227 in ag 0 (inode=8227)
sb_ifree 228, counted 227
sb_fdblocks 2398405, counted 2426486
- 02:26:05: scanning filesystem freespace - 17 of 17 allocation groups done
- found root inode chunk
Phase 3 - for each AG...
- scan and clear agi unlinked lists...
- 02:26:05: scanning agi unlinked lists - 17 of 17 allocation groups done
- process known inodes and perform inode discovery...
- agno = 0
- agno = 15
- agno = 16
Metadata CRC error detected at xfs_bmbt block 0x2d9ff8/0x1000
data fork in ino 8226 claims free block 1112
correcting imap
Metadata CRC error detected at xfs_bmbt block 0x2d9ff8/0x1000
btree block 2/48127 is suspect, error -74
bad magic # 0 in inode 8227 (data fork) bmbt block 572415
bad data fork in inode 8227
cleared inode 8227
- agno = 1
- agno = 2
- agno = 3
- agno = 4
- agno = 5
- agno = 6
- agno = 7
- agno = 8
- agno = 9
- agno = 10
- agno = 11
- agno = 12
- agno = 13
- agno = 14
- 02:26:05: process known inodes and inode discovery - 384 of 384 inodes done
- process newly discovered inodes...
- 02:26:05: process newly discovered inodes - 17 of 17 allocation groups done
Phase 4 - check for duplicate blocks...
- setting up duplicate extent list...
- 02:26:05: setting up duplicate extent list - 17 of 17 allocation groups done
- check for inodes claiming duplicate blocks...
- agno = 0
- agno = 6
- agno = 7
- agno = 8
- agno = 9
- agno = 10
- agno = 11
- agno = 12
- agno = 13
- agno = 14
- agno = 15
- agno = 16
- agno = 1
- agno = 3
- agno = 4
- agno = 5
- agno = 2
- 02:26:05: check for inodes claiming duplicate blocks - 384 of 384 inodes done
Phase 5 - rebuild AG headers and trees...
- 02:26:05: rebuild AG headers and trees - 17 of 17 allocation groups done
- reset superblock...
Phase 6 - check inode connectivity...
- resetting contents of realtime bitmap and summary inodes
- traversing filesystem ...
- traversal finished ...
- moving disconnected inodes to lost+found ...
Phase 7 - verify and correct link counts...
- 02:26:05: verify and correct link counts - 17 of 17 allocation groups done
Metadata corruption detected at xfs_bmbt block 0x2d9ff8/0x1000
libxfs_writebufr: write verifer failed on xfs_bmbt bno 0x2d9ff8/0x1000
Maximum metadata LSN (82836:5824) is ahead of log (1:64).
Format log to cycle 82839.
releasing dirty buffer (bulk) to free list!done
[root@twtpedev013 /]# mount /dev/rbd0 test-mysql/
[root@twtpedev013 /]# ls test-mysql/
#ib_16384_0.dblwr binlog.000009 client-cert.pem ib_logfile1 performance_schema sys
#ib_16384_1.dblwr binlog.index client-key.pem ibdata1 private_key.pem undo_001
#innodb_temp binlog.~rec~ eams ibtmp1 public_key.pem undo_002
auto.cnf ca-key.pem ib_buffer_pool mysql server-cert.pem
binlog.000008 ca.pem ib_logfile0 mysql.ibd server-key.pem
[root@twtpedev013 /]# umount /dev/rbd0
[root@twtpedev013 /]# rbd unmap /dev/rbd0
```
之後再重新將該MySQL deployment重啟發現正常了
```
2023-05-04 10:28:38+08:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.21-1debian10 started.
2023-05-04 10:28:41+08:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2023-05-04 10:28:41+08:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.21-1debian10 started.
2023-05-04T02:28:42.380318Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.21) starting as process 1
2023-05-04T02:28:42.428012Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2023-05-04T02:28:47.779944Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2023-05-04T02:28:48.922746Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2023-05-04T02:28:49.052930Z 0 [System] [MY-010229] [Server] Starting XA crash recovery...
2023-05-04T02:28:49.070234Z 0 [System] [MY-010232] [Server] XA crash recovery finished.
2023-05-04T02:28:49.287142Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2023-05-04T02:28:49.287524Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2023-05-04T02:28:49.330334Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
2023-05-04T02:28:50.168132Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.21'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
```
## ref
https://docs.ceph.com/en/quincy/start/quick-rbd/
https://blog.51cto.com/u_13210651/2352686
https://www.tecmint.com/find-linux-filesystem-type/
