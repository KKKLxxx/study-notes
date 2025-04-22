# CentOS7.6修改MySQL密码

## 1、通过修改配置文件实现免密码登录

用`vi /etc/my.cnf`命令打开MySQL的配置文件，找到`skip-grant-tables`这一行，将其前面的注释符`#`删掉，这样可以免密码登录

然后重启MySQL：`sudo service mysqld restart`

![image-20211118174928309](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111181749395.png)

## 2、免密码登录MySQL

通过`mysql -u root -p`登录，当提示输入密码时直接回车即可

更换当前数据库：`use mysql`;

查看用户信息：`select host, user, authentication_string, plugin from user;`

更改root用户信息：`update user set authentication_string='' where user='root';`这里将密码置为空

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111181755648.png" alt="image-20211118175510603" style="zoom: 67%;" />

## 3、重启密码验证

在`mysql >`后输入`quit`退出MySQL

再通过`vi /etc/my.cnf`命令打开配置文件，将`skip-grant-tables`这一行的注释复原

重启MySQL：`sudo service mysqld restart`

## 4、设置密码

通过`mysql -u root -p`登录，当提示输入密码时直接回车即可，因为此时密码为空

修改root用户密码：`ALTER user 'root'@'%' IDENTIFIED BY 'root123##ROOT';`

修改完成

## 5、注意事项

**密码规则**：大于8位，包含英文大小写、数字、特殊字符

前面3步都是在忘了原先的密码的情况下的额外操作，如果记得旧密码，可以直接登录，然后通过ALTER语句修改密码