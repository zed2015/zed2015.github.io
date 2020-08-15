### centos 6.3, kernel 3.10 install docker and k8s
#### install docker
- download builded docker-ce from [url](https://mirrors.aliyun.com/docker-ce/linux/static/stable/x86_64/docker-17.03.2-ce.tgz)
- unpack download file, use `tar xf file`
- `sudo cp docker/* /usr/bin`
- create config used by dockerd `sudo vim /etc/docker/daemon.json`
```json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://registry.aliyuncs.com",
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "hosts": [
    # "tcp://0.0.0.0:8777",
    "unix:///var/run/docker.sock"
  ],
  "storage-driver": "overlay"
}

```
- start docker `sudo dockerd`
