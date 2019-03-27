---
title: SSH用户等效性配置
date: 2016-05-20 14:43:49
tags: [linux]
categories: [linux,linux基本配置]
---
#SSH用户等效性配置 #
以下均以oracle用户执行
linuxrac1
[oracle @linuxrac1 ~]$mkdir ~/.ssh
[oracle @linuxrac1 ~]$chmod 755 ~/.ssh
[oracle @linuxrac1 ~]$ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/oracle/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/oracle/.ssh/id_rsa.
Your public key has been saved in /home/oracle/.ssh/id_rsa.pub.
The key fingerprint is:
e9:2b:1a:2b:ac:5f:91:be:0f:84:17:d7:bd:b7:15:d2 oracle@linuxrac1
[oracle @linuxrac1 ~]$ssh-keygen -t dsa
Generating public/private dsa key pair.
Enter file in which to save the key (/home/oracle/.ssh/id_dsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/oracle/.ssh/id_dsa.
Your public key has been saved in /home/oracle/.ssh/id_dsa.pub.
The key fingerprint is:
f5:0f:f5:0c:55:37:6a:08:ef:06:07:37:65:25:4a:15 oracle@linuxrac1
 
linuxrac2
[oracle @linuxrac2 ~]$ mkdir ~/.ssh
[oracle @linuxrac2 ~]$ chmod 755 ~/.ssh
[oracle @linuxrac2 ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/oracle/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/oracle/.ssh/id_rsa.
Your public key has been saved in /home/oracle/.ssh/id_rsa.pub.
The key fingerprint is:
56:47:a0:94:67:44:d9:31:12:57:44:08:9d:84:25:a1 oracle@linuxrac2
 
[oracle @linuxrac2 ~]$ ssh-keygen -t dsa
Generating public/private dsa key pair.
Enter file in which to save the key (/home/oracle/.ssh/id_dsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/oracle/.ssh/id_dsa.
Your public key has been saved in /home/oracle/.ssh/id_dsa.pub.
The key fingerprint is:
ae:f0:06:77:62:33:86:dc:f4:0d:d9:c6:38:5e:cb:61 oracle@linuxrac2
 
以上用默认配置,一路回车即可
linuxrac1
cat ~/.ssh/*.pub >> ~/.ssh/authorized_keys
ssh oracle@linuxrac2 cat ~/.ssh/*.pub >> ~/.ssh/authorized_keys
或
ssh oracle@linuxrac2 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
ssh oracle@linuxrac2 cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
[oracle@linuxrac1 ~]$ cd .ssh
[oracle@linuxrac1 .ssh]$ ll
total 48
-rw-r--r-- 1 oracle oinstall 2008 Sep 25 02:20 authorized_keys
-rw------- 1 oracle oinstall  668 Sep 25 02:09 id_dsa
-rw-r--r-- 1 oracle oinstall  606 Sep 25 02:09 id_dsa.pub
-rw------- 1 oracle oinstall 1675 Sep 25 02:09 id_rsa
-rw-r--r-- 1 oracle oinstall  398 Sep 25 02:09 id_rsa.pub
-rw-r--r-- 1 oracle oinstall  404 Sep 25 02:20 known_hosts
linuxrac2
cat ~/.ssh/*.pub >> ~/.ssh/authorized_keys
ssh oracle@linuxrac1 cat ~/.ssh/*.pub >> ~/.ssh/authorized_keys
或
ssh oracle@linuxrac1 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
ssh oracle@linuxrac1 cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
 

建立等效性 rac1,rac2双节点执行
[oracle@linuxrac1 ~]$ exec ssh-agent $SHELL
[oracle@linuxrac1 ~]$ ssh-add
Identity added: /home/oracle/.ssh/id_rsa (/home/oracle/.ssh/id_rsa)
Identity added: /home/oracle/.ssh/id_dsa (/home/oracle/.ssh/id_dsa)
[oracle@linuxrac1 ~]$ ssh linuxrac1 date
[oracle@linuxrac1 ~]$ ssh linuxrac1-priv date
[oracle@linuxrac1 ~]$ ssh linuxrac2 date
[oracle@linuxrac1 ~]$ ssh linuxrac2-priv date
 
[oracle@linuxrac2 ~]$ exec ssh-agent $SHELL
[oracle@linuxrac2 ~]$ ssh-add
Identity added: /home/oracle/.ssh/id_rsa (/home/oracle/.ssh/id_rsa)
Identity added: /home/oracle/.ssh/id_dsa (/home/oracle/.ssh/id_dsa)

The authenticity of host '<host>' can't be established.  
解决办法：在连接目标机上执行ssh  -o StrictHostKeyChecking=no  xxxx(机器名)



