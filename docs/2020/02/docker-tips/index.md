# Docker Tips


## Cannot Resolve Host Name inside Running Containers

```bash
sudo vim /etc/docker/daemon.json

{
    "dns": ["10.245.177.15"]
}

sudo systemctl restart docker.service
```
