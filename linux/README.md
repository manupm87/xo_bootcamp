# SaasOps Bootcamp Week 2

## Assignment 3 | Linux, Automation & Orchestration

### Command Line Quiz

#### 1.a

```
$ rm *
```

#### 1.b
```
$ rm $(ls | grep -P "file\d\.txt")
```

#### 2.a

```
$ ls -l my*
```

#### 2.b

```
$ ls -l -I 'my*'
```

#### 2.c

```
$ touch -t 198701141630 herfile.txt
```

#### 3

```
't' stands for the 'sticky' bit. Files within this directory will only be removable and renamable by its owner, directory owner, or root.
Any user can read, write and execute into this directory. A typical example is the /tmp/ directory
```

#### 4

```
$ find . -type f | xargs sed -i '42d' 
```

#### 5

```
$ ps efww
```

### Cron

#### 1

```
45 5 * * 1-5 /usr/bin/tar cf /var/log/backups/messages-`date +%Y%m%d-%H%M%S`.tar /var/log/messages
0 9 * * 6-7 /usr/bin/tar cf /var/log/backups/messages-`date +%Y%m%d-%H%M%S`.tar /var/log/messages
```

#### 2
```
Cron is not meant to run these kind of tasks. Typically nginx would be running as a daemon and systemd / initd would take proper care of killing it before shutdown / reboot.
```

#### 3
```
@daily /usr/bin/find /tmp/ -type f -user ec2-user | xargs rm -f
```

### SSH

#### 1
```
# ssh 10.0.42.10 -l root -i /root/.ssh/remotesrv usermod -s /sbin/nologin john
```

#### 2
```
$ ssh-copy-id -i ~/.ssh/id_rsa ubuntu@10.0.42.10
```

#### 3.a
```
$ scp ~/.ssh/id_rsa.pub ubuntu@10.0.42.10:/home/ubuntu/.ssh/
$ ssh ubuntu@10.0.42.10 cat /home/ubuntu/.ssh/id_rsa.pub >>/home/ubuntu/.ssh/authorized_keys
```

#### 3.b
```
cat ~/.ssh/id_rsa.pub | ubuntu@10.0.42.10 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

#### 4
```
- There might be a firewall rule blocking the SSH port -> I would inspect iptables and act accordingly.
- The SSH daemon might not be running -> I would check whether the service is running.
- The sshd service might be configured to listen on a port different to the default 22. I would, through the physical consel, check sshd configuration.
- I might not be able to reach the server itself (I can test with ping) -> I would check through the physical console whether the NICs are up and running and properly configured.
- There might be some routing misconfiguration -> I would, through the physical console check routes and act accordingly.
- PubkeyAuthentication could be set to 'no' at /etc/ssh/sshd_config -> I would change this value.
```

### Networking and Firewall

#### 1
```
# iptables -I INPUT -p tcp --dport 443 -j ACCEPT
# iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```

#### 2
- Allow connections on external if on port 3022
- Port forward from 3022 to 8080
- Allow forwarding rule
```
# iptables -I INPUT -p tcp -i eth0 --dport 3022 -j ACCEPT
# iptables -t nat -A PREROUTING -p tcp --dport 3022 -j DNAT --to-destination :8080
# iptables -I FORWARD -p tcp -i eth0 --dport 8080 -j ACCEPT
```
:+1: For testing, I started (on my VPS) a netcat server listening on port 8080 and started a telnet client from my localhost to the port 3022. I can then chat both ways.

#### 3
```
# tcpdump -i eth0 port 9055 or port 9300 -w capture.pcap
```

#### 4
```
# wget http://www.ipdeny.com/ipblocks/data/countries/kr.zone
# while IFS= read -r ip; do iptables -A droplist -i eth1 -s $ip -j DROP; done < kr.zone
# iptables -I INPUT -j droplist
# iptables -I OUTPUT -j droplist
# iptables -I FORWARD -j droplist
```

### Systemd

#### 1
Create both files.
```
# cat /etc/systemd/system/redis1.service
[Unit]
Description=Redis persistent key-value database
After=network.target

[Service]
ExecStart=/usr/bin/redis-server --port 1234
Type=simple
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755
LimitNOFILE=10240

[Install]
WantedBy=multi-user.target
```

```
# cat /etc/systemd/system/redis2.service
[Unit]
Description=Redis persistent key-value database
After=redis1.target

[Service]
ExecStart=/usr/bin/redis-server --port 1235
Type=simple
User=redis
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```

```
# systemctl daemon-reload
# systemctl enable redis1
# systemctl enable redis2
```

### Nginx

#### 1