## IO(网络通信) and NIO(网络通信)

IO(JDK 1.4)通信流程:

1. ServerSocket打开端口，等待请求<font color="#D98880">(阻塞)</font>
2. Socket与ServerSocket建立连接，Socket需要等待建立连接的完成(TCP3次握手...)<font color="#D98880">(阻塞)</font>。Socket完成连接才可以写入。ServerSocket才可以收取数据。
3. Socket通过OutputStream写入数据(假想写入还需要到内核，通过系统调用发送数据，通过网络)<font color="#D98880">(日常阻塞)</font>，传输到ServerSocket。
4. ServerSocket已经完成连接过程，接收写入的数据之前是内核先接收数据，然后复制到用户空间(ServerSocket读取数据之前的数据准备)<font color="#D98880">(日常阻塞)</font>
5. ServerSocket通过客户端的Socket然后由InputStream读取数据。即使通过Buffer包装读取，这个过程也是在用户空间操作(用户来操作)(读取准备完成，Socket是可读状态)
6. ServerSocket通过客户端的Socket然后由OutputStream写入数据。<font color="#D98880">写入又是日常阻塞</font>

总结： <br>

1. Socket连接到ServerSocket，写入数据之前需要等到连接完成。所以写入之前只有等待连接完成，这个过程Socket阻塞。
2. ServerSocket接入连接已经完成。从Socket中读取数据，Socket写入数据到网络连接到数据复制到用户空间(可读之前的操作)，这个过程阻塞。
3. Socket & ServerSocket通过单向的Stream传输数据，写入要GetOutputStream,读取要GetInputStream
4. 到真正操作的时候还是One By One,Buffer包装是通过用户代码完成。

NIO(JDK 1.5)通信流程：
