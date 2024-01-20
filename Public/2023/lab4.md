[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-24ddc0f5d75046c5622901739e7c5dd533143b0c8e959d652212380cedb1ea36.svg)](https://classroom.github.com/a/mUzf3GgF)

# Lab4-Switch

## 设计
函数可以分成两类：
- 无状态的
    - 基础函数、成帧部分是无状态的，无需写到类的方法中。因此无需在头文件的类中声明，直接在源文件中实现即可。
- 有状态的
    - MAC表和处理内核是交换机相关的，因此需要在实现在类的方法中，并且在头文件的类定义中声明。

## 功能
1. 基础函数
    > 复用函数的抽象
    ```cc
    int CompByte(char byte1, uint8_t byte2)
    uint64_t GetMac(mac_addr_t mac_addr);
    void PrintFrame(char* framePtr);
    ```
2. 成帧
    > 这里比较特殊，一般的成帧是事先不知道长度的，依据定界符来划分，这里下层已经预处理好了划分，知道了帧的长度，只需调整定界符即可。
    ```cc
    int PackFrame(char* unpacked_frame, char* packed_frame, int frame_length);
    int UnpackFrame(char* unpacked_frame, char* packed_frame, int frame_length);
    ```
    
3. MAC表
    > 第一版方案中，MAC表使用线性表存储，实现简单。问题在于：每次查找一个表项需要全量搜索，并且不支持动态扩展。好在域内个体不多，搜索代价不大.
    ```cc
    int Switch::LookupMac(uint64_t mac);
    void Switch::UpdateMac(int index);
    void Switch::InsertMac(uint64_t mac, int port);
    void Switch::AgingMac();
    ```
    > 在下一个版本中，使用map实现。这样查找速度快，并且支持动态扩展。
    ```cc
    int FastSwitch::LookupMac(uint64_t mac);
    void FastSwitch::UpdateMac(uint64_t mac);
    void FastSwitch::InsertMac(uint64_t mac, int port);
    void FastSwitch::AgingMac();
    ```
4. 处理内核
    ```cc
    int HandleData(int inPort, char* framePtr);
    int HandleControl(int inPort, char* framePtr);

    void InitSwitch(int numPorts) override;
    int ProcessFrame(int inPort, char* framePtr) override;
    ```
## 细节
1. 大小端
    - 主机和网络都是小端法，无需转换。
2. MAC
    - 需要依据MAC地址查询表项信息。由于MAC地址是6字节的结构，因此可以用一个长整型来存放
3. 基本类型比较
    - 宏定义的一个字节值隐式地包含了无符号的类型uint8_t，char则是有符号的字节。
    - 二者虽然长度相同，但比较运算符并不会直接比较，而是会将它们扩展到int下比较。
    - 因此会造成二进制表示相同而比较运算不同的情况，比如有符号可以扩展成负数，而无符号数则总是扩展为正数。
    - 需要通过显式的强制转换来解决。

## Copilot
> 狂按Tab就完事了！
> 
> 函数功能变复杂导致参数增多，使用STL封装调用导致括号嵌套的情况下：亲历亲为消耗过大精力，Copilot是一个强力的填充助手。
> 
> 而实现交换机逻辑过程中，Copilot可能不一定能理解设计思路。