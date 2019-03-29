title: 操作系统-用户接口与作业调度
---
1、程 序 的 启 动 和 结 束
1.1．程序的启动 
– 程序开始执行时必须满足两个前提条件： 
• 程序已装入内存 
• 程序计数器PC中已置入该程序在内存的入口地址 

1.2 五种启动程序执行的方式
1）命令方式 
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi1.png?raw=true)

2）批处理方式 
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi2.png?raw=true)

3）EXEC方式 
– 在一个程序中运行另一个程序 
– 返回原来的程序 
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi3.png?raw=true)

4）由硬件装入程序和启动程序执行

5）自启程序 
• 自己装入自己，并启动自己开始执行的程序 
• 自启程序由两部分组成：引导程序和程序主体 
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi4.png?raw=true)

2、程序的结束 
正常结束：程序按自身的逻辑有效地完成预定功能
后结束 
（a）返回父程序并回送结果信息。 
（b）释放所用资源（空间、设备），记录使用情
况，记帐等 
– 异常结束：发生了某些错误而导致程序在没有完成
预定功能时提前结束 

3、用户与操作系统的接口
为用户提供两种接口：
（1）命令接口 
用户通过这些命令来组织和控制作业的执行。 
（2）程序接口 
编程人员使用他们来请求操作系统服务。

作业控制的主要方式：
（1）联机命令接口 
又称交互式命令接口，由一组键盘操作命令组
成。用户通过控制台或者终端键入操作命令，完
成对作业的控制。 
（2）脱机命令接口 
又称批处理命令接口，由一组作业控制语言组
成，由系统负责解释执行。（涉及到作业的相关
概念） 

3、作 业 的 基 本 概 念
（1）作业 
用户在一次计算过程中，或者一次事务处理过程中，
要求计算机系统所做工作的总称 
（2）作业步 
 一个作业可划分成若干部分，称为一个作业步 
（3）典型的作业控制过程： 
 “编译”、“连接装配”、“运行” 

 作业组织：
 作业由三部分组成，即程序、数据、作业说明书。

 作 业 控 制 块（JCB：Job Control Block） 
– 作业控制块是作业存在的标志 
– 保存现有系统对于作业进行管理所需要的全部信息 
– 位于磁盘区域中 

![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi5.png?raw=true)

作 业 的 状 态 及 转 换：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi6.png?raw=true)

4、作 业 调 度
调度的实质
• 资源的分配 
– 调度算法定义
• 根据系统的资源分配策略所规定的资源分配算法

调 度 算 法 的 性 能 准 则：
– （平均）周转时间：作业从提交到完成的时间（用户角度）
周转时间：Ti=Tc-Ts 
Tc作业完成时刻；
Ts作业进入系统时刻

平均周转时间：（T1+T2+...+Tn)/n

– （平均）带权周转时间：周转时间 / CPU运行时间（用户角度）
带权周转时间:
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi7.png?raw=true)

平均带权周转时间：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi8.png?raw=true)

– 响应时间：用户输入一个请求（如击键）到系统给出首次响应（如屏幕
显示）的时间（处理机的角度）
– 公平性：不因作业或进程本身的特性而使上述指标过分恶化（算法本身的角度）
– 优先级：可以使关键任务达到更好的指标（算法本身的角度）

5、先来先服务算法FCFS （First Come First Served）
– 按作业的先后顺序进行调度
– 处理过程
1）按照作业提交先后次序，分配CPU执行；
2）当前作业占用CPU，直到执行完或阻塞（如申请I/O）让出CPU；
3）作业被唤醒后（如I/O完成），不立即恢复执行，等待当前作业 
 让出CPU后才可以恢复执行。
– 最简单的调度算法 
– 对短作业不利（平均周转时间延长）

举例：假设在单道批处理环境下有四个作业，已知它们进入系统的时间、估计运行时间 ，求：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi9.png?raw=true)

6、短 作 业 优 先 算 法 SJF （Shortest Job First ）
按作业的长短顺序进行调度，短作业优先
• 对预计执行时间短的作业优先分配CPU
• 通常后来的短作业不抢占正在执行的作业
– 对FCFS算法的改进，目标是减少平均周转时间

– 优点： 
• 相比于FCFS改善平均周转时间和平均带权周转时间；
• 缩短作业的等待时间；
• 提高系统的吞吐量；
– 缺点： 
• 对长作业非常不利，可能长时间得不到执行；
• 难以准确估计作业（进程）的执行时间，从而影响调度性能。
• 未能依据作业的紧迫程度来划分执行的优先级；

举例：
假设在单道批处理环境下有四个作业，已知它们进入系统的时间、估计运行时间 ，求：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi10.png?raw=true)

7、最 短 剩 余 时 间 优 先 算 法 SRT （Shortest Remaining Time）
– 短作业优先算法的变型，也称作抢占式的短作业优先算法 
– 允许比当前进程剩余时间更短的进程来抢占 
– 抢占时机：新作业加入队列时 

8、最 高 响 应 比 优 先 算 法HRRN （Highest Response Ratio Next）
– 从就绪队列中选出响应比最高的作业投入执行
– 响应比R = (等待时间W +要求执行时间T) / 要求执行时间T
– FCFS和SJF的折衷
 
– 优点：既照顾了短作业，也考虑到先后顺序 
– 缺点：每次调度时要调用响应比计算，增加了系统开销 

举例：假设在单道批处理环境下有四个作业，已知它们进入系统的时间、估计运行时间 ，求：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi11.png?raw=true)

9、基 于 优 先 数 调 度 算 法 HPF （ Highest Priority First ）
（a）由用户规定优先数（外部优先数）
 用户提交作业时，根据急迫程度规定适当的优先数
 作业调度程序根据JCB优先数决定进入内存的次序
（b）由系统计算优先数（内部优先数） 

举例：
在两道环境下有四个作业，已知它们进入系统的时间、估计运行时间； 作业调度采用短作业优先调度算法（作业被调度
运行后直到结束前不再退出内存）； 进程调度采用最短剩余时间优先调度算法（当新作业投入运行后，可按照作业剩余运行时间长短调整次
序，可抢占CPU）； 请给出这四个作业的执行时间序列，并计算出平均周转时间及平均带权周转时间；
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/osapi12.png?raw=true)

10、系 统 调 用
系统调用，是用户在程序中调用操作系统所提供的一些子功能（程序接口）。
这个指令还将系统转入管态
系统调用是操作系统提供给编程人员的唯一接口 