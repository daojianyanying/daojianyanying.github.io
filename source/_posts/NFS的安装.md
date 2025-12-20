---
title: hexo搭建博客(github+hexo)
date: 2025-08-11 15:42:58
update: 2025-08-12 15:42:58
tags: [linux]
index_img: /img/bg/nfs.jpg
excerpt: hexo搭建博客发布到github上
sticky: 100
category: linux
---

```sh
# 服务端
sudo apt-get install nfs-kernel-server
sudo apt-get install nfs-common

sudo vim /etc/exports

/home/test *(rw,sync,no_root_squash) 
/shared_nfs *(rw,sync,no_subtree_check)

sudo /etc/init.d/nfs-kernel-server start

sudo systemctl enable --now nfs-kernel-server
```
```sh
# 查看NFS 支持的版本
sudo cat /proc/fs/nfsd/versions
# nfs v4
# 假设我们设定的NFSv4根目录是 /srv/nfs4
# 我们希望共享两个目录：/export/data 和 /export/media

# 1. 创建必要的目录结构
sudo mkdir -p /srv/nfs4/{data,media}
sudo mkdir -p /export/data /export/media

# 2. 使用 mount --bind 将实际目录绑定到伪根目录下的子目录
# 为了使绑定在重启后依然生效，将以下内容添加到 /etc/fstab
echo "/export/data /srv/nfs4/data none bind 0 0" | sudo tee -a /etc/fstab
echo "/export/media /srv/nfs4/media none bind 0 0" | sudo tee -a /etc/fstab

# 立即执行绑定
sudo mount -a

# 3. 配置 /etc/exports
# 注意：NFSv4的配置是针对伪根目录 /srv/nfs4 进行的
cat << EOF | sudo tee /etc/exports
/srv/nfs4  *(ro,fsid=0,crossmnt,no_subtree_check,sync) # 声明伪根目录为fsid=0，只读
/srv/nfs4/data 192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash) # 对特定网段读写
/srv/nfs4/media *.example.com(ro,sync,no_subtree_check) # 对域名example.com只读
EOF

# 4. 导出配置并重启服务
sudo exportfs -arv
sudo systemctl restart nfs-server

```

```sh
# 客户端
sudo apt-get install nfs-common

```