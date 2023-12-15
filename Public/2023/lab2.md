[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-24ddc0f5d75046c5622901739e7c5dd533143b0c8e959d652212380cedb1ea36.svg)](https://classroom.github.com/a/W4MZkKgl)
# Lab2-RTP

## 测试
- 隐藏测试点：描述已在文档中给出。可以在test.cpp中增加测试case
- 问题：限时45s。有没有可能运气非常差产生过多的重传，而导致超时？
- 本地测试：有可能出现bind问题，可以多利用GitHub测试。

## 核心思路

1. 握手 & 挥手
    - Trivial：由于限定了超时次数，因此可以保证调用深度有限。使用递归实现状态机，转移方式最为直观。
    - WEAPON X: 使用无限循环+计时器，最为合理
2. 超时机制设计
    - lazy mode: 直接按超时值等待，等待结束后尝试收包
    - buzy mode: 启用计时器，同时不断循环尝试收包，直到超时跳出
3. 封装抽象
    - MSG_triple_t: 集成了fd, sockaddr, seq_num，这是RTP高度复用的数据结构，描述了消息的必要信息。在执行过程中会改变seq_num（Receiver最初还会获取addr），请在使用时传入指针。
    - MSG_Send & MSG_Recv: 短消息，直接发送/接收
4. 解耦合
    - 所有涉及网络和文件的函数都集成到basic库
    - .h头文件用于定义（链接），.c源文件用于实现（编译）
5. UDP
    - 无连接：每次收发都需要fd（端口）和addr（对端）两个参数
    - 如果Recvfrom不需要记录地址，addr可以填NULL，但len依然要正确，否则会出错
    - Receiver: 不知道Sender来自哪里，第一次需要记录addr
    - Sender: 发送后将会绑定端口
6. 状态机：维护状态
    - GBN
        - Sender: lb & ub
        - Receiver: seq_num
    - OPT/SACK
        - Sender: lb & ub & flags
        - Receiver: seq_num & flags
7. 细节
    - 检查流程：总是先检查length是否合理。有可能一个损坏的包拥有错误的length，作checksum检查将导致访存越界（段错误）
    - 幂等性：计算checksum过程会破坏checksum，因此需要先存下来，计算后再写回
    - 缓冲区：使用malloc分配在堆上，以应对HUGE_WINDOW。直接在栈上分配会爆内存。
    - 释放资源：文件结束fclose，套接字结束close，缓冲区结束free

## 文件说明
- test_party
    - 测试框架
- test
    - 测试模块
- src
    - util: 常用基础函数
    - rtp: rtp协议定义
    - myRTP_basic: 收发的可复用部分，逐层封装成简单易用的调用
    - sender: 发送端
    - receiver: 接收端
    - test: 测试函数
- build
    - 编译结果
- garbage
    - 弃用代码

## 函数表
```c
    int Socket_UDP();
    int fd_control(int sockfd);

    struct sockaddr_in Sender_Init_Addr(const char* dst_ip, int dst_port);
    struct sockaddr_in Receiver_Init_Addr(int lst_port);
    MSG_triple_t MakeMSGTriple(int fd, struct sockaddr_in addr, uint32_t seq_num);

    void Random_Init();
    int Random_Generate();

    void Checksum_Calculate(rtp_packet_t* pkt);
    int Checksum_Check(rtp_packet_t* pkt);

    FILE* Fopen_R(const char* path);
    FILE* Fopen_W(const char* path);
    int File_len(const char* path);

    void wait(int ms);
    double chrono(struct timespec begin);

    void PrintPacket(rtp_packet_t* pkt);
    void MakePacket(rtp_packet_t* pkt, uint32_t seq_num, uint16_t length, uint8_t flags);
    int VerifyPacket(rtp_packet_t* pkt, uint32_t seq_num, uint16_t length, uint8_t flags);

    int MSG_Send(MSG_triple_t* t, int flag);
    int MSG_Recv(MSG_triple_t* t, int flag);

    rtp_packet_t** Buffer_Init(int window_size);
    void Buffer_Finalize(rtp_packet_t** pointer);

    MSG_triple_t Sender_Init(const char* dst_ip, int dst_port);
    MSG_triple_t Receiver_Init(int lst_port);


    int Sender_1st_Shake(MSG_triple_t* t);
    int Sender_2nd_Shake(MSG_triple_t* t);
    int Sender_3rd_Shake(MSG_triple_t* t);
    int Sender_Shadow_Shake(MSG_triple_t* t);

    int Sender_1st_Wave(MSG_triple_t* t);
    int Sender_2nd_Wave(MSG_triple_t* t);

    int Receiver_1st_Shake(MSG_triple_t* t);        // set addr & seq_num
    int Receiver_2nd_Shake(MSG_triple_t* t);
    int Receiver_3rd_Shake(MSG_triple_t* t);

    int Receiver_1st_Wave(MSG_triple_t* t);
    int Receiver_2nd_Wave(MSG_triple_t* t);


    int Sender_Shake(MSG_triple_t* t, int stage, int times);
    int Sender_Wave(MSG_triple_t* t, int stage, int times);


    int Receiver_Shake(MSG_triple_t* t, int stage, int times);
    int Receiver_Wave(MSG_triple_t* t, int stage, int times);

    int Sender_Shake_X(MSG_triple_t* t);
    int Sender_Wave_X(MSG_triple_t* t);

    int Receiver_Shake_X(MSG_triple_t* t);
    int Receiver_Wave_X(MSG_triple_t* t);

    int Sender_PushPull_GBN(MSG_triple_t* t, const char* path, int window_size);s

    int Receiver_PushPull_GBN(MSG_triple_t* t, const char* path, int window_size);

    int Sender_PushPull_OPT(MSG_triple_t* t, const char* path, int window_size);

    int Receiver_PushPull_OPT(MSG_triple_t* t, const char* path, int window_size);
```