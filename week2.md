# TeeRex: Discovery and Exploitation of Memory  Corruption Vulnerabilities in SGX Enclaves



这篇文章是攻击类文章，主要解决的问题是挖掘SGX内的内存崩溃漏洞。由于SGX特殊的威胁模型，SGX面临着巨大的攻击面，包络：应用程序对enclave输入的数据、系统调用、空指针解引用等威胁。作者实现了一套基于符号执行的漏洞挖掘方法，并且利用了经典的符号执行框架ANGR。而将ANGR应用到enclave中面临着多种挑战：

1. ANGR 不能进入到enclave中
2. 在调用ECALL之前没有准备好的环境
3. enclave利用了ANGR不支持的CPU指令
4. ANGR 不支持多线程
5. 常见的受信的内存分配函数不被ANGR支持



# Formally Verified Memory Protection for a Commodity Multiprocessor Hypervisor

本文试图解决的问题是虚拟机的TCB过大，通过分层的设计，使得Hypervisor的TCB规模达到可以进行形式化验证的级别。







# Automatic Policy Generation for Inter-Service Access Control of Microservice



这篇文章主要解决的问题是如何自动化地生成细粒度、准确的大规模微服务之间的访问控制策略。现有的方案对该问题的解决存在以下问题：

1. 灵活性差、可拓展性差
2. 需要大量的历史数据

对于此问题的难点在于：

1. 如何获取完整的、细粒度的调用逻辑
2. 如何产生和更新访问控制策略

而本文的主要贡献在于：

1. 实现了一个基于请求提取的静态分析方法

2. 实现了一个基于图的策略管理机制

本文仍旧存在着很多弊端：

1. 由于请求提取使用了静态分析的方法，所以可能出现提取不充分
2. 无法处理闭源的服务
3. 假设基于微服务代码都是可信的
4. 特殊情况的处理

