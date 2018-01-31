# Ubuntu
查看内存使用状况 free -h
查看磁盘使用状况 df -h
解压 tar -zxvf 文件名.tar.gz
查看当前系统正在运行的所有进程 ps -aux|less

[Ubuntu下的MySQL](https://www.cnblogs.com/boshen-hzb/p/5889633.html)

lsof -i:端口号   列出在某端口运行的进程  [source](http://blog.csdn.net/sela0708/article/details/72846334)

lsof命令详解 [source](https://www.douban.com/note/531569168/)

**将日志直接打印到控制台时，ctrl+c 即可退出**

## linux下打开与关闭tomcat 实时查看tomcat运行日志

一，linux下打开与关闭tomcat

启动：一般是执行sh tomcat/bin/startup.sh
停止：一般是执行sh tomcat/bin/shutdown.sh脚本命令
 
查看：执行ps -ef |grep tomcat 输出：
*** 5144   。。。等等.Bootstrap start
 
说明tomcat已经正常启动， 5144 就为进程号 pid = 5144

杀死：kill -9 5144

二，linux下实时查看tomcat运行日志

1、先切换到：cd tomcat/logs

2、tail -f catalina.out

3、这样运行时就可以实时查看运行日志了

Ctrl+c 是退出tail命令。

## [idea部署项目到远程tomcat](http://blog.csdn.net/tianjun2012/article/details/52795202)

cp catalina.sh /etc/init.d/tomcat
