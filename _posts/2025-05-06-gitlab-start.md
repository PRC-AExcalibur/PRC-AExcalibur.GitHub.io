---
title:  【GIT】GitLab Linux 部署与简单使用指南
categories:
- GIT
tags:
- GitLab
- GIT
---

试试用 GitLab 搭建自己的代码仓库，并介绍简单使用。

---

# 

### 一、环境准备

系统要求

- 推荐Ubuntu 20.04+/CentOS 7+系统
- 内存≥4GB
- 开放HTTP(80)、HTTPS(443)、SSH(22)端口

### 二、安装GitLab

#### 安装过程
首先安装需要的其他环境：
```bash
sudo apt update 
sudo apt install -y ca-certificates curl openssh-server postfix tzdata perl
sudo apt install build-essential # 配置 C/C++ 编译环境，可选
```

安装已经编译好的GitLab, "http://your_server_ip" 替换为自己的服务器域名或IP:

```bash
sudo EXTERNAL_URL="http://your_server_ip" apt install gitlab-ce
```

也可以修改配置文件 `/etc/gitlab/gitlab.rb` 来修改域名等：
```bash
sudo nano /etc/gitlab/gitlab.rb
```

添加/修改域名/IP:
```
external_url 'http://your_server_ip'  # 域名/IP
```

应用配置并启动：
```bash
sudo gitlab-ctl reconfigure  # 初始化配置
sudo gitlab-ctl restart      # 重启服务
sudo gitlab-ctl status    # 查看服务状态
```

之后即可访问域名/IP上的GitLab服务了。


#### 常用管理命令速查

```bash
# 查看所有服务状态
sudo gitlab-ctl status

# 启停控制
sudo gitlab-ctl start           # 启动全部服务
sudo gitlab-ctl stop            # 停止全部服务
sudo gitlab-ctl restart         # 重启全部服务（生产慎用）

# 单组件控制
sudo gitlab-ctl start nginx     # 启动指定组件
sudo gitlab-ctl stop postgresql # 停止指定组件

# 常用操作
sudo nano /etc/gitlab/gitlab.rb # 修改主配置文件
sudo gitlab-ctl reconfigure     # 应用配置变更（必执行）
sudo gitlab-backup create       # 手动备份（默认存储至/var/opt/gitlab/backups）
sudo gitlab-ctl tail	        # 实时查看日志（用于排查问题）
```


### 三、注册登录

#### root 账户重置密码
浏览器输入配置文件external_url设置的地址（默认安装后访问服务器IP或域名：http://your_server_ip）
- 若首次访问自动跳转至密码重置页面，重置密码即可
- root 账户初始密码可以用以下命令获取
```bash
sudo grep '^Password' /etc/gitlab/initial_root_password |awk '{print $2}'
```

修改密码：
```
头像 -> Profile Settings -> Password -> 修改密码 -> Save password
```
重置密码后重新登陆即可。

#### 切换中文
点击头像，然后选择 `Preferences` ；
滚动至 `Localization` ，将 `Language` 修改成“简体中文”，保存。
```markdown
头像 -> Preferences -> Localization -> Language
```

#### 注册其他账户
可以各用户在登陆页面自由注册后，管理员批准注册即可；
也可以管理员在网页端控制面板直接注册。

```markdown
管理员 -> 设置 -> 通用 -> 注册限制

配置用户创建新帐户的方式。
- 已启用注册功能
    任何访问()的用户都可以创建帐户。
- 新的注册需要管理员批准
    任何访问()并创建帐户的用户，在登录之前必须得到管理员的明确批准。仅在启用注册后有效。
```

### 四、简单使用

#### 新建项目
> 注：也可以在组里新建项目，流程一致。
```
项目 -> 新建项目 -> 创建空白项目 -> 新建项目
```
- 项目名称，
  - 必须以小写或大写字母、数字、表情符号或下划线开头。也可以包含点、加号、破折号或空格。
- 可见性级别
  - 私有
  项目访问权限必须明确授予每个用户。如果此项目是一个群组的一部分，访问权限将授予该群组的成员。
  - 内部
  除外部用户外，任何登录用户均可访问该项目。
  - 公开
  无需任何身份验证即可访问该项目。

填写完各项之后，点击新建项目，项目创建成功。

创建项目成功后可以编辑/删除项目，具体参考网页端。

#### 组管理

```
群组 -> 新建群组 -> 创建群组 -> 创建群组
```
- 群组名称
  - 组名必须以字母、数字、表情或下划线开头，可以包含句点、破折号、空格和括号。
如果您打算使用 SCIM 集成，您的组名称不得包含句点，因为这可能会导致错误。

- 可见性级别
  - 私有
群组及其项目只能由成员查看。
  - 内部
除外部用户外，任何登录用户均可查看该群组和任何内部项目。
  - 公开
群组和任何公开项目可以在没有任何身份验证的情况下查看。

填写完各项之后，点击创建群组，群组创建成功。

创建群组成功后可以编辑/删除群组，具体参考网页端。

进入群组后可以管理组成员：
```
群组A -> 管理 -> 成员 -> 邀请成员 ->邀请
```
此时要选择成员权限：
- Guest(匿名用户) 
  - 创建项目、写留言薄。

- Reporter（报告人）
  - 创建项目、写留言薄、拉项目、下载项目、创建代码片段。

- Developer（开发者）
  - 创建项目、写留言薄、拉项目、下载项目、创建代码片段、创建合并请求、创建新分支、推送不受保护的分支、移除不受保护的分支 、创建标签、编写wiki。

- Master（管理者）
  - 创建项目、写留言薄、拉项目、下载项目、创建代码片段、创建合并请求、创建新分支、推送不受保护的分支、移除不受保护的分支 、创建标签、编写wiki、增加团队成员、推送受保护的分支、移除受保护的分支、编辑项目、添加部署密钥、配置项目钩子。

- Owner（所有者）
  - 创建项目、写留言薄、拉项目、下载项目、创建代码片段、创建合并请求、创建新分支、推送不受保护的分支、移除不受保护的分支 、创建标签、编写wiki、增加团队成员、推送受保护的分支、移除受保护的分支、编辑项目、添加部署密钥、配置项目钩子、开关公有模式、将项目转移到另一个名称空间、删除项目。

选择指定成员和权限后点击邀请即可。
邀请成员后权限也可以在成员界面中修改。

---
现在可以自由使用 GitLab 了。