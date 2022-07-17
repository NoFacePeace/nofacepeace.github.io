---
title: 初始化 Git
date: 2018-02-05 02:50:37
tags:
- Git
- Github
categories:
- 教程
---

初始化 Git 的一般步骤。

<!-- more -->

# 配置信息

```bash
git config --global user.name "Firstname lastname"
git config --global user.email "your_email@example.com"
git config --global color.ui auto
```

配置信息保存在 "~/.gitconfig" 文件里

# 设置 SSH Key

```bash
$ ssh-keygen -t rsa -C "your_email@example.com"
Generation public/private rsa key pair.
Enter file in which to save the key
(/Users/your_user_directory/.ssh/id_rsa):  #按回车键
Enter passphrase (empty for no passphrase):  #输入密码
Enter same passphrase again:  #再次输入密码
```

id_rsa 文件是私有密钥，id_rsa.pub 是公开密钥

# 添加公开密钥

1. 点击账户设定按钮（Settings）
2. 选择  SSH  and   GPG  keys
3. 点击 New SSH key
4. Title 输入密钥名称
5. Key 粘贴 id_rsa.pub 内容
6. 收到提示“公共密钥添加完成“的邮件
7. 认证和通信
    ```bash
    ssh -T git@github.com
    # 出现如下结果即为成功
    Hi hirocastest!You've successfully authenticated,but GitHub does no provide shell access.
    ```
