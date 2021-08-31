---
layout: post
title: Updating FreeBSD and iocage
date: 2021-08-31
---

In this document we will explain how to migrate to a newer FreeBSD version both on hosts and on jails.

You can always refer to [Chapter 24: Updating and Upgrading FreeBSD](https://docs.freebsd.org/en/books/handbook/cutting-edge/) for more detailed information.

 
## Updating the host

FreeBSD is known for being reliable and to be painlessly upgradeable, even with custom setups, like we do, with manual partitioning, ZFS on root, FreeBSD can handle this easily.

We should start by evaluating the current version we have installed, with:

```sh
uname -mrs
```

which in my case renders `FreeBSD 12.2-RELEASE amd64`.

We should make sure you apply all existing pending updates for FreeBSD 12.x:

```sh
sudo freebsd-update fetch
sudo freebsd-update install
sudo pkg update
sudo pkg upgrade
```

Make sure to have a TransIP snapshot of the machine or VMWare snapshot for a VM

Once all updates are installed, we can then proceed to do the upgrade (in this case to 13.0-RELEASE):

```sh
sudo freebsd-update -r 13.0-RELEASE upgrade
```

After that process is done, which may take a while, you will get prompted

```
To install the downloaded upgrades, run "/usr/sbin/freebsd-update install"
```

You should thus run:

```sh
sudo /usr/sbin/freebsd-update install
```

Then reboot the machine, then run again:

```sh
sudo /usr/sbin/freebsd-update install
```

Then you should perform a second reboot and you will have a system with the new OS but old packages.

Your `/etc/resolv.conf` might be overriden during the update. Make sure to check this.

The next logical step to follow, after the last reboot, is to upgrade all packages of the system, once again with: 

```sh
sudo pkg update
sudo pkg upgrade
```

Once all software is up-to-date, run again: 

```sh
sudo /usr/sbin/freebsd-update install
```

and another reboot is recommended.

## Updating jails

Even though this is a low-risk process, for production systems that cannot allow any downtime it is recommended to simply create a new jail with the new release, and switch traffic to the new jail. This requires more work, but will be safer, and is a benefit of having a jailed environment.

Upon a successful update of your host’s system, any of your existing jails will still be using the release where you created them. Ideally you should always keep your jails to the same major release as your host.

You can get an overview of your jails with:

```sh
sudo iocage list
```

You should prefetch the wanted release files, which will then be available for all upgrades and for any new jails you might want to create with said version.

```
sudo iocage fetch -r 13.0-RELEASE
```

 
If the machine you are working on contains many jails, you should likely open several SSH sessions and upgrade the jails in parallel, of course at the cost of CPU & RAM usage.

In order to upgrade a jail’s FreeBSD version you should run:

```
sudo iocage upgrade <name of your jail> -r 13.0-RELEASE
```

then we must update all packages within the jail:

```
sudo iocage pkg <name of your jail> update
sudo iocage pkg <name of your jail> upgrade -y
```

it is recommendable to restart the jail after the procedure:

```
sudo iocage restart <name of your jail>
```

 
## Cleaning up

It is highly recommendable to cleanup your system after verifying everything is stable and in order. Several files, backups and rollbacks can then be deleted, often saving 8GB+ of space. There are several handy tricks to be able to clean a FreeBSD machine and shave some disk GB.

You can do some cleanup by removing the fetched pkg files, both on your host

```sh
sudo pkg clean
```

and on your jails:

```sh
sudo iocage pkg <your jail> clean -y
```

You can check the current allocated size for zpools by running: 

```
zpool list

NAME     SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
iocage  19.5G  13.5G  5.97G        -         -    66%    69%  1.00x    ONLINE  -
zroot   17.5G  3.72G  13.8G        -         -    23%    21%  1.00x    ONLINE  -
```

You can check which zfs file systems are currently taking more space with:

```
zfs list

NAME                                       USED  AVAIL     REFER  MOUNTPOINT
iocage                                    13.5G  5.38G       24K  /iocage
iocage/iocage                             13.5G  5.38G     29.5K  /iocage/iocage
iocage/iocage/download                     803M  5.38G       24K  /iocage/iocage/download
iocage/iocage/download/12.2-RELEASE        402M  5.38G      402M  /iocage/iocage/download/12.2-RELEASE
iocage/iocage/download/13.0-RELEASE        401M  5.38G      401M  /iocage/iocage/download/13.0-RELEASE
iocage/iocage/images                        24K  5.38G       24K  /iocage/iocage/images
iocage/iocage/jails                       10.5G  5.38G       24K  /iocage/iocage/jails
iocage/iocage/jails/elk-d                 4.19G  5.38G     25.5K  /iocage/iocage/jails/elk-d
iocage/iocage/jails/elk-d/root            4.19G  5.38G     4.51G  /iocage/iocage/jails/elk-d/root
iocage/iocage/jails/jaeger-d              4.00G  5.38G     25.5K  /iocage/iocage/jails/jaeger-d
iocage/iocage/jails/jaeger-d/root         4.00G  5.38G     4.32G  /iocage/iocage/jails/jaeger-d/root
iocage/iocage/jails/web-server-d          2.26G  5.38G     25.5K  /iocage/iocage/jails/web-server-d
iocage/iocage/jails/web-server-d/root     2.26G  5.38G     2.57G  /iocage/iocage/jails/web-server-d/root
iocage/iocage/log                           28K  5.38G       28K  /iocage/iocage/log
iocage/iocage/releases                    2.26G  5.38G       24K  /iocage/iocage/releases
iocage/iocage/releases/12.2-RELEASE       1.20G  5.38G       24K  /iocage/iocage/releases/12.2-RELEASE
iocage/iocage/releases/12.2-RELEASE/root  1.20G  5.38G     1.20G  /iocage/iocage/releases/12.2-RELEASE/root
iocage/iocage/releases/13.0-RELEASE       1.06G  5.38G       24K  /iocage/iocage/releases/13.0-RELEASE
iocage/iocage/releases/13.0-RELEASE/root  1.06G  5.38G     1.06G  /iocage/iocage/releases/13.0-RELEASE/root
iocage/iocage/templates                     24K  5.38G       24K  /iocage/iocage/templates
zroot                                     3.72G  13.2G       24K  /zroot
zroot/ROOT                                3.23G  13.2G       24K  none
zroot/ROOT/default                        3.23G  13.2G     3.23G  /
zroot/tmp                                 26.5K  13.2G     26.5K  /tmp
zroot/usr                                  497M  13.2G       24K  /usr
zroot/usr/home                            3.50M  13.2G     3.50M  /usr/home
zroot/usr/obj                               24K  13.2G       24K  /usr/obj
zroot/usr/ports                             72K  13.2G       24K  /usr/ports
zroot/usr/ports/distfiles                   24K  13.2G       24K  /usr/ports/distfiles
zroot/usr/ports/packages                    24K  13.2G       24K  /usr/ports/packages
zroot/usr/src                              493M  13.2G      493M  /usr/src
zroot/var                                  259K  13.2G       24K  /var
zroot/var/audit                             24K  13.2G       24K  /var/audit
zroot/var/crash                             24K  13.2G       24K  /var/crash
zroot/var/log                              135K  13.2G      135K  /var/log
zroot/var/mail                              28K  13.2G       28K  /var/mail
zroot/var/tmp                               24K  13.2G       24K  /var/tmp
```

By looking at the above output you can see we have both the 12.2-RELEASE and 13.0-RELEASE versions of FreeBSD saved in our machine. In our case, we have successfully upgraded to 13.0-RELEASE and thus can remove all files stored related to 12.2-RELEASE. We can do this by simply removing the zfs dataset containing the 12.2-RELEASE download:

```sh
sudo zfs destroy -r iocage/iocage/download/12.2-RELEASE
```

Be extremely wary when running this command and ask someone more experienced if you are doubting!

We also can then clean the freebsd-update remaining files by removing the rollback and creating a new empty folder:

```sh
sudo rm -rf /var/db/freebsd-update/files
sudo mkdir /var/db/freebsd-update/files
sudo chmod 755 /var/db/freebsd-update/files
```

This should also be done in the jails if you upgraded them:

```sh
sudo iocage exec <your jail> rm -rf /var/db/freebsd-update/files
sudo iocage exec <your jail> mkdir /var/db/freebsd-update/files
sudo iocage exec <your jail> chmod 755 /var/db/freebsd-update/files
```

When you update to a new major version you might end up keeping old datasets lying around. These can be seen like above in the output of zfs list.

Very likely at `iocage/iocage/releases/12.2-RELEASE` and `iocage/iocage/releases/12.2-RELEASE/root`.

We must first find the “cloned” datasets with:

```sh
sudo zfs list -t snapshot -o name,clones
```

which outputs something like:

```
NAME                                                  CLONES
iocage/iocage/releases/12.2-RELEASE/root@database     iocage/iocage/jails/database/root
iocage/iocage/releases/12.2-RELEASE/root@cache-store  iocage/iocage/jails/cache-store/root
```

and then we can promote the clones to be the primary datasets for that jail:

```sh
sudo zfs promote iocage/iocage/jails/database/root
sudo zfs promote iocage/iocage/jails/cache-store/root
 ```

The clones can be removed with:

```sh
sudo zfs destroy iocage/iocage/releases/12.2-RELEASE/root
sudo zfs destroy iocage/iocage/releases/12.2-RELEASE
```

After all the cleanup it is recommended to restart the machine and verify it all works as expected.
 

