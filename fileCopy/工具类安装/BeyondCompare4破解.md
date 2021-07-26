第一种办法（也是最有效的）

![img](https://img2020.cnblogs.com/blog/696983/202107/696983-20210722140441823-1691656198.png)

 

删除C:\Users\用户名\AppData\Roaming\Scooter Software\Beyond Compare 4下的所有文件，重启Beyond Compare 4即可（注意：用户名下的AppData文件夹有可能会被隐藏起来）

第二种办法
删除C:\Program Files\Beyond Compare 4\BCUnrar.dll（安装目录下的BCUnrar.dll文件），这个文件重命名或者直接删除。

第三种办法
修改注册表
1、在搜索栏中输入 regedit ，打开注册表
2、删除项目CacheId ：
HKEY_CURRENT_USER\Software\Scooter Software\Beyond Compare 4\CacheId
