## Docker环境运行

#### docker安装

docker安装比较简单，可参考以下两种安装方式：

- 从[阿里云Docker镜像源](https://yq.aliyun.com/articles/110806)安装
- 从[Docker官方镜像源](https://docs.docker.com/install/linux/docker-ce/ubuntu/)安装

#### 运行容器

- 拉取platon镜像

```bash
$ sudo docker pull platonnetwork/platon:tag #例如: platonnetwork/platon:0.5.0
```

- 运行platon容器

执行以下命令在容器中启动platon节点，其默认得RPC端口为6789，P2P端口为16789：

```bash
$ sudo docker run -d platonnetwork/platon:tag
```

若想开启端口映射，可执行以下命令：

```bash
$ sudo docker run -d -e PLATONIP="192.168.120.20" -p 6789:6789  -p 16789:16789 --name platon platonnetwork/platon:0.5.0
```

其中，`PLATONIP`为本机服务器地址。

执行以下命令进入容器(进入容器后默认目录就在/opt/node下):

```bash
$ sudo docker exec -it container_name bash #container_name为容器id或者名字
```

执行一下命令进入platon的js终端:

```bash
# platon attach ipc:./data/platon.ipc
```

注意：若没有开通端口映射，只能在容器内进入以上命令进入js终端，开启端口映射之后可在本机通过以下命令进入js终端：

```bash
# platon attach http://PLATONIP:6789
```

在容器内启停platon服务，使用如下命令：

```bash
# supervisorctl start platon #启动platon
# supervisorctl stop platon #停止platon
# supervisorctl restart platon #重启platon
```

如果以上命令无效，请使用以下命令

```bash
# /etc/init.d/supervisor stop
# /etc/init.d/supervisor start
```

**说明：容器中platon的数据目录位于`/opt/node`目录下，系统默认只有一个钱包账户，密码为:123456**