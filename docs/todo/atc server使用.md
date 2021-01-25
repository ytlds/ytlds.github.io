根据mdc使用说明confluence在10.9.2.47部署常驻docker，启动atcserver，开始运行

```bash
docker pull artifactory.momenta.works/docker-mpe-dev/mdc610_ubuntu18.04:v13

docker run 
```

run_docker.sh

```bash
#!/usr/bin/env bash
set -e -u
 
AUTOSETUP=${AUTOSETUP:="FALSE"}
 
DOCKER=docker
USER_NAME=HwHiAiUser
USER_HOME=/home/$USER_NAME
 
# IMAGE_NAME=mdc610_ubuntu18.04
IMAGE_NAME=artifactory.momenta.works/docker-mpe-dev/mdc610_ubuntu18.04
TAG_NAME=v13
 
CONTAINER_NAME='mdc610_atc_ubuntu18.04'-$TAG_NAME
DISPLAY=${DISPLAY:=""}
 
echo "Running container '$CONTAINER_NAME' from image '$IMAGE_NAME:$TAG_NAME' with '$DOCKER'..."
 
function get_volumes_opt() {
    # VOLUMES_OPT="--volume $HOME/volumes:$USER_HOME/volumes"
    # VOLUMES_OPT="$VOLUMES_OPT --volume $HOME/.Xauthority:$USER_HOME/.Xauthority:rw"
    VOLUMES_OPT="--volume /home/MDC/workspace/:$USER_HOME/workspace"
}
get_volumes_opt
echo "Volumes..., '$VOLUMES_OPT'"
 
$DOCKER start $CONTAINER_NAME > /dev/null 2> /dev/null || {
    echo "Creating new container..."
    $DOCKER run \
          --detach \
          --net=host \
          --name $CONTAINER_NAME \
          $VOLUMES_OPT \
          --tty \
          --env DISPLAY=$DISPLAY \
          $IMAGE_NAME:$TAG_NAME
    echo "Changed user uid/gid... (this may take a while)"
    $DOCKER exec --tty $CONTAINER_NAME chown -R $(id -u):$(id -g) $USER_HOME
    $DOCKER exec --tty $CONTAINER_NAME usermod -u $(id -u) $USER_NAME
    $DOCKER exec --tty $CONTAINER_NAME groupmod -g $(id -g) $USER_NAME
}
 
if [ "$AUTOSETUP" = "FALSE" ]; then
    if [ "$#" -eq  "0" ]; then
        $DOCKER exec --interactive --tty --user $USER_NAME $CONTAINER_NAME bash
    else
        $DOCKER exec --interactive --tty --user $USER_NAME $CONTAINER_NAME bash -c "$@"
    fi
fi
```

运行 上述脚本，启动docker，部署如下：

```bash
 docker exec -it --user HwHiAiUser mdc610-atc-server-ubuntu18.04-v13 bash
```



atc server 连接步骤：

```bash
ssh HwHiAiUser@10.9.2.47 -p 9247  # 密码：9247
cd ~/workspace
部署dlinfer，运行即可。
```

atc server 的workspace 映射了47的MDC用户的workspace，由34或47管理员处理