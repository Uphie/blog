---
title: TencentOS安装gitlab
author: Uphie
date: 2023-06-01 21:03:00 +0800
categories: [技术]
tags: [linux,git,腾讯云,安装]
math: true
toc: true
---

由于之前承担 git 的腾讯云服务器配置低性能较差，这次需要更换一个服务器，再安装一次 gitlab，记录一下过程。

本次安装免费的社区版，其自带 nginx 、Postgresql、redis，因此注意下避免重复安装。

本机系统版本：
```
$ cat /etc/redhat-release
TencentOS Server release 3.1 (Final)
```

# 安装依赖

```
$ yum install -y curl policycoreutils-python openssh-server perl
```

如果出现下面这个错误，
> Error: Unable to find a match: policycoreutils-python

配置下 epel 源再重新安装：
```
$ yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

# 配置 gitlab 软件源

下载软件源配置脚本：
```
$ wget  https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh
```

执行安装脚本，由于 TencentOS 不是常见的 Linux 发行版，gitlab 没有相应的软件源，不能直接执行。但由于 TencentOS 兼容 CentOS8，可以执行脚本变量来执行脚本：
```
[root@VM-1-32-tencentos ~]# os=el dist=8 bash ./script.rpm.sh
Detected operating system as el/8.
Checking for curl...
Detected curl...
Downloading repository file: https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/config_file.repo?os=el&dist=8&source=script
done.
Installing yum-utils...
TencentOS Server 3.1 - TencentOS                                                                                                                                                                                                                                                                                                             16 MB/s | 3.1 MB     00:00
TencentOS Server 3.1 - Updates                                                                                                                                                                                                                                                                                                               69 MB/s |  24 MB     00:00
TencentOS Server 3.1 - TencentOS-AppStream                                                                                                                                                                                                                                                                                                   32 MB/s |  17 MB     00:00
TencentOS Server 3.1 - Base                                                                                                                                                                                                                                                                                                                 5.5 kB/s | 257  B     00:00
TencentOS Server 3.1 - AppStream                                                                                                                                                                                                                                                                                                            5.4 kB/s | 257  B     00:00
TencentOS Server 3.1 - Extras                                                                                                                                                                                                                                                                                                               115 kB/s | 6.7 kB     00:00
TencentOS Server 3.1 - PowerTools                                                                                                                                                                                                                                                                                                           4.6 kB/s | 257  B     00:00
Docker CE Stable - x86_64                                                                                                                                                                                                                                                                                                                   800 kB/s |  46 kB     00:00
Extra Packages for TencentOS Server 3.1 - x86_64                                                                                                                                                                                                                                                                                             45 MB/s |  14 MB     00:00
Extra Packages for TencentOS Server 3.1 Modular - x86_64                                                                                                                                                                                                                                                                                    7.2 MB/s | 733 kB     00:00
gitlab_gitlab-ce-source                                                                                                                                                                                                                                                                                                                     135  B/s | 862  B     00:06
gitlab_gitlab-ce-source                                                                                                                                                                                                                                                                                                                     2.1 kB/s | 3.1 kB     00:01
导入 GPG 公钥 0x51312F3F:
 Userid: "GitLab B.V. (package repository signing key) <packages@gitlab.com>"
 指纹: F640 3F65 44A3 8863 DAA0 B6E0 3F01 618A 5131 2F3F
 来自: https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
gitlab_gitlab-ce-source                                                                                                                                                                                                                                                                                                                     2.9 kB/s | 4.6 kB     00:01
导入 GPG 公钥 0xF27EAB47:
 Userid: "GitLab, Inc. <support@gitlab.com>"
 指纹: DBEF 8977 4DDB 9EB3 7D9F C3A0 3CFC F9BA F27E AB47
 来自: https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
gitlab_gitlab-ce-source                                                                                                                                                                                                                                                                                                                      40  B/s | 296  B     00:07
软件包 yum-utils-4.0.21-14.1.tl3.noarch 已安装。
依赖关系解决。
无需任何处理。
完毕！
Generating yum cache for gitlab_gitlab-ce...
导入 GPG 公钥 0x51312F3F:
 Userid: "GitLab B.V. (package repository signing key) <packages@gitlab.com>"
 指纹: F640 3F65 44A3 8863 DAA0 B6E0 3F01 618A 5131 2F3F
 来自: https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
导入 GPG 公钥 0xF27EAB47:
 Userid: "GitLab, Inc. <support@gitlab.com>"
 指纹: DBEF 8977 4DDB 9EB3 7D9F C3A0 3CFC F9BA F27E AB47
 来自: https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey/gitlab-gitlab-ce-3D645A26AB9FBD22.pub.gpg
Generating yum cache for gitlab_gitlab-ce-source...

The repository is setup! You can now install packages.
```

# 安装

安装前先将自己的域名解析到当前机器，然后指定你的域名，如 git.example.com，安装 gitlab：
```
$ EXTERNAL_URL="git.example.com" yum install -y gitlab-ce
```

安装完成后，gitlab 已经为你配置好了 nginx ，无需再做配置。

# 配置

获取 gitlab 生成的 root 用户密码：
```
$ cat /etc/gitlab/initial_root_password
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: huEE18anOuWzTFr/KJQbSxxxxxxxxxxxxxx

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
**
```

使用上面的 `root` 用户 和 `huEE18anOuWzTFr/KJQbSxxxxxxxxxxxxxx` 密码登录 gitlab，然后就可以添加用户和创建仓库了。


注：
- nginx 日志和配置目录：`/var/opt/gitlab/nginx/conf/`；
- gitlab 数据目录：`/var/opt/gitlab/git-data/`；
- postgresql 数据和配置目录：`/var/opt/gitlab/postgresql/`；
- redis 数据和配置目录：`/var/opt/gitlab/redis/`；