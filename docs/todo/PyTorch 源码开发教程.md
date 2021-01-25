1 工具准备
本地：vscode
服务器：带 gpu 支持的 docker 环境
2 源码编译环境设置

```bash
cd ~
git clone --recursive -b dev git@gitlab.momenta.works:TrainingPlatform/pytorch-dev/pytorch.git
cd pytorch
git pull --rebase
git submodule sync --recursive
git submodule update --init --recursive
 
 
# 容器名字
CONTAINER_NAME="your-fancy-container-name"
 
# 把外部的pytorch挂载到/workspace/pytorch目录下
docker run -it --name $CONTAINER_NAME --hostname $CONTAINER_NAME --shm-size=16g --ulimit memlock=-1 --ulimit stack=67108864 --network=host --ipc=host --privileged --runtime=nvidia -v $HOME/pytorch:/workspace/pytorch -v $HOME:$HOME registry.momenta.works/perception_hpc/torchland:dev bash
 
# 在容器内
cd /workspace
export PATH="/usr/lib/ccache:$PATH"
# 设置 ccache 目录，建议指向到 native 实际存储，如 /home/user/.ccache 目录，避免容器退出后 cache 没了
mkdir /home/liuhuiya/.ccache
export CCACHE_DIR=/home/liuhuiya/.ccache
 
# 看看 gcc、nvcc 是不是指向 ccache
which gcc; which nvcc
./build_pytorch.sh
```
```bash
# 运行测试
python3 test/test_cuda.py
```
3 vscode 开发环境设置
```bash
# 在刚才的容器里安装 ssh，修改端口至 12345（端口有时候不可用，需要换别的端口）
apt-get update
apt-get install -y ssh curl
sed -i s/#Port\ 22/Port\ 12345/g /etc/ssh/sshd_config
sed -i s/PermitRootLogin\ prohibit-password/PermitRootLogin\ yes/g /etc/ssh/sshd_config
mkdir ~/.ssh/
echo "本机的id_rsa.pub，用来免密码登录" >> ~/.ssh/authorized_keys
/etc/init.d/ssh restart
 
# 在本地设置免密码登录
vim ~/.ssh/config
# 写入以下内容
Host m47-23456
   HostName 10.9.2.47
   Port 12345
   User root
   IdentityFile ~/.ssh/id_rsa
 
# 打开vscode，安装插件Remote Development
# 打开左侧栏的Remote Explorer，打开刚才的服务器
# 打开左侧栏的Extensions，在 server 上安装 python、c++ 等必要的开发环境
# 开始快乐开发。。。
```
4 C++ debug 方法
```bash
# 在容器内
apt install python3-dbg gdb
# 修改 /workspace/build_pytorch.sh，打开 DEBUG 标志，重新编译
# 设置 vscode 的 gdb 调试环境，编写 .vscode/launch.json
 
{
        "name": "(gdb) Launch",
        "type": "cppdbg",
        "request": "launch",
        "program": "/usr/bin/python3",
        "args": ["test/m/test_minfer_export.py"],
        "stopAtEntry": true,
        "cwd": "${workspaceFolder}",
        "environment": [],
        "externalConsole": false,
        "MIMode": "gdb",
        "setupCommands": [
            {
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            }
        ]
}
 
# 具体内容自己修改
# 在C++源码上打上断点，开始debug
```
5 pip 安装包打包过程

```bash
cd /workspace/pytorch
python3 setup.py bdist_wheel -d /workspace/
```



