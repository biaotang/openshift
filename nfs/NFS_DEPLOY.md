#### NFS服务部署

创建nfs目录

```
mkdir -p /var/export/dbvol
chown nfsnobody:nfsnobody /var/export/dbvol
chmod 777 /var/export/dbvol
```

编辑/etc/exports，增加以下命令

```
/var/export/dbvol *(rw,async,all_squash)
```

配置iptables（/etc/sysconfig/iptables）, 添加以下内容

```
# BEGIN NFS server
-A OS_FIREWALL_ALLOW -p tcp -m state --state NEW -m tcp --dport 53248 -j ACCEPT
-A OS_FIREWALL_ALLOW -p tcp -m state --state NEW -m tcp --dport 50825 -j ACCEPT
-A OS_FIREWALL_ALLOW -p tcp -m state --state NEW -m tcp --dport 20048 -j ACCEPT
-A OS_FIREWALL_ALLOW -p tcp -m state --state NEW -m tcp --dport 2049 -j ACCEPT
-A OS_FIREWALL_ALLOW -p tcp -m state --state NEW -m tcp --dport 111 -j ACCEPT
# END NFS server
```

编辑/etc/sysconfig/nfs

```
# Optional arguments passed to rpc.mountd. See rpc.mountd(8)
RPCMOUNTDOPTS="-p 20048"
#
# Optional arguments passed to rpc.statd. See rpc.statd(8)
STATDARG="-p 50825"
```

设置nfs对虚拟化支持

```
 setsebool -P virt_use_nfs=true
 systemctl enable rpcbind nfs-server
 systemctl start rpcbind nfs-server
```

在node节点配置

```
setsebool -P virt_use_nfs=true
```

测试

```
mkdir /root/test
mount -t nfs nfs-server.example.com:/var/export/dbvol  /root/test
cd /root/test
touch abc
echo "success" > abc

more abc

umount /root/test
```

