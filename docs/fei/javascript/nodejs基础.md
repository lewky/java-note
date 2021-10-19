# nodeJs基础

## 包管理工具

* `npm`

  ```shell
  # ubuntu install
  sudo apt install npm
  # 设置registry的国内镜像
  npm config set registry https://registry.npm.taobao.org
  # 查看registry
  npm config get registry
  ```

* `yarn`

  ```shell
  # npm install
  npm install yarn -g
  # 设置registry的国内镜像
  npm config set registry https://registry.npm.taobao.org
  # 查看registry
  npm config get registry
  ```

## `CommonJS`规范

每一个`js`文件都是一个模块，js文件的名字就是模块的名字，每个模块内部使用的变量名和函数名互不冲突

一个模块想要对外暴露变量或函数，可以用`module.exports = variable`，还可以`exports.variable = variable`

如果要输出一个键值对对象`{}`，可以利用`exports`这个已存在的空对象`{}`，并继续在上面添加新的键值对

如果要输出一个函数或数组，必须直接对`module.exports`对象赋值

一个模块要引用其他模块暴露的变量，用`var ref = require('module_name')`，就拿到了引用模块的变量。在引用模块时要把相对路径写对，直接写模块名会依次在内置模块、全局模块和当前模块下查找
