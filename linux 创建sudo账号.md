***内容来源自网络，只是做了整理***
## 方案一 ##
### root登录 ###
ssh root@server_ip_address
### 新增用户 ###
adduser username
### 设置密码 ###
passwd username
输入两次密码
### 修改帐户所属分组，有的linux中wheel分组用户默认有sudo权限 ###
usermod -aG wheel username
### 确定/etc/sudoers 文件下如下行是启用的 ###
%wheel  ALL=(ALL)       ALL
### 不在使用该功能，则可以使用 ###
userdel username

## 方案二 ##
### 登录拥有管理员权限的帐号 ###
### 添加文件的写权限 ###
chmod u+w /etc/sudoers
### 编辑/etc/sudoers 找到这一行 ###
root ALL=(ALL) ALL
在该行下面添加：
username ALL=(ALL) ALL
### 撤销文件的写权限 ###
chmod u-w /etc/sudoers
## 注意 ##
方案一和方案二不能够混合使用，混合使用反而达不到效果

