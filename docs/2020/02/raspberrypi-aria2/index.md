# Raspberrypi Aria2 Downloader


## UPDATE: raspberrypi is not used any more.

## Installation and configuration

Refer to https://www.cnblogs.com/hzdx/p/raspberry_aria2.html?utm_source=itdadao&utm_medium=referral

**NOTE** I did some customization and put them in a shell. Just run it to configure aria2.

```bash
#!/bin/bash

function die() {
  echo $1 ; exit 1
}

echo Installing aria2
sudo apt install aria2 || die Installation failure.

echo Configuring aria2
{ mkdir ~/.aria2 ; touch ~/.aria2/aria2.session ; cat <<EOF >~/.aria2/aria2.conf
continue=true
#后台运行
daemon=true
#默认下载目录
dir=/home/pi/Downloads
#立即分配下载所需的空间对ext4支持最好
file-allocation=falloc
log-level=warn
max-connection-per-server=4
max-concurrent-downloads=3
max-overall-download-limit=200K
min-split-size=5M
enable-http-pipelining=true
#启用rpc调用接口
enable-rpc=true
rpc-listen-all=true
#rpc的访问密码
#rpc-secret=hzdx#保存下载会话
save-session=/home/pi/.aria2/aria2.session
input-file=/home/pi/.aria2/aria2.session
EOF } || die Preparing aria2.conf failure.

echo Configuring systemd
echo '\
[Unit]
Description=Aria2 Service
After=network.target

[Service]
User=pi
Type=forking
ExecStart=/usr/bin/aria2c --conf-path=/home/pi/.aria2/aria2.conf

[Install]
WantedBy=multi-user.target' | sudo tee /lib/systemd/system/aria.service \
|| die Configuring systemd failure.

echo Configuring webui for aria2
sudo apt install git nginx-light || die Installing git and nginx-light failure.
git clone https://github.com/ziahamza/webui-aria2.git /tmp/webui-aria2
sudo mv /tmp/webui-aria2/* /var/www/html/

sudo systemctl daemon-reload
sudo systemctl enable aria2
sudo systemctl enable nginx

```
