---
title: 测试驱动开发TDD
date: 2020-06-15 14:27:12
tags:
---
## 测试驱动开发TDD（Test Driven Development）
### 1、什么是 TDD
#### 1、TDD 有广义和狭义之分
1、常说的是狭义的TDD，也就是单元测试驱动开发UTDD（Unit Test Driven Development）；

2、广义的TDD：是验收测试驱动开发ATDD（Acceptance Test Driven Development），
包括行为驱动开发BDD（Behavior Driven Development）和消费者驱动契约开发Consumer-Driven Contracts Development 等。

#### 2、TDD 有三层含义：
 - Test-Driven Development，测试驱动开发。
 - Task-Driven Development，任务驱动开发，要对问题进行分析并进行任务分解。
 - Test-Driven Design，测试保护下的设计改善。TDD 并不能直接提高设计能力，它只是给你更多机会和保障去改善设计。

#### 3、TDD 流程
TDD 的基本流程是：红，绿，重构。
![tdd2](./tdd2.jpg)

更详细的流程是：
![tdd1](./tdd1.jpg)

1、编写测试
2、运行测试，观察测试结果是否如期失败（变红）
3、测试结果不如期失败，返回第1步修改测试
4、测试结果如期失败，编写刚好能够让测试通过的产品代码实现
5、运行测试，观察测试结果是否如期成功（变绿）
6、测试结果不如期成功，返回第4步修改实现
7、测试结果如期成功，分析代码是否需要重构
8、需要重构，返回第4步修改实现
9、不需要重构，编写下一个测试
 
#### 4、TDD 优点
1、降低开发者负担
通过明确的流程，让我们一次只关注一个点，思维负担更小。

2、保护网
TDD 的好处是覆盖完全的单元测试，对产品代码提供了一个保护网，让我们可以轻松地迎接需求变化或改善代码的设计。
所以如果你的项目需求稳定，一次性做完，后续没有任何改动的话，能享受到 TDD 的好处就比较少了。

3、提前澄清需求
先写测试可以帮助我们去思考需求，并提前澄清需求细节，而不是代码写到一半才发现不明确的需求。

4、快速反馈
有很多人说 TDD 时，我的代码量增加了，所以开发效率降低了。但是，如果没有单元测试，你就要手工测试，你要花很多时间去准备数据，启动应用，跳转界面等，反馈是很慢的。准确说，快速反馈是单元测试的好处。

#### 5、TDD 的三条规则
1、除非是为了使一个失败的 unit test 通过，否则不允许编写任何产品代码

2、在一个单元测试中，只允许编写刚好能够导致失败的内容（编译错误也算失败）

3、只允许编写刚好能够使一个失败的 unit test 通过的产品代码

如果违反了会怎么样呢？
1、违反第一条，先编写了产品代码，那这段代码是为了实现什么需求呢？怎么确保它真的实现了呢？
2、违反第二条，写了多个失败的测试，如果测试长时间不能通过，会增加开发者的压力，另外，测试可能会被重构，这时会增加测试的修改成本。
3、违反第三条，产品代码实现了超出当前测试的功能，那么这部分代码就没有测试的保护，不知道是否正确，需要手工测试。可能这是不存在的需求，那就凭空增加了代码的复杂性。如果是存在的需求，那后面的测试写出来就会直接通过，破坏了 TDD 的节奏感。

我认为它的本质是：
1、分离关注点，一次只戴一顶帽子
在我们编程的过程中，有几个关注点：需求，实现，设计。
TDD 给了我们明确的三个步骤，每个步骤关注一个方面。

 - 红：
写一个失败的测试，它是对一个小需求的描述，只需要关心输入输出，这个时候根本不用关心如何实现。
 - 绿：
专注在用最快的方式实现当前这个小需求，不用关心其他需求，也不要管代码的质量多么惨不忍睹。
 - 重构：
 既不用思考需求，也没有实现的压力，只需要找出代码中的坏味道，并用一个手法消除它，让代码变成整洁的代码。

2、注意力控制
人的注意力既可以主动控制，也会被被动吸引。注意力来回切换的话，就会消耗更多精力，思考也会不那么完整。
使用 TDD 开发，我们要主动去控制注意力，写测试的时候，发现一个类没有定义，IDE 提示编译错误，这时候你如果去创建这个类，你的注意力就不在需求上了，已经切换到了实现上，我们应该专注地写完这个测试，思考它是否表达了需求，确定无误后再开始去消除编译错误。

参考：https://www.jianshu.com/p/62f16cd4fef3