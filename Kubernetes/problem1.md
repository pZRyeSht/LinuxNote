
# Kubernetes 记Rancher部署状态一直Restarting

部署Rancher命令：
```shell
sudo docker run -d --restart=unless-stopped \
  -v /etc/rancher:/etc/rancher \
  -p 80:80 -p 443:443 rancher/rancher:latest
```

查看部署状态：
```
docker container ls

CONTAINER ID   IMAGE                       COMMAND               CREATED              STATUS            PORTS           NAMES
1412ca4f4cc3   rancher/rancher:latest      "entrypoint.sh"       About a minute ago   Restarting (1) 34 seconds ago
```

查看Rancher部署日志：
```
docker logs -f 1412ca4f4cc3

ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
ERROR: Rancher must be ran with the --privileged flag when running outside of Kubernetes
```

大概意思：在Kubernetes之外运行时，Rancher必须使用--privileged标志运行。

A:
```shell
sudo docker run --privileged -d --restart=unless-stopped \
  -v /etc/rancher:/etc/rancher \
  -p 80:80 -p 443:443 rancher/rancher:latest
```
privileged 的作用其实就是启动的 container内的root拥有真正的root权限！！！