# Remote Desktop to Linux


1. Install xrdp.
2. Install tigervnc-server.
3. Use gnome session instead of classic one:
```bash
echo "gnome-session" > ~/.Xclients
chmod +x ~/.Xclients
sudo systemctl restart xrdp.service
```
4. Start xrdp service and make it start as system boots.
```bash
systemctl start xrdp.service
systemctl enable xrdp.service
```
5. If the connection fails, it could be caused by firewall settings. Just add a new rule to firewall.
```bash
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
```

