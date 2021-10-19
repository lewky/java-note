# Maven

## `pom(Project Object Model)`

`pom.xml`

* 根标签`<project>`
* `GAV`信息

  ```xml
  <project>
    ...
    <!-- GAV配置 -->
    <groupId>com.fjh</groupId>
    <artifactId>template</artifactId>
    <version>1.0-SNAPSHOT</version>
    ...
  </project>
  ```

* 父级`maven`工程

  ```xml
  <project>
  ...
    <parent>
      <!-- spring boot -->
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.4.3</version>
    </parent>
  ...
  </project>
  ```

所有的`pom`文件都继承一个`super pom`文件，`super pom`文件里包含一些默认的配置，例如`Maven`的中央资源库地址

`mvn help:effective-pom`，查看该项目完整的`pom`文件信息

## `maven`的工程类型

* `pom`

  一般作为父级工程存在，主要是进行统一的依赖版本管理，一般不引入依赖库

* `jar`

  用于打包依赖库的工程

* `war`

  用于打包`web`工程

## 为不同的环境定制构建方式

```xml
<project>
  ...
  <profiles>
      <profile>
          <id>dev_evn</id>
          <properties>
              <db.driver>com.mysql.jdbc.Driver</db.driver>
              <db.url>jdbc:mysql://localhost:3306/dev_db</db.url>
              <db.username>root</db.username>
              <db.password>root</db.password>
          </properties>
      </profile>
      <profile>
          <id>test_evn</id>
          <properties>
              <db.driver>com.mysql.jdbc.Driver</db.driver>
              <db.url>jdbc:mysql://localhost:3306/test_db</db.url>
              <db.username>root</db.username>
              <db.password>root</db.password>
          </properties>
      </profile>
  </profiles>
  ...
</project>
```

激活指定的`profile`

* 利用`mvn`的`-P`参数指定`profile`

  `mvn clean install -Pdev_env,test_evn`，指定多个`profile`

* 在`settings.xml`中配置`activeProfiles`标签

  ```xml
  <settings>
    ...
    <activeProfiles>
        <activeProfile>dev_evn</activeProfile>
    </activeProfiles>
    ...
  </settings>
  ```

* 利用系统属性激活

  ```xml
  <project>
    ...
    <profiles>
      <profile>
          <activation>
              <property>
                  <name>profileProperty</name>
                  <value>dev</value>
              </property>
          </activation>
      </profile>
    </profiles>
    ...
  </project>
  ```

  也可以使用`mvn clean install -DprofileProperty=dev`，`-D`指定激活的属性和值，`-P`指定的是`profile`的`id`

* 利用系统环境激活

  ```xml
  <project>
    ...
    <profiles>
      <profile>
          <activation>
              <os>
                  <name>Window XP</name>
                  <family>Windows</family>
                  <arch>x86</arch>
                  <version>5.1.2600</version>
              </os>
          </activation>
      </profile>
    </profiles>
    ...
  </project>
  ```

* 文件存在与否激活

  ```xml
  <project>
    ...
    <profiles>
      <profile>
          <activation>
              <file>
                  <missing>t1.properties</missing>
                  <exists>t2.properties</exists>
              </file>
          </activation>
      </profile>
    </profiles>
    ...
  </project>
  ```

* 默认激活的`profile`

  ```xml
  <project>
    ...
    <profiles>
      <profile>
          <activation>
              <activeByDefault>true</activeByDefault>
          </activation>
      </profile>
    </profiles>
    ...
  </project>
  ```

`mvn help:active-profiles`，查看当前激活的`profile`

`mvn help:all-profiles`，查看所有的`profile`

## 依赖资源库

* 本地资源库

  默认路径为`~/.m2`，也可以通过修改`settings.xml`中的`<localRepository>`来进行修改，`Maven`会先从本地资源库寻找依赖库

* 中央资源库

  本地资源库不存在的依赖库，`Maven`再从`Maven`中央资源库下载依赖库

* 远程资源库

  本地资源库和中央资源库都不存在的依赖库，再从远程资源库中找

  ```xml
  <!-- 在settings.xml中设置远程资源库 -->
  <settings>
    ...
    <repositories>
      <repository>
        <id>companyname.lib1</id>
        <url>http://download.companyname.org/maven2/lib1</url>
      </repository>
    </repositories>
    ...
  </settings>
  ```

## 插件

每个生命周期中包含着一系列的阶段`phase`，这些`phase`就相当于`Maven`提供的统一接口，这些`phase`的实现行为由`Maven`的插件来决定

`mvn [plugin-name]:[goal-name]`

### 插件类型

* Build plugins

  在构建时执行，并在`pom.xml`的元素中配置

* Reporting plugins

  在网站生成过程中执行，并在`pom.xml`的元素中配置

常用插件

* clean
* compile
* jar

## 快速创建项目

`mvn archetype:generate "-DgroupId=com.companyname.bank" "-DartifactId=consumerBanking" "-DarchetypeArtifactId=maven-archetype-quickstart" "-DinteractiveMode=false"`

## 引入资源库中的依赖库

```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>jiehuifang.com</groupId>
      <artifactId>test</artifactId>
      <version>1.0</version>
      <scop>system</scop>
      <systemPath>${basedir}\src\lib\test.jar</systemPath>
    </dependency>
  </dependencies>
  ...
</project>
```

### 依赖范围`scop`

`maven`在编译、测试、运行(包含打包)阶段`phase`中所需的依赖是不一样的，`maven`通过三种`classpath`来实现不同阶段`phase`引入不同的依赖

* `compile classpath`
* `test classpath`
* `runtime classpath`

依赖范围的类型

* `compile`

  对`compile classpath`、`test classpath`、`runtime classpath`都起效

  `scop`的默认类型

* `test`

  只对`test classpath`起效

* `provided`

  对`compile classpath`、`test classpath`起效

* `runtime`

  对`test classpath`、`runtime classpath`起效

* `system`

  对`compile classpath`、`test classpath`起效

  引用非`maven`资源库的依赖库，用`<systemPath>`指定依赖库的路径

* `import`

  对`compile classpath`、`test classpath`、`runtime classpath`都不起效，只导入依赖的信息

### 排除依赖 `exclusions`

```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>com.apple</groupId>
      <artifactId>B</artifactId>
      <version>2.3</version>
      <exclusions>
        <exclusion>
          <groupId>com.google</groupId>
          <artifactId>C</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
  ...
</project>
```

### 可选依赖`option`

A库依赖B库，B库依赖C库(C库为可选依赖)，则A库不会依赖C库

```xml
<project>
  ...
  <dependencies>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
  </dependencies>
  ...
</project>
```

## 依赖管理

```xml
<project>
  ...
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
  ...
</project>
```

### `dependencies`与`dependencyManagement`

* `dependencies`

  引入依赖，在父级工程中引入的依赖，子级工程会直接继承父级工程中引入的依赖，不用重复引入依赖

* `dependencyManagement`

  声明依赖，不会引入依赖，在父级工程中声明的依赖，子级工程在引入依赖时，可以直接继承父级工程中声明依赖的`version`和`scop`而不用具体指定值

## 快照与版本

* 版本

  `<version>1.0</version>`

  如果`maven`以前下载过指定版本的依赖库，`maven`将不会再从资源库中下载新的依赖库(相同版本)，若想要下载新版本的依赖库，则要修改版本号

* 快照

  `<version>1.0-SNAPSHOT</version>`

  `maven`将自动下载最新的快照
