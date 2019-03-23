## Running in Docker Environment
### Install docker
The docker installation is relatively simple, you can refer to the following two ways to install:

- Installation from [Aliyun image](https://yq.aliyun.com/articles/110806)
- Installation From [Docker’s official image](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

### Run Container
- Pull Platon Image
```bash
$ sudo docker pull platonnetwork/platon:tag #e.g: platonnetwork/platon:0.5.0
```

- Run Platon Container

Start the Platon node in the container by executing the following commands. The default RPC port is 6789 and the P2P port is 16789:
```bash
$ sudo docker run -d platonnetwork/platon:tag
```

To open port mapping, execute the following commands:
```bash
$ sudo docker run -d -e PLATONIP="192.168.120.20" -p 6789:6789  -p 16789:16789 --name platon platonnetwork/platon:0.5.0
```
`PLATONIP` is the local server address.

Execute the following commands to enter the container:
```bash
$ sudo docker exec -it container_name bash #container_name is the container name or id.
```

Execute a command to enter the platform’s JS terminal(The default directory is under /opt/node when you enter the container):
```bash
# platon attach ipc:./data/platon.ipc
```

Note: If not enable port mapping, you can only enter into the JS terminal through the above commands in the container. After opening the port mapping, you can enter the JS terminal locally through the following commands:
```bash
# platon attach http://PLATONIP:6789
```

Start and stop the platform service in the container using the following commands:
```bash
# supervisorctl start platon 
# supervisorctl stop platon 
# supervisorctl restart platon 
```

If the above command is invalid, use the following command：
```bash
# /etc/init.d/supervisor stop
# /etc/init.d/supervisor start
```

>Note: The data directory of Platon in the container is located in opt/node directory. By default, the system has only one wallet account, and the password is 123456.

