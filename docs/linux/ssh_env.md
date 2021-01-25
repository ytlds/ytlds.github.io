# sshpass 登陆服务器获取环境变量失败

### 1. 背景

通过 ssh 连接 atc server 启动 rpc 服务失败，具体方式为

```bash
sshpass -p <password> ssh -o "StrictHostKeyChecking no" user@remote -p <port> <command>
```

通过 ssh user@remote 登陆服务器后，执行 <command>，则无此问题。

### 2. 原因

不同的执行方式加载的环境变量不同是导致问题的原因

### 3. 解决办法

## 1.问题背景

通过 ssh 连接 atc server 启动 rpc 服务失败，具体方式为

```
sshpass -p <password> ``ssh` `-o ``"StrictHostKeyChecking no"` `user@remote -p <port> <``command``>
```

通过 ssh user@remote 登陆服务器后，执行 <command>，则无此问题。

## 2.处理记录

根据现象，猜测不同的执行方式加载的环境变量不同是导致问题的原因。测试步骤如下：

#### 2.1 验证命令正确性

将 1 中的 bash 命令中的 <command> 替换为普通命令，例如 ls、pwd、echo 等，均能正常运行，证明该方式可用。

#### 2.2 对比环境变量

对比 remote 和 ssh remote 分别执行 env 的结果，如下：

![img](https://confluence.momenta.works/download/attachments/109697946/image2021-2-2_21-3-8.png?version=1&modificationDate=1612270988517&api=v2)

发现 $PATH 的环境变量差距很大，ssh remote 的结果基本就是系统初始化的结果，未加载任何配置。

#### 2.3 查询配置加载流程

查询 bash 资料后，可以确定 1 中的方式为 bash 的四种模式中的 non-interactive + non-login shell 模式，此模式和我们常用的 interactive + login shell 加载配置文件的区别非常大。

多次验证后，可以确定在 ssh -o "StrictHostKeyChecking no" user@remote -p <port> <command> 此种方式下无法加载任何配置，也无法远程启动 server。

#### 2.4 解决方法

解决该问题的方法需要正确的加载环境变量配置，为了方便用户操作，将加载环境变量、启动任务等编写脚本执行，并在脚本 shebang 中增加 --login 参数，将 bash 强行修改为 login 模式运行。

按照加载顺序，首先是 /etc/profile，然后是 ~/.profile，而在 ~/.profile 中首先加载 ~/.bashrc，如下图所示，

![img](https://confluence.momenta.works/download/attachments/109697946/image2021-2-2_21-17-40.png?version=1&modificationDate=1612271860850&api=v2)

其中 ~/.bashrc 包含了我们真正需要的环境变量，打开后发现在文件开始判断交互模式，如果是 non-interactive 则退出，如下图所示,

![img](https://confluence.momenta.works/download/attachments/109697946/image2021-2-2_21-20-13.png?version=1&modificationDate=1612272013411&api=v2)

将此部分注释即可。

#### 2.5 优化启动

在脚本中增加对输出的重定向，解决了启动 server 会占用当前 session 的问题，如下：

```
#!/bin/bash --login` `cd` `/home/HwHiAiUser/workspace` `ps` `aux | ``grep` `server | ``grep` `-``v` `grep` `| ``awk` `'{print $2}'` `| ``xargs` `kill` `echo` `"kill old server process"` `nohup` `python3 server.py >``/dev/null` `2>&1 &` `sleep` `2` `echo` `"success"
```



## 3 验证结果

对比 remote 和 ssh remote 分别执行 env 的结果，其中 *PAT**H*和 PYTHONPATH 如下，结果正常。

![img](https://confluence.momenta.works/download/attachments/109697946/image2021-2-2_21-21-43.png?version=1&modificationDate=1612272103393&api=v2)

验证 remote inference 结果，如下：

![img](https://confluence.momenta.works/download/attachments/109697946/image2021-2-2_21-30-25.png?version=1&modificationDate=1612272625636&api=v2)