# IO流

## 输入流与输出流

* 输入流的抽象类`InputStream`，`Reader`
* 输出流的抽象类`OutputStream`，`Writer`

> 一般用装饰者模式来添加流对象的功能

## 字节流与字符流

适配类`InputStreamReader`和`OutputStreamWriter`用于把字节流包装成字符流

包装成字符流，主要是为了在IO操作中都支持unicode(16位)，java本身的char也是unicode(16位)，而有一些老版本的IO操作只支持8位的字节流

## 常规读写文件

```java
public class Test {
    // 带缓存区的读取文件内容，并将内容输出到bak文件中
    static void bufferCase() {
        // 嵌套流对象，添加功能
        try (BufferedReader in = new BufferedReader(new FileReader("Test.java"));
                PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("Test.java.bak")));) {
            String temp;
            while ((temp = in.readLine()) != null) {
                // readLine()会删掉所有的换行符
                out.println(temp);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        bufferCase();
    }
}
```

## 标准IO流

`System.out`和`System.err`已被包装成`PrintStream`对象

`System.in`是未被包装过的`InputStream`，使用之前必须进行包装

### 标准IO流重定向

* `System.setIn();// 重定向标准输入流`
* `System.setOut();// 重定向标准输出流`
* `System.setErr();// 重定向标准异常信息输出流`

> 标准IO流重定向的流都是字节流，而不是字符流

## 获取进程的IO流

```java
public class Test {
    public static void main(String[] args) throws IOException {
        String[] cmd = { "java", "-version" };
        Process process = new ProcessBuilder(cmd).start();
        try (BufferedReader in1 = new BufferedReader(new InputStreamReader(process.getInputStream()));
                BufferedReader in2 = new BufferedReader(new InputStreamReader(process.getErrorStream()));) {
            String temp1 = null;
            String temp2 = null;
            while ((temp1 = in1.readLine()) != null || (temp2 = in2.readLine()) != null) {
                if (temp1 != null) {
                    System.out.println(temp1);
                }
                if (temp2 != null) {
                    System.out.println(temp2);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 压缩流

* 压缩输入流(解压)：`GZIPInputStream`，`ZipInputStream`
* 压缩输出流(压缩)：`GZIPOutputStream`，`ZipOutputStream`
