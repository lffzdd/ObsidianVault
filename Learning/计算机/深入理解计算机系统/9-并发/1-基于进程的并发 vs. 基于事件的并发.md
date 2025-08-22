[[并发中的经典问题:
- 竞争
- 死锁: 
...
# 事件驱动
在基于事件的并发模型中，**“事件”**可以理解为某种异步发生的**可检测**的情况，比如：
- **I/O 事件**：例如，客户端发送了数据，服务器的 `socket` 变得可读（可以调用 `recv()` 读取数据）。
- **定时事件**：比如，某个定时器到期了，需要执行某个回调函数。
- **信号事件**：例如，进程收到一个特定的信号（如 `SIGINT`）。
- **用户定义的事件**：程序内部触发的某种特定状态变化。
### **基于进程的并发 vs. 基于事件的并发**
你之前学的基于进程的并发是**通过多个进程，每个进程独立处理一个连接**，并行性依靠操作系统的进程调度。而**基于事件的并发**则是单进程/单线程，但使用**事件驱动的方式**，在一个循环里**不断检查事件并处理**。
### **事件驱动的核心机制**
1. **事件循环**：通常程序会有一个主循环，负责检测事件并执行相应的处理逻辑。
2. **事件检测**：
    - **多路复用 I/O 机制**：使用 `select()`, `poll()`, `epoll()`, `kqueue()` 这样的系统调用检查多个 socket 是否有事件发生。
3. **事件处理**：
    - **回调函数**：一旦某个事件发生，就调用相应的函数来处理。
4. **非阻塞 I/O**：
    - 由于只有一个进程/线程，所有 I/O 操作都是**非阻塞**的，避免程序等待某个 I/O 操作完成而停止执行其他任务。
### **示例：简单事件驱动服务器**
一个使用 `select()` 监听多个客户端连接的简单服务器：
```python
import socket
import select
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('0.0.0.0', 12345))
server_socket.listen(5)
server_socket.setblocking(False)  # 设置为非阻塞模式
sockets_list = [server_socket]  # 监听的 socket 列表
clients = {}  # 存储客户端连接
print("Server is listening…")
while True:
    readable, _, _ = select.select(sockets_list, [], [])  # 监听可读事件
    for sock in readable:
        if sock == server_socket:
            # 处理新连接
            client_socket, client_address = server_socket.accept()
            client_socket.setblocking(False)
            sockets_list.append(client_socket)
            clients[client_socket] = client_address
            print(f"New connection from {client_address}")
        else:
            # 处理客户端消息
            try:
                data = sock.recv(1024)
                if data:
                    print(f"Received from {clients[sock]}: {data.decode()}")
                    sock.send(b"Message received!")
                else:
                    # 断开连接
                    print(f"Client {clients[sock]} disconnected")
                    sockets_list.remove(sock)
                    del clients[sock]
                    sock.close()
            except:
                pass
```
### **总结**
- **“事件”指的是 I/O 变化、超时、信号等状态变化，可以被检测和处理。**
- **事件驱动编程使用事件循环、非阻塞 I/O 和回调函数来管理并发。**
- **相比于基于进程的并发，它减少了进程/线程的开销，适合高并发的 I/O 密集型应用。**
你对这个概念的理解有没有更清晰一些？如果有具体的代码或场景不理解，我们可以深入讨论！
# 进程到线程
进程上下文:
![[Pasted image 20250402202825.png]]
线程上下文把栈取出来:
![[Pasted image 20250402202918.png]]  