# 安全保护机制

## java security manager

启用`java security manager`来控制运行代码的权限，防止未知的java程序运行恶意代码(删除系统文件，关闭系统)

默认的`java security manager`的配置文件是`java.policy`

用`Jvm`参数来启动`java security manager`

```text
-Djava.security.manager
// 指定自定义的配置文件，自定义的配置文件只对默认的配置文件进行补充
-Djava.security.manager -Djava.security.policy="E:/java.policy"

// java.policy
// permission java.util.PropertyPermission "os.name", "read"; // 不能读取操作系统的名字

// java 程序报错
Exception in thread "main" java.security.AccessControlException: access denied ("java.util.PropertyPermission" "os.name" "read")
```
