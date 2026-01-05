# xuexi_test
//测试上传

创建工程遇到的难题

第一个问题是：没有正确的进行文件的创建

出现了这个情况，导致我以为自己的电脑没有装stm32 的芯片支持包。  

<img width="1026" height="756" alt="Image" src="https://github.com/user-attachments/assets/3e9b83e3-5d6b-4002-a9c3-ba5afb7affcb" />

关闭这个界面，然后重新创建一个工程，就不会出现这个问题。

第二个问题就是关于启动文件的选择：根据资料不同的芯片型号选择不同的启动文件。
STM32 标准库把 F1 系列分为 LD(低密度)、MD(中密度)、HD(高密度)、XL 等，而 F103ZE 这类 512KB Flash 的芯片归类为高密度 HD。


对应关系里给出的规则是：256K–512K Flash 选用 HD，因此 F103ZET6 应使用 startup_stm32f10x_hd.s 这个启动文件。

第三个问题就是创建启动文件后，编译出现这个错误(截取部分)：

```
Rebuild started: Project: led01
*** Using Compiler 'V6.19', folder: 'D:\arm_KeilMDK\ARM\ARMCLANG\Bin'
Rebuild target 'Target 1'
Start/core_cm3.c(445): error: non-ASM statement in naked function is not supported
uint32_t result=0;
^
Start/core_cm3.c(442): note: attribute is here
uint32_t __get_PSP(void) __attribute__( ( naked ) );
void
./Start/core_cm3.h(1217): error: unknown type name 'inline'
static __INLINE void __CLREX() { __ASM volatile ("clrex"); }
^
./Start/core_cm3.h(751): note: expanded from macro '__INLINE'
#define __INLINE inline /*!< inline keyword for GNU Compiler */
^
./Start/core_cm3.h(1217): warning: a function declaration without a prototype is deprecated in all versions of C [-Wstrict-prototypes]
static __INLINE void __CLREX() { __ASM volatile ("clrex"); }
^
void
./Start/core_cm3.h(1468): error: unknown type name 'inline'
static __INLINE void NVIC_SetPriorityGrouping(uint32_t PriorityGroup)
^
./Start/core_cm3.h(751): note: expanded from macro '__INLINE'
#define __INLINE inline /*!< inline keyword for GNU Compiler */
^
./Start/core_cm3.h(1489): error: unknown type name 'inline'
static __INLINE uint32_t NVIC_GetPriorityGrouping(void)
^
./Start/core_cm3.h(751): note: expanded from macro '__INLINE'
#define __INLINE inline /*!< inline keyword for GNU Compiler */
^
./Start/core_cm3.h(1489): error: expected ';' after top level declarator
static __INLINE uint32_t NVIC_GetPriorityGrouping(void)
^
;
./Start/core_cm3.h(1756): error: unknown type name 'inline'
static __INLINE uint32_t ITM_SendChar (uint32_t ch)
^
./Start/core_cm3.h(751): note: expanded from macro '__INLINE'
#define __INLINE inline /*!< inline keyword for GNU Compiler */
^
./Start/core_cm3.h(1756): error: expected ';' after top level declarator
static __INLINE uint32_t ITM_SendChar (uint32_t ch)
^
;
Start/system_stm32f10x.c(162): warning: no previous extern declaration for non-static variable 'SystemCoreClock' [-Wmissing-variable-declarations]
uint32_t SystemCoreClock = SYSCLK_FREQ_72MHz; /*!< System Clock Frequency (Core Clock) */
^
Start/system_stm32f10x.c(162): note: declare 'static' if the variable is not intended to be used outside of this translation unit
uint32_t SystemCoreClock = SYSCLK_FREQ_72MHz; /*!< System Clock Frequency (Core Clock) */
^
Start/system_stm32f10x.c(167): warning: no previous extern declaration for non-static variable 'AHBPrescTable' [-Wmissing-variable-declarations]
__I uint8_t AHBPrescTable[16] = {0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 3, 4, 6, 7, 8, 9};
^
Start/system_stm32f10x.c(167): note: declare 'static' if the variable is not intended to be used outside of this translation unit
__I uint8_t AHBPrescTable[16] = {0, 0, 0, 0, 0, 0, 0, 0, 1, 2, 3, 4, 6, 7, 8, 9};
^
17 warnings and 17 errors generated.
compiling system_stm32f10x.c...
assembling startup_stm32f10x_hd.s...
".\Objects\led01.axf" - 38 Error(s), 53 Warning(s).
Target not created.
Build Time Elapsed: 00:00:11
```

主要原因还是
你用的是 Arm Compiler 6（clang 内核），但 core_cm3.c / core_cm3.h 和 startup_stm32f10x_*.s 这些文件是给 旧版 AC5（armcc） 和老 CMSIS 写的，语法和关键字与 AC6 不兼容，所以出现 “naked function 不允许非汇编语句 / inline 未知类型” 之类错误。按照这个办法解决：打开 “魔法棒” → 选项里的 Target 或 C/C++ 选项卡，把 “Using Compiler V6.x” 改成 Use default Compiler Version 5 或直接选择 Arm Compiler 5。
<img width="626" height="470" alt="Image" src="https://github.com/user-attachments/assets/feac2d0b-60fd-48e4-8ce0-6ec66d2c6d4b" />

新建的这个工程文件夹中放的文件是：
1、User 文件夹：暂时只有一个文件主函数，程序入口，包含主要应用逻辑。以后会放一些业务层逻辑。

2、Library 文件夹：是我从官方提供的文件中拷贝的STM32标准外设库， STM32标准外设库包含所有外设驱动程序和头文件。如GPIO、RCC（时钟管理）、EXTI（外部中断）等。

3、Start 文件夹： 芯片初始化和启动代码也是从官方的文件中拷贝的。

4、DebugConfig 文件夹: Keil MDK调试器配置文件 ,配置STM32F103ZE芯片在调试模式下的行为，特别是当CPU被暂停/断点暂停时，各个外设和计时器如何运作.

5、Objects 文件夹: 编译生成文件（中间/输出文件),这是Keil MDK编译生成的中间文件和输出文件夹，包含编译、链接后的产物。(Objects文件夹可以完全删除，重新编译时会自动生成。这是临时编译文件，不需要版本控制。)

6、Listings 文件夹 - 编译列表文件。()





2026年1月3日2026年1月2号

浏览了野火stm32初级教程前三章的内容：

第一章 介绍配套开发板的使用，keil的一些相关的功能和j-link下载。

第二章 主要说的是jlink、keil的安装和芯片的安装包，以及创建一个新的工程和里面的相关文件是干啥的。

第三章 进行新工程的创建，我需要重点看一下keil的仿真调试，因为我很少调试过这些，并将自己写的模板工程点亮一个led小灯进行了一个调试，进去看了一些可以进去看到一些相关的函数。发现一个问题就是在函数跳转到初始化的时候，会卡死。原因是：自己对于仿真调试的三个按键不够了解，首先先了解这三个按键

Step (Step Into) ： 逐语句每一行都要进去执行，当你怀疑某个具体函数（比如自己写的 LED_On()）内部逻辑有问题，想进去一行一行检查细节时。

Step Over ：逐个函数执行，比如你调用了自己定义的某个函数，你确信这个库函数没问题，不想浪费时间进去看它怎么置位寄存器，就点这个按钮直接“跨”过去。

Step Out ：跳出某个函数，你不小心用“Step Into”进了一个很长的库函数（比如 printf 实现），发现里面有几百行代码，不想一行一行按了，就点这个按钮直接跳出来。

今天遇到的这个情况是：在 main 函数的主循环里点这个。因为 main 函数通常是死循环，永远不会“Return”，点这个等于全速运行，会导致调试器看起来“卡死”。



| 按钮     | 名称      | 执行动作       | 进入函数       | 相关的场景                           |
| -------- | --------- | -------------- | -------------- | ------------------------------------ |
| F11      | step into | 向下或进入内部 | 进入函数内部   | 底层细节以及自己写的函数             |
| F10      | step over | 向下           | 直接运行完函数 | 查看主逻辑流程以及跳过已知正确的函数 |
| Ctrl+F11 | step out  | 向外           | 立即做完并返回 | 进入一个长函数向退出                 |

最后写了一个流水灯，我才了解到标准库没有延时含糊需要用Systick系统滴答定时器来写。还未进行分层，以及了解gpio引脚的输入输出模式。



2026年1月3号：

1.介绍库函数的作用，是配置最底层的寄存器，有利于快速的开发，易于阅读，维护成本低。是与寄存器介于驱动层与用户之间的代码，向下处理寄存器直接相关的配置，向上为用户提供配置寄存器接口。

2.库函数开发方式与寄存器开发方式：

库函数开发方式：从驱动层====》库函数层，调用相关的库接口；库函数层===》特殊寄存器层，通过函数、宏封装配置寄存器的操作来做的。

直接配置寄存器的方式：驱动层====》特殊寄存器层，是直接配置寄存器来实现的。

3.cmsis标准：ARM 与芯片厂商建立了 CMSIS 标准(Cortex MicroController Software Interface  Standard)。

<img width="700" height="416" alt="Image" src="https://github.com/user-attachments/assets/35f130d1-387b-475a-ad59-b61258d972e0" />

<img width="756" height="626" alt="Image" src="https://github.com/user-attachments/assets/d938b3e8-b7c8-4f7a-a08d-875c9dd35c6c" />

内核层的函数：包括访问寄存器的名称、地址定义，主要是有ARM公司提供。

设备外设访问层：提供了片上的核外外设的地址和中断的定义，主要由芯片的厂商提供。

4.了解安装包中库文件夹中相关文件的作用：

- core_cm3.c 跟启动文件一样都是底层文件，都是由 ARM 公司提供的，遵 守 CMSIS 标准，即所有 CM3 芯片的库都带有这个文件，这样软件在不同的 CM3 芯片的移植工作就得以简化。
- 在 DeviceSupport 文件夹下的是启动文件、外设寄存器定义&中断向量 定义层 的一些文件，这是由 ST 公司提供的。system_stm32f10x.c 在实现系统时钟的时候要用到 PLL（锁相环），这 就需要操作寄存器，寄存器都是以存储器映射的方式来访问的，所以该文件中 包含了stm32f10x.h 这个头文件。
- stm32f10x.h这个文件，是一个底层文件。所有处理器厂商都会将对内存的操作封装成一个宏，即我们通常说的寄存 器，并且把这些实现封装成一个系统文件，包含在相应的开发环境中。

5.启动文件：Libraries\CMSIS\Core\CM3\startup\arm 文件夹下是由汇编编写的 系统启动文件，不同的文件对应不同的芯片型号，在使用时要注意。一些英文缩写的意义如下：

cl：互联型产品，stm32f105/107 系列 

vl：超值型产品，stm32f100 系列 

xl：超高密度（容量）产品，stm32f101/103 系列 

ld：低密度产品，FLASH 小于 64K 

md：中等密度产品，FLASH=64 or 128 

hd：高密度产品，FLASH 大于 128

6.重点：启动文件的作用：

- 初始化堆栈指针SP。
- 初始化程序计数器指针PC‘；
- 设置堆栈的大小；
- 设置异常向量表的入口地址。
- 设置外部SRAM作为数据存储器（这个有用户配置，一般的开发板可没有外部SRAM）。
- 设置C库的分支入口__main(最终用来调用main函数)。
- 在3.5版的启动文件还调用了在system_stm32f10x.c文件中的systemIni()函数配置系统时钟，在旧版本工程中要用户进入main函数自己调用systemIni()函数。

7.Libraries\STM32F10x_StdPeriph_Driver 文件夹下有 inc（include 的 缩写）跟 src（source 的简写）这两个文件夹，这都属于 CMSIS 的设备外设函 数部分。src 里面是每个设备外设的驱动程序，这些外设是芯片制造商在 Cortex-M3 核外加进去的。

8.还需要添加这三个文件：在库目录的\Project\STM32F10x_StdPeriph_Template 目录下，存放 了官方的一个库工程模板，我们在用库建立一个完整的工程时，还需要添加这 个目录下的 stm32f10x_it.c、stm32f10x_it.h、stm32f10x_conf.h 这三 个文件。下面是这三个文件的作用：

- stm32f10x_it.c，是专门用来编写中断服务函数的，在我们修改前，这个 文件已经定义了一些系统异常的接口，其它普通中断服务函数由我们自己添 加。
- stm32f10x_conf.h，这个文件被包含进 stm32f10x.h 文件。是用来配 置使用了什么外设的头文件，用这个头文件我们可以很方便地增加或删除上面 driver 目录下的外设驱动函数库。
- stm32f10x_conf.h 这个文件还可配置是否使用“断言”编译选项，在开 发时使用断言可由编译器检查库函数传入的参数是否正确，软件编写成功后， 去掉“断言”编译选项可使程序全速运行。可通过定义 USE_FULL_ASSERT 或取消定义来配置是否使用断言。

8.库函数的关系：
<img width="665" height="802" alt="Image" src="https://github.com/user-attachments/assets/97b38aad-bdee-4bad-ae52-c0f2e5812f05" />

2026年1月4日与1月3日

第五章：

1.存储器映射图：

<img width="751" height="574" alt="Image" src="https://github.com/user-attachments/assets/5f4c8252-ce8f-4e8b-b61f-85a9efae1b4f" />

寄存器的最终地址(绝对地址)=外设基地址+总线基地址+寄存器组基地址

外设基地址：
-   每个外设(比如GPIOA、USART、TIME)所属的总线占据一小段的连续寻址范围，这段范围起始地址叫做外设基地址 。
- 这个就是某个具体外设在内存里的起点，所有的外设寄存器都在这一区域内按偏移排列。
-  例如：首先看到 PERIPH_BASE 这个宏，宏展开为 0x4000 0000,并把它强制
转换为 uint32_t 的 32 位类型数据，这是因为地 STM32 的地址是 32 位的，Cortex-M3 核分配给片 上外设的从 0x4000 0000 至 0x5FFF FFFF 的 512MB 寻址空间中 的第一个地 址，我们把 0x4000 0000 称为外设基地址。


总线基地址：
- 在MCU中，片上外设挂载在不同总线上(比如AHB、APB1、APB2)。
- 每条总线在整个存储器映射中有一段连续的地址，这段区域的第一个地址叫“总线基地址”，也是这条总线上第一个外设的起始地址。
- 例如：接下来是宏 APB2PERIPH_BASE，宏展开为 PERIPH_BASE（外设基地
址）加上偏移地址 0x1 0000，即指向的地址为 0x4001 0000。STM32 不同的外设是挂载在不同的总
线上的，见图 5-8。有 AHB 总线、APB2 总线、APB1 总线，挂载在这些总线上
的外设有特定的地址范围。


寄存器组基地址：
- 针对“单个外设”（如GPIOA、USART1），这个外设控制/状态寄存器是一整组连续地址。这组寄存器最前面的那个地址，就是这个外设的寄存器组地址，也被称为“这个外设的基地址”。
- 最后到了宏 GPIOC_BASE，宏展开为 APB2PERIPH_BASE (APB2 总线
外设的基地址)加上相对 APB2 总线基地址的偏移量 0x1000 得到了 GPIOC 端
口的寄存器组的基地址。


2.C语言关键字volatile：
volatile：告诉编译器“这个变量随时可能变，每次读写都算有副作用”。

作用：禁止对它的读写做省略、合并等优化，保证每次都真正访问内存/寄存器。

典型用途：外设寄存器、中断/多线程共享标志、延时空循环。

不能做的事：不能保证原子性，不能解决所有并发问题。
3.stm32对于库函数的封装：
在 stm32f10x.h 文件中，有以下代码：

```c
代码一
#define GPIOA               ((GPIO_TypeDef *) GPIOA_BASE)
#define GPIOB               ((GPIO_TypeDef *) GPIOB_BASE)
#define GPIOC               ((GPIO_TypeDef *) GPIOC_BASE)
#define GPIOD               ((GPIO_TypeDef *) GPIOD_BASE)
#define GPIOE               ((GPIO_TypeDef *) GPIOE_BASE)
#define GPIOF               ((GPIO_TypeDef *) GPIOF_BASE)
```

有了这些宏，我们就可以定位到具体的寄存器地址，在这里发现了一个陌 生的类型 GPIO_TypeDef ，追踪它的定义，可以在 stm32f10x.h 文件中找 到如下代码：图1

<img width="721" height="359" alt="Image" src="https://github.com/user-attachments/assets/38b5538c-6d60-4ad8-83a1-aad5d340dc34" />

其中 __IO 也是一个 ST 库定义的宏，宏定义如下：图二
<img width="920" height="470" alt="Image" src="https://github.com/user-attachments/assets/f9340f53-94a7-4d36-939d-f57e8b08e431" />

只要匹配结构体 的首地址，就可以确定各寄存器的具体地址了。图三
<img width="617" height="621" alt="Image" src="https://github.com/user-attachments/assets/46eab5b8-9064-464e-ba38-63b4b12017de" />
分析代码一：GPIOA_BASE 在上一小节已解析，是一个代表 GPIOA 组寄存器的基地址。(GPIO_TypeDef *) 在这里的作用则是把 GPIOA_BASE 地址转换为GPIO_TypeDef 结构体指针类型。

4.STM32的时钟系统

为了实现低功耗，STM32设计了一套功能完善的但却非常复杂的时钟系统。普通的MCU，在配置好GPIO的寄存器后就可以使用，，但是STM32，还需开启外设时钟。

总结：今天晚上修改了隐藏文件，因为昨天晚上在.gitignore隐藏一些不必要的文件，没有自习观察，把所有的文件全部隐藏进去了。还有寄存器的地址是如何来的，stm32有外设四个总线，APB1,2,AHB1,2。了解stm32的时钟系统的一小部分。

2026年1月5号

补充1.4c语言关键字volatile内容和 “只读 / 读写” 这两个概念是可以组合使用的：

- “只读/读写” 是通过 const（是否允许程序修改）来定义的。volatile 只表示“值可能随时变化，禁止优化”，不决定能不能写 。

- 看是否有const关键字，如果有则只允许读，如果没有则允许读写。

- ```
  volatile int *p;        // 指向 volatile int 的普通指针（对象是 volatile）
  int * volatile q;       // 自身是 volatile 的指针，指向普通 int
  const volatile int *r;  // 指向 const volatile int（对象只读且 volatile）
  
  ```

   “对象是否只读/读写”：看对象类型里有没有 const。
  “对象是否 volatile”：看对象类型里有没有 volatile。
   指针本身是否 volatile，只影响指针值是否会被优化缓存，不影响指向对象的读写属性  。


1.时钟的分类:

| 分类方式 | 种类                   | 功能                                                         |
| -------- | ---------------------- | ------------------------------------------------------------ |
| 频率     | 高速时钟和低速时钟     | 高速时钟是提供给芯片主体的主时钟；<br />低速时钟只是提供给芯片中的RTC(实时时钟)及独立看门狗使用 |
| 时钟源   | 内部时钟源与外部时钟源 | 内部时钟是在芯片内部RC振荡器产生的，起震快，在芯片刚刚上电的时候，默认使用是内部高速时钟。<br />而外部时钟信号是由外部晶振输入的，在精度与稳定性上有很大优势，因此上电后通过软件配置，转而采用外部时钟信号。 |
|          |                        |                                                              |

2.如上表所示stm32有4种时钟源：

- 高速外部时钟（HSE）:以外部晶振作为时钟源，晶振频率可取范围为4~16Mhz,我们一般采用8MHZ。

- 高速内部时钟（HSI）：有内部RC震荡器产生，频率为8MHz,但是不稳定。

- 低速外部时钟（LSE）：以外部晶振为时钟源，主要是提供给实时时钟模块提供，一般是32.768KHz。

- 低速内部时钟（LSI）：有内部的RC振荡器产生，也是主要给实时时钟模块供电，一般是40kHz。

  

3.<img width="725" height="794" alt="Image" src="https://github.com/user-attachments/assets/889edcd3-4f24-4795-8c0e-fe4f86a5d940" />

 从 STM32 的 OSC 引脚接外部晶振得到 8MHz 的 HSE 时钟，先经过 PLLXTPRE 分频器（这里选不分频，保持 8MHz），再通过 PLLSRC 开关选 HSE 作为 PLL 的输入，用 PLLMUL 设 9 倍频得到 72MHz 的 PLLCLK；接着经 SW 开关选 PLLCLK 作为系统时钟 SYSCLK（72MHz）；SYSCLK 经 AHB 预分频器（不分频）输出 72MHz 的 HCLK，再通过 APB2 预分频器（不分频），最终让挂载在 APB2 上的 GPIO 外设时钟也达到 72MHz。 

总结：复习昨天看的内容，看了关于volatile与const的读写组合，今天晚上有初步了解8M的外部时钟是如何倍频到72Mhz的。