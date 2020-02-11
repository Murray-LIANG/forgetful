# Set Up Shadowsocks on VPS


## Buy a Linux VPS service from iozoom.com

The default user is `root`.

## Install SSR
https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B

```console
$ wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh

$ # or

$ wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
$ chmod +x shadowsocksR.sh
$ ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

## Modify SSR
```console
$ bash ssr.sh
```

## [Do NOT use this, python shadowsocks not maintained, too old] Bring shadowsocks online

```bash
# Install
$ apt-get install python-pip
$ pip install shadowsocks

# Do some configuration and start service
$ ssserver -h
$ vim /etc/shadowsocks.json
$ echo 3 > /proc/sys/net/ipv4/tcp_fastopen
$ ssserver -c /etc/shadowsocks.json -d start

# Trouble shooting via log
sudo less /var/log/shadowsocks.log

# Stop service cli
ssserver -c /etc/shadowsocks.json -d stop
```
