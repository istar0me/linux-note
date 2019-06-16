# Google Authenticator
* 記得先關閉 SELinux 與 防火牆
* 關閉 SELinux 步驟：
```sh
vim /etc/selinux/config
# 設定成 disabled

# 如果設定後不想重新開機，可以輸入以下指令來操作：
# sudo setenforce 0 # 強制生效
# sudo getenforce permissive

systemctl stop firewalld
systemctl disable firewalld
```

* 其他安裝過程接續：[CentOS 7 SSH 兩步驟驗證 (Using Google Authenticator) – Ken Wu](https://kenwu0310.wordpress.com/2016/12/09/centos-7-ssh-%E9%9B%99%E5%9B%A0%E7%B4%A0%E8%AA%8D%E8%AD%89-using-google-authenticator/)

## 移除 `google-authenticator-libpam` 過程
* 移除原因：兩步驟驗證開啟時，無法啟用 ssh 的記住密碼功能

```bash
[root@localhost .ssh]# rpm -qa | grep openssh
openssh-6.4p1-8.el7.x86_64
openssh-clients-6.4p1-8.el7.x86_64
openssh-server-6.4p1-8.el7.x86_64
[root@localhost .ssh]# rpm -e openssh-server
警告：/etc/ssh/sshd_config 已被另存為 /etc/ssh/sshd_config.rpmsave
警告：/etc/pam.d/sshd 已被另存為 /etc/pam.d/sshd.rpmsave
[root@localhost .ssh]# yum install -y openssh-server

# 省略安裝過程

[root@localhost .ssh]# systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:sshd(8)
           man:sshd_config(5)

 6月 05 11:10:49 localhost.localdomain sshd[7623]: Connection closed by 192.168.56.103 [preauth]
 6月 05 11:11:22 localhost.localdomain sshd(pam_google_authenticator)[7628]: Invalid verification code for root
 6月 05 11:11:24 localhost.localdomain sshd[7625]: error: PAM: Authentication failure for root from 192.168.56.103
 6月 05 11:11:32 localhost.localdomain sshd(pam_google_authenticator)[7629]: Invalid verification code for root
 6月 05 11:11:34 localhost.localdomain sshd[7625]: error: PAM: Authentication failure for root from 192.168.56.103
 6月 05 11:11:34 localhost.localdomain sshd[7625]: Postponed keyboard-interactive for root from 192.168.56.103 port 47...auth]
 6月 05 11:11:34 localhost.localdomain sshd[7625]: Connection closed by 192.168.56.103 [preauth]
 6月 05 11:12:30 localhost.localdomain systemd[1]: Stopping OpenSSH server daemon...
 6月 05 11:12:30 localhost.localdomain sshd[3188]: Received signal 15; terminating.
 6月 05 11:12:30 localhost.localdomain systemd[1]: Stopped OpenSSH server daemon.
Hint: Some lines were ellipsized, use -l to show in full.
[root@localhost .ssh]# systemctl start sshd
[root@localhost .ssh]# systemctl status sshd
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since 三 2019-06-05 11:22:32 CST; 1s ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 7867 (sshd)
   CGroup: /system.slice/sshd.service
           └─7867 /usr/sbin/sshd -D

 6月 05 11:22:32 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
 6月 05 11:22:32 localhost.localdomain sshd[7867]: Server listening on 0.0.0.0 port 22.
 6月 05 11:22:32 localhost.localdomain sshd[7867]: Server listening on :: port 22.
 6月 05 11:22:32 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
```