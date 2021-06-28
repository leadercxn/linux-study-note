# samba 服务器的搭建

## 链接介绍
1. [samba服务器的搭建和配置](https://www.linuxidc.com/Linux/2018-11/155466.htm)


## 步骤
1. 更新当前软件
    ```
        sudo apt-get upgrade 
        sudo apt-get update 
        sudo apt-get dist-upgrade
    ```
2. 安装samba服务器
    ```
        sudo apt-get install samba samba-common
    ```
3. 选定分享的samba文件目录
    /home/cxn
4. 添加用户cxn和设置密码123456
    sudo smbpasswd -a cxn

5. 配置samba的配置文件
    sudo nano /etc/samba/smb.conf

    ```
        [cxn]
        comment = cxn folder
        browseable = yes
        path = /home/cxn
        create mask = 0700
        directory mask = 0700
        valid users = cxn
        force user = cxn
        force group = cxn
        public = yes
        available = yes
        writable = yes
    ```
6. samba 服务器重启
    sudo service smbd restart

7. 在win下的,在我的电脑输入如下路径：\\192.168.1.101\cxn ，账号 cxn ，密码(刚设定的 123456)
    即可在window下查看到对应的共享文件目录

8. 在我的电脑设置 映射网络驱动器，并添加对应局域网内的路径
    \\192.168.1.101\cxn