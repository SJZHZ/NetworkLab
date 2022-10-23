# lab1-myFTP
by 1900017702 ZY
## 说明
详细信息请参见 [lab1 documents](https://edu.n2sys.cn/#/tut_lab/lv1/README)

## 代码
### myFTP_basic.h
> 1. 宏定义及库载入
> 2. 数据结构及其构造和解析
> 3. 基础send/recv函数（循环）
> 4. 短报文send/recv函数（Header&Payload）
> 5. 长文件send/recv函数（循环）
> 6. 提取Server/Client API（不需要全局性）
>       > 监听套接字<br>
>       > 连接套接字<br>
>       > 其他封装函数

### ftp_client.c
> 1. 循环解析输入命令行
> 2. 路由分发到功能函数
> 3. 实现功能函数
> 4. 提取Client专用API
>       > Client短报文send-recv
### ftp_server.c
> 1. 建立监听并循环连接
> 2. 服务EPOLL事件
> 3. 路由读取报头并分发到功能函数
> 4. 实现功能函数
> 5. 提取Server专用API
>       > Server短报文send
### CMakeLists.txt
> 对.c源文件加上.h头文件链接：<br>
    在add_executable后面增加一项.h文件
```CMake
add_executable(ftp_server ftp_server.c myFTP_basic.h)
add_executable(ftp_client ftp_client.c myFTP_basic.h)
```

## 操作
1. 获取本地测试方法（在test_local子模块中）
```txt
根目录下提供了两个样例程序的可执行文件
    ftp_server_std
    ftp_client_std
./build文件夹中提供了自动测试程序的源代码（输入make编译出可执行文件）
    ftp_test
上述三个文件即本地测试的必要环境，复制出来即可
```
2. 编译源代码
```txt
根目录下存放要用的代码
    ftp_server.c
    ftp_client.c
    CMakeLists.txt（指定编译和链接方式，比如添加头文件）
    myFTP_basic.h（自行添加）
./build文件夹提供了自动编译源代码生成可执行文件的方式（输入make编译出可执行文件）
    ftp_server
    ftp_client
```
3. 执行自动测试
```txt
把上述五个可执行文件放在同一目录下（直接把前述3个复制到./build文件夹中）
    ftp_server_std
    ftp_client_std
    ftp_test
    ftp_server
    ftp_client
输入./ftp_test运行自动测试程序，在标准输出中得到测试结果（注意权限问题，chmod）
```

## 备注
1. Server未完成多线程功能，Pthread好难😭
2. 假设状态机在每个状态只会收到指定命令（而不是所有命令），在实际应用中不够周密
3. 最初测试程序存放路径不对，本地怎么测都没分。原谅我依靠疯狂push来测试🥺
4. send/recv和read/write要封装成循环形式，以便完全读取
    > 还未完成时会阻塞，连接失败则会直接返回。
5. 注意大/小端法转换（ntohX和htonX效果一样）
    > 报头是整12bytes传输的，其中的int是大端法的（字符串不是）<br>
    > 报文是流式（按字节）传输的，没有大小端法的区别
6. EPOLL可以消除阻塞
    > Linux默认采用水平触发（依状态），而非边缘触发（依变化）<br>
    > EPOLLIN：只要缓冲区还未读取完，就会一直触发
7. 重复功能先复制粘贴以便快速开发，等测试通过之后再提取为统一API以便简化结构<br>
   （也可以先设计好统一API，再直接调用）
8. 运行样例程序作为对端进行手动测试，方便单侧开发。两侧分别开发完成后，再手动测试对齐
9. 使用argc作为main函数的条件分支，可以兼容自动测试（有参数）和手动测试（无参数）