# linux启动执行

## 系统在引导期间执行的脚本

/etc/rc.d/rc.local

## 能被service识别的init script

/etc/rc.d/init.d

```bash
#!/bin/bash
# chkconfig: 2345 10 90
# description: a sample service
# config: /etc/sysconfig/sample
start()
{
    #some script
}
stop()
{
    #some script
}
usage()
{
    #some script
}
case "$1" in
    start)
        start
    ;;
    stop)
        stop
    ;;
    *)
        usage
        exit 1
    ;;
esac
exit 0
```

> chkconfig：2345 10 90，该注释用于给chkconfig命令识别，2345为以start为参数传递给脚本的Linux运行级别，10表示启动优先级，90表示停止优先级，优先级范围是0~100，数字越大，优先级越低

## Linux运行级别

* 0：关机
* 1：单用户模式
* 2：无网络连接的多用户命令行模式
* 3：有网络连接的多用户命令行模式
* 4：不可用
* 5：带图形界面的多用户模式
* 6：重新启动

Linux的七个运行级别分别对应7个目录(/etc/rc.d/rc*.d)，当某个运行级别加载时，就会执行在对应目录下面的脚本

> * K10serviceName:表示在该运行级别下，系统加载时以stop为参数，调用serviceName脚本，优先级为10
> * S20serviceName:表示在该运行级别下，系统加载时为start为参数，调用serviceName脚本，优先级为20

## chkconfig

chkconfig命令能识别/etc/rc.d/init.d中的脚本，并识别脚本注释在相应的/etc/rc.d/rc*.d中建立软连接

```bash
# 常用命令
chkconfig --add serviceName
chkconfig --list serviceName
chkconfig --level 35 serviceName on
```
