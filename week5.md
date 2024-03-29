# SHARD: Fine-Grained Kernel Specialization with Context-Aware Hardening



## 问题

- 内核膨胀，引入攻击面；
- 但是对于特定的应用或workload，只有一小部分内核代码被需要；
- 引入了Kernel specialization方法
- 目前的方法缺点：【见背景节】
  - 粒度太粗
  - 缺少严格的强制措施



前人的成果：

- ***FACECHANGE: application-driven dynamic kernel view switching in a virtual machine***
- ***Multik: A framework for orchestrating multiple specialized kernels***

可以将内核代码减少至8.89%



## 方法

1. 识别需要执行一个系统调用的内核代码；
2. 在运行时保证应用在调用这个相同的系统调用时其他的内核代码不会被执行；
3. 识别其他应当被允许的system call
4. 实现了 **上下文敏感** 的hardening，利用了细粒度的控制流完整性
5. offline分析，决定内核覆盖率；online阶段，利用硬件辅助虚拟化实现透明地切换。



## 解决思路

为应用每一个正在执行的系统调用提供一个不同的“kernel-view”。所以是应用角度+系统调用角度：



给代码分类：

1. 可到达的函数system call handler
2. 潜在的可到达的函数
3. 不可到达的函数【静态分析可以确定哪些函数是一定无法到达的】



在1->2转换时，加固内核。这种技巧通过CFI完整性做到。



### 架构

![image-20220309192047427](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20220309192047427.png)



#### offline阶段

1. 构建内核的控制流图，识别每个系统调用的unreachable代码；

2. 动态执行，获取reachable代码，剩余部分即为potential reachable代码；

3. 生成应用的配置文件。

   

##### 如何产生CFG

- 两层算法产生CFG（这里作者参考了两篇文章）【Detecting missing-check bugs via semantic-and context-aware criticalness and constraints inferences、Check it again: Detecting lacking-recheck bugs in os kernels】
  - 内核中频繁使用struct和函数指针；



##### 识别reachable代码

- 通过轻量级虚拟化的方式识别【利用UD2指令，UD2指令会产生一个#UD异常】
  - 记录应用调用的系统调用
  - 执行的内核函数



##### Offline Kernel Instrumentation

- 每个内核代码页分为了三个版本：
  - UNRESTRICTED：启用所有内核函数，用于信任的应用；
  - RESTRICTED：启用每个系统调用的reachable代码
  - HARDENED：包含reachable和potential reachable代码，用于不信任的应用。

- SHARD保证函数不会横跨三个页



 

#### online阶段

VMX security monitor：理解上是一个轻量级的hypervisor，其中的功能如下：

1. 追踪不受信任的应用的上下文切换和系统调用
2. 对应用的每个系统调用实现专门化的kernel-view
3. 在执行potential reachable代码时，实现控制流完整性



SHARD透明地替换内核代码页，如果检测到执行到potential reachable代码，会将内核代码页替换成hardened版本。



##### 无限制的代码页

为了能够进行上下文的切换，也填充了NOP指令。



##### 限制的代码页

只包含可到达的内核函数，其他的函数都被UD2指令所替换。



##### 加固的代码页

potentially reachable和reachable代码，实现了CFI，保证所有控制流跳转都是在CFG中。这里基本上使用前人的实现方式



### 运行时监控

1. 初始化kernel-view：启用UNRESTRICTED 版内核代码页；
2. debloating enforcement：上下文切换到untrusted应用或者untrusted应用进行系统调用时；
3. hardening enforcement：在RESTRICTED页中，执行unreachable或者potentially reachable会触发#UD。如果是访问unreachable区域，那么会kill应用，如果是访问potentially，那么切换到hardened kernel view，开启CFI检查，使用CPU last branch record。
4. 禁用hardening：monitor禁用hardening

![image-20220309213625546](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20220309213625546.png)



**如何提高效率**：在hypervisor层，做页目录表级的切换（page directory-level）



**基于LBR的控制流完整性检查**：主要为了保护这一次的控制流跳转



## 安全性评估

作者选择的实验对象是：

1. NGINX
2. Redis

有对应的实验套件：

1. ab
2. redis-benchmark



评估的内容：

1. 对应系统调用所需要的指令数；
2. ROP/JOP的gadget数
3. 攻击防御以及分析



## 性能评估

- Memory overhead
- 执行时间
- 动态执行的准确性





# Donky: Domain Keys - Efficient In-Process Isolation for RISC-V and x86



## 问题

现有的进程内隔离方案：

1. 控制流机制；
2. 基于权限的设计；
3. Protection key机制



### Intel MPK



#### 机制

- 不需要内核的参与以修改页表的数据。
- 4-bit也保存在PTE（page-table entry）
- 对应Keys的权限保存在PKRU寄存器中

**优点：**

- 效率很高；

**缺点**

- 可以访问到MPK的控制寄存器；

**使用中如何克服缺点：**

- 二进制扫描；
- 不可写的代码页；



## 解决



![image-20220315145939521](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20220315145939521.png)

###  软件

- 每个domain会有自己的protection key的组合
- 访问控制有Policy寄存器控制
- domain monitor管理protection key和policy寄存器以及系统调用过滤



#### Donky Monitor

- Without involvement of the kernel
- 只有在monitor中可以修改保护策略和策略寄存器



#### 软件抽象层

- Donky API
  - 管理domain、protection key和其相关联的内存、和其他domain共享keys
  - 跨域调用（dcalls）

![image-20220315154051197](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20220315154051197.png)

- did：domain id
- Donky API遵循secure-by-default原则



![image-20220315154651300](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20220315154651300.png)





- 从一个root domain启动，可以建立子domains（第8行）
- 请求新的protection key（第4行）
- 将protection keys和其他domain相关联（第9行）
- 环境切换函数需要注册以及赋予权限（第11-12行）
- drop 子domain的权限



#### Domain转换

![image-20220315155402755](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20220315155402755.png)

- 这个dcall就很像是Intel SGX
- wrapper在调用和目标domain中都存在，保存参数，清理寄存器



# DECAF: Automatic, Adaptive De-bloating and Hardening of COTS Firmware



## 问题

一个经典的问题，降低固件中的攻击面。

- 代码量大
- 代码比较滞后
- 被利用后会危害极大



### UEFI

- UEFITool
- 目标即为替代BIOS
- 启动分为4个阶段


#### Security

- SEC阶段是系统的信任根
- 进行最早的硬件初始化、验证firmware的镜像
- 将控制区交给PEI

#### PEI

- Pre-EFI Initialization Environment
- PEI阶段会终止硬件的初始化
- 枚举平台信息到一系列的Hand Off Blocks (HOBs)，然后传递给DXE阶段
- 依赖于处理器架构，因为其只初始化用到资源直到内存配置完成
- 代码尽可能简单

#### DXE

- Driver Execution Environment
- 被设计为用户空间的UEFI环境
- 驱动的接口会被安装到初始化了硬件上
- 其负责以正确的顺序发现、加载和执行驱动
- 将控制流转移给BDS，OS的boot loader会接管控制流

#### BDS

![[Pasted image 20220317200646.png]]

## Background

#### Firmware的布局

UEFI的固件由一组flash描述符区域组成，这些描述符表标识了镜像的其他区域，比如ME或者网络接口。BIOS区域被拆分为firmware volumes，每个包含一组模块，每个模块包含一个或多个sections。部分模块会包含PE32二进制区域，运行时是会执行。

![[Pasted image 20220317201232.png]]

（大概的层次结构：Region -> Volume -> Module -> Section）

对于可执行的 UEFI 模块，其中一个部分将包含一个 PE32 二进制映像。 这是由固件调度的独立可执行文件。 可执行模块还将包含一个依赖项 (DEPEX) 部分，它将确定执行模块的顺序。 在执行期间，模块将安装指向使用 UEFI 系统函数的函数的指针。 安装的功能称为协议，由全局唯一标识符 (GUID) 标识。 其他模块使用这些 GUID 来查找已安装的协议并调用它们。 这就是独立模块相互链接的方式。 

每个模块有一个 DEPEX section，告诉了DXE dispatcher哪些模块和protocols需要在执行前初始化。如果DEPEX表达式为真，模块加载，否则推迟。

作者认为通过DEPEX section的方式获取协议的依赖关系并不准确，最终选择了使用动态监控的方式实现。



## 解决



![image-20220321104737667](C:\Users\xsw\AppData\Roaming\Typora\typora-user-images\image-20220321104737667.png)



软硬结合的设计：



- 【A】 Luigi：分布式管理监控
- 【B】 UEFITool：静态分析、编辑UEFI镜像



大概的思路是在保证启动顺序的不变的情况下，去修改不同代码的实现。代码运行的结果是在内存中，然后把内存dump出来，分析内存内容。
