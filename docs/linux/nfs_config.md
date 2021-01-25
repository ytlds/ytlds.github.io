# NFS服务配置

【配置NFS】

NFS配置起来还是蛮简单的，只需要编辑配置文件/etc/exports即可。下面笔者先创建一个简单的NFS服务器。

[root@localhost ~]# cat /etc/exports


/home/  10.0.2.0/24(rw,sync,all_squash,anonuid=501,anongid=501)


这个配置文件就这样简单一行。共分为三部分，第一部分就是本地要共享出去的目录，第二部分为允许访问的主机（可以是一个IP也可以是一个IP段）第三部分就是小括号里面的，为一些权限选项。关于第三部分，笔者简单介绍一下：


rw ：读写；


ro ：只读；


sync ：同步模式，内存中数据时时写入磁盘；


async ：不同步，把内存中数据定期写入磁盘中；


no_root_squash ：加上这个选项后，root用户就会对共享的目录拥有至高的权限控制，就像是对本机的目录操作一样。不安全，不建议使用；


root_squash ：和上面的选项对应，root用户对共享目录的权限不高，只有普通用户的权限，即限制了root；


all_squash ：不管使用NFS的用户是谁，他的身份都会被限定成为一个指定的普通用户身份；


anonuid/anongid ：要和root_squash 以及 all_squash一同使用，用于指定使用NFS的用户限定后的uid和gid，前提是本机的/etc/passwd中存在这个uid和gid。


介绍了上面的相关的权限选项后，再来分析一下笔者刚刚配置的那个/etc/exports文件。其中要共享的目录为/home，信任的主机为10.0.2.0/24这个网段，权限为读写，同步，限定所有使用者，并且限定的uid和gid都为501。


【使用NFS】


         当编辑完配置文件/etc/exports后，就该启动NFS服务了。启动方法为：


[root@localhost ~]# service portmap start; service nfs start


NFS是依托portmap的，所以首先要启动portmap，然后启动NFS才能是刚才的配置生效。启动完NFS后，就该使用NFS服务了。


[root@localhost ~]# showmount -e 127.0.0.1 （用在client上）


Export list for 127.0.0.1:


/home 10.0.2.0/24


用shoumount -e 加IP就可以查看NFS的共享情况，上例中，就可以看到127.0.0.1的共享目录为/home，信任主机为10.0.2.0/24这个网段。另外这个showmount 命令还有一个常用的选项就是-a了，它的意思是，把连接本机的NFS的client全部列出。


[root@localhost ~]# mount -t nfs 10.0.2.69:/home /mnt （client上）


[root@localhost ~]# showmount -a （nfs服务器上）


All mount points on localhost:


10.0.2.69:/home


前面的mount 命令为挂载NFS共享目录，相信你能看懂这个格式。showmount -a 命令列出所有的clinet。


NFS服务中还有一个常用的命令那就是exportfs，它的常用选项为[-aruv]。


-a ：全部挂载或者卸载；


-r ：重新挂载；


-u ：卸载某一个目录；


-v ：显示共享的目录；


使用exportfs命令，当改变/etc/exports配置文件后，不用重启nfs服务直接用这个exportfs即可。


[root@localhost ~]# cat /etc/exports


/tmp/   10.0.2.0/24(rw,sync,no_root_squash)


[root@localhost ~]# exportfs -arv （nfs服务器上）


exporting 10.0.2.0/24:/tmp


更改目录后，直接exportfs -arv即可生效。


在上面使用到了mount命令来挂载nfs，其实mount这个nfs服务还是有些说法的。首先是用-t nfs 来指定挂载的类型为nfs。另外在使用nfs时，常用一个选项就是nolock了，即在挂载nfs服务时，不加锁。


[root@localhost ~]# mount -t nfs -o nolock 10.0.2.69:/tmp /mnt/


[root@localhost ~]# showmount -a


All mount points on localhost:


10.0.2.69:/home


10.0.2.69:/tmp


另外我们还可以把要挂载的nfs目录写到client上的/etc/fstab文件中，挂载时只需要mount -a即可。


[root@localhost ~]# cat /etc/fstab


LABEL=/                 /                       ext3    defaults        1 1

LABEL=/boot             /boot                   ext3    defaults        1 2

tmpfs                   /dev/shm                tmpfs   defaults        0 0

devpts                  /dev/pts                devpts  gid=5,mode=620  0 0

sysfs                   /sys                    sysfs   defaults        0 0

proc                    /proc                   proc    defaults        0 0

LABEL=SWAP-hda2         swap                    swap    defaults        0 0

10.0.2.69:/tmp          /mnt                    nfs     nolock          0 0

 


写完/etc/fstab文件后，只需要mount -a即可挂载nfs服务的共享目录。


[root@localhost ~]# umount /mnt/ 首先把刚才挂载的nfs卸载掉


[root@localhost ~]# mount -a


[root@localhost ~]# df -h


Filesystem            Size  Used Avail Use% Mounted on

/dev/hda3             7.3G  3.7G  3.3G  53% /

/dev/hda1              99M   12M   83M  12% /boot

tmpfs                  84M     0   84M   0% /dev/shm

10.0.2.69:/tmp        7.3G  3.7G  3.3G  53% /mnt


关于NFS部分就讲这么多，内容并不多，相信你很快就能掌握！

上一页 学会使用简单的MySQL操作  

下一页 配置FTP服务  

回到主目录

