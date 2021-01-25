# shell 常用命令

### 文件夹查看操作




```bash

# kill 包含 xxx 的进程
ps -aux | grep xxx | grep -v grep | awk ‘{print $1}’ | xargs kill
# 去掉文件中的 ^M
dos2unix filename
# 查看某个文件夹下文件的个数
ls -l | grep '^-' | wc -l 
# 查看某个文件夹下文件的个数，包括子文件夹下的文件个数
ls -lR |grep '^-' | wc -l
# 查看某个文件夹下文件夹的个数
ls -l | grep '^d' wc -l
# 查看某个文件夹下文件夹的个数，包括子文件夹下的文件夹个数
ls -lR | grep '^d' | wc -l
# 查看文件夹下所有的文件和文件夹。也就是统计ls -l命令所输出的行数
ls -l | wc -l
```


```bash
# 查看有哪些 shell 可用
cat /etc/shells
```

```bash
# 目录下按照文件大小排序，du 命令
du -sh ./* | sort -nr
```

```bash
useradd -m test_user  					#添加用户
passwd test_user  	  					#设置用户密码
usermod -s /bin/bash test_user 	#设置用户登陆 shell
userdel -r test_user						#删除用户, 常用的选项是-r, 它的作用是把用户的主目录一起删除


# /etc/passwd 文件信息格式
# 用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell
# LOGNAME:PASSWORD:UID:GID:USERINFO:HOME:SHELL
test_user:x:11011:11011::/home/test_user:/bin/bash
```

```bash
# 解压
tar –xvf file.tar 			#解压 tar包
tar -xzvf file.tar.gz 	#解压tar.gz
tar -xjvf file.tar.bz2 	#解压 tar.bz2
tar –xZvf file.tar.Z 		#解压tar.Z
unrar e file.rar 				#解压rar
unzip file.zip 					#解压zip

# 总结
1、*.tar 用tar –xvf 解压
2、*.gz 用gzip -d或者gunzip 解压
3、*.tar.gz和*.tgz用 tar –xzf 解压
4、*.bz2 用bzip2 -d或者用bunzip2 解压
5、*.tar.bz2用tar –xjf 解压
6、*.Z 用uncompress 解压
7、*.tar.Z 用tar –xZf 解压
8、*.rar 用unrar e解压
9、*.zip 用unzip 解压
```

ln命令

```bash
# 创建软链接
ln  -s  [源文件或目录]  [目标文件或目录]
# 例如：当前路径创建test 引向/var/www/test 文件夹 
ln –s  /var/www/test  test
# 创建/var/test 引向/var/www/test 文件夹 
ln –s  /var/www/test   /var/test 

# 删除软链接
rm –rf 软链接名称 #（请注意不要在后面加”/”，rm –rf 后面加不加”/” 的区别，可自行去百度下啊）
# 例如：删除test
rm –rf test

# 修改软链接
ln –snf  [新的源文件或目录]  [目标文件或目录]
ln –snf  /var/www/test1   /var/test

```

