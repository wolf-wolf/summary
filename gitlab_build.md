# 相关问题
## 报错postfix: fatal: parameter inet_interfaces: no local interface found for ::1

启动postfix出错，查看centos中的postfix日志
```shell
more  /var/log/maillog
postfix: fatal: parameter inet_interfaces: no local interface found for ::1
vi  /etc/postfix/main.cf
```

发现配置为：
```shell
inet_interfaces = localhost
inet_protocols = all
```

改成：
```shell
inet_interfaces = all
inet_protocols = all
```
重新启动
```
service postfix start

OK！
```


# 相关链接

 - [centos 7 部署 汉化版 gitlab](https://www.cnblogs.com/straycats/p/7637373.html)
 - [手把手教你 GitLab 的安装及使用](https://www.jianshu.com/p/b04356e014fa)
 - [在Centos中yum安装和卸载软件的使用方法](https://blog.51cto.com/gzmaster/72278)
 - [gitlab搭建与基本使用](https://blog.51cto.com/caiyuanji/2114796?cid=707746)
 - [gitlab安装调试小记](https://peiqiang.net/2014/07/30/install-gitlab.html)
 - [gitlab安装与配置(Centos6.8)](https://www.cnblogs.com/xxoome/p/6366212.html)
 - [配置GitLab笔记](https://www.jianshu.com/p/30d05b81caa2)
 - [postfix报错](https://blog.csdn.net/xiangshanqishi/article/details/23439397)


