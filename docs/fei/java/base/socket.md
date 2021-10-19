# Socket编程

## java socket

两个java应用程序可通过一个双向的网络通信连接实现数据交换，这个双向链路的一端被称为一个`socket`

`socket`通常用来实现`client-server`连接

建立连接所需的寻址信息为远程计算机的ip地址和端口号(`port number`)，端口号又分`udp`端口和`tcp`端口，各有`2^16 = 65536`个

## tcp socket 通信模型

```java
// tcp server
class TcpServer {
    public static void main(String[] args) throws Exception {
        int port = 8888;
        ServerSocket serverSocket = new ServerSocket(port);
        Socket socket = serverSocket.accept();
        BufferedReader bufferedReader = new BufferedReader(
                new InputStreamReader(socket.getInputStream(), StandardCharsets.UTF_8));
        String line;
        while ((line = bufferedReader.readLine()) != null) {
            System.out.println(line);
        }
    }
}

// tcp client
class TcpClient {
    public static void main(String[] args) throws Exception {
        Socket socket = new Socket("127.0.0.1", 8888);
        BufferedWriter bufferedWriter = new BufferedWriter(
                new OutputStreamWriter(socket.getOutputStream(), StandardCharsets.UTF_8));
        bufferedWriter.write("hello server");
        bufferedWriter.flush();
        bufferedWriter.close();
    }
}
```

## udp datagram socket 通信模型

```java
// udp server
class UdpServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket datagramSocket = new DatagramSocket(8888);
        byte[] buffer = new byte[1024];
        DatagramPacket datagramPacket = new DatagramPacket(buffer, buffer.length);
        while (true) {
            Thread.sleep(3000);
            datagramSocket.receive(datagramPacket);
            System.out.println(new String(buffer, 0, datagramPacket.getLength()));
        }
    }
}

//udp client
class UdpClient {
    public static void main(String[] args) throws Exception {
        byte[] buffer = "hello server".getBytes();
        DatagramPacket datagramPacket = new DatagramPacket(buffer, buffer.length,
                new InetSocketAddress("127.0.0.1", 8888));
        DatagramSocket datagramSocket = new DatagramSocket(9999);
        datagramSocket.send(datagramPacket);
        datagramSocket.close();
    }
}
```
