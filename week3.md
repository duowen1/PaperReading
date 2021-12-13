# Довер´яй, но провер´яй SFI safety for native-compiled Wasm

本文主要解决的问题是，Wasm提供的代码安全隔离性依赖于编译器插入的安全检查。但是如果编译器是恶意的或者存在一些bug则可能对Wasm的隔离性造成破坏。而对编译器进行形式化验证会带来巨大的工作量。

为了解决这个问题，作者提供了一个验证器，其验证经过编译产生的二进制程序，确保其安全性。而验证器本身经过了形式化验证。

验证器的主要工作如下

![image-20211213141953628](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20211213141953628.png)

1. 首先对Wasm二进制程序进行反汇编，绘制控制流图
2. 解析控制流图中的所有间接跳转，保证所有的直接和间接跳转目标函数都在符号表中
3. VeriWasm检查反汇编代码，和插入的检查代码，并且拒绝二进制的不安全指令：例如int和syscall
4. VeriWasm分析每个函数以验证本地的安全属性：
   1. 线性的内存隔离
   2. 栈的隔离和完整性保护
   3. 全局变量隔离
   4. 控制流安全性



# CHANCEL Efficient Multi-client Isolation Under Adervarial Programs

本文的攻击模型仍然是在SGX的UAM（Untrusted Application Model）下，现有的工作没有解决同一enclave中线程隔离问题。本文想要设计实现multi-Client SFI机制，每个线程可以拥有自己独立的数据，数据保存在线程的上下文中。而线程间的共享数据是只读的，线程可以读取共享内存中的数据但是不能改变他。

对于和外部进行交互的数据，使用一个enclave内共享的秘钥对其进行加密。

CHANCEL提供了动态内存分配、in-enclave文件系统、客户端外部通信加密和隐蔽信道攻击的防御。



![image-20211210102235954](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20211210102235954.png)