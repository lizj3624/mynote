首先给大家推荐一下我老师大神的人工智能教学网站。教学不仅零基础，通俗易懂，而且非常风趣幽默，还时不时有内涵黄段子！[点这里可以跳转到网站](https://www.captainbed.net/lhyd)

**PS：笔者强烈建议诸位注册一个EETOP的账号，每天签到或者发贴、回贴就有积分了，里面的资源非常丰富，各种软件、资料都能找到。**

**一、入门首先要掌握HDL（HDL=verilog+VHDL）。**

​    第一句话是：还没学数电的先学[**数电**](http://bbs.eetop.cn/thread-157244-1-1.html)。然后你可以选择verilog或者VHDL，有C语言基础的，建议选择VHDL。因为verilog太像C了，很容易混淆，最后你会发现，你花了大量时间去区分这两种语言，而不是在学习如何使用它。当然，你思维能转得过来，也可以选verilog，毕竟在国内verilog用得比较多。

​    接下来，首先找本实例抄代码。抄代码的意义在于熟悉语法规则和编译器（这里的编译器是硅编译器又叫综合器，常用的编译器有：[**Quartus**](http://blog.csdn.net/k331922164/article/details/46275247)、[**ISE**](http://bbs.eetop.cn/thread-451518-1-1.html)、[**Vivado**](http://bbs.eetop.cn/thread-598057-1-1.html)、[**Design Compiler**](http://bbs.eetop.cn/thread-558981-1-1.html) 、[**Synopsys**](http://www.synopsys.com/Tools/Implementation/FPGAImplementation/Pages/default.aspx)的[**VCS**](http://bbs.eetop.cn/thread-412008-1-1.html)、iverilog、**[Lattice](http://www.latticesemi.com/Products.aspx)**的[**Diamond**](http://blog.csdn.net/k331922164/article/details/51379364)、Microsemi/Actel的[**Libero**](https://www.microsemi.com/products/fpga-soc/design-support/fpga-soc-design)、[**Synplify pro**](http://bbs.eetop.cn/thread-578576-1-1.html)），然后再模仿着写，最后不看书也能写出来。编译完代码，就打开RTL图，看一下综合出来是什么样的电路**。**

​    HDL是硬件描述语言，突出硬件这一特点，所以要**用数电的思维去思考HDL，而不是用C语言或者其它高级语言**，如果不能理解这句话的，可以看《[**什么是硬件以及什么是软件**](http://blog.csdn.net/k331922164/article/details/46730523)》。在这一阶段，推荐的教材是**《[Verilog传奇](http://product.dangdang.com/24036476.html)》、《[Verilog HDL高级数字设计](http://bbs.eetop.cn/thread-281674-1-1.html)》**或者是《[**用于逻辑综合的VHDL**](http://search.dangdang.com/?key=%20%D3%C3%D3%DA%C2%DF%BC%AD%D7%DB%BA%CF%B5%C4VHDL&act=input)》。不看书也能写出个三段式状态机就可以进入下一阶段了。

​    此外，你手上必须准备Verilog或者VHDL的官方文档，《[**verilog_IEEE官方标准手册-2005_IEEE_P1364**](http://bbs.eetop.cn/thread-556080-1-1.html)》、《[**IEEE Standard VHDL Language_2008**](http://bbs.eetop.cn/thread-436703-1-1.html)》，以便遇到一些语法问题的时候能查一下。

**二、独立完成中小规模的数字电路设计。**

​    现在，你可以设计一些数字电路了，像交通灯、电子琴、DDS等等，推荐的教材是《[**Verilog HDL应用程序设计实例精讲**](http://bbs.eetop.cn/viewthread.php?tid=587313)》。在这一阶段，你要做到的是：给你一个指标要求或者时序图，你能用HDL设计电路去实现它。这里你需要一块开发板，可以选[**Altera**](https://www.altera.com/support/literature/lit-index.smartphone.html)的cyclone IV系列，或者[**Xilinx**](http://china.xilinx.com/support.html)的Spantan 6。**还没掌握HDL之前千万不要买开发板，因为你买回来也没用**。这里你没必要每次编译通过就下载代码，咱们用[**modelsim仿真**](http://blog.csdn.net/k331922164/article/details/47988847)（此外还有[**QuestaSim**](http://bbs.eetop.cn/thread-452999-1-1.html)、[**NC verilog**](http://bbs.eetop.cn/thread-471086-1-1.html)、Diamond的Active-HDL、VCS、Debussy/[**Verdi**](http://wenku.baidu.com/link?url=1cMQfcz0XQVXzvk2bOj_hLCfy6EAZR8KmlmRl-7pMnq-BCz8bLzYhfmQXcV9aqxVO0EB9rfV0X1nXgEpUUmUOQFv682BHIUB8HFUwOGfNoO)等仿真工具），如果仿真都不能通过那就不用下载了，肯定不行的。在这里先掌握简单的testbench就可以了。推荐的教材是《[**WRITING TESTBENCHES Functional Verification of HDL Models**](http://bbs.eetop.cn/thread-413725-1-1.html)》。

**三、掌握设计方法和设计原则。**

​    你可能发现你综合出来的电路尽管没错，但有很多警告。这个时候，你得学会同步设计原则、优化电路，是速度优先还是面积优先，时钟树应该怎样设计，怎样同步两个异频时钟等等。推荐的教材是《[**FPGA权威指南**](http://bbs.eetop.cn/thread-335134-1-1.html)》、《[**IP核芯志-数字逻辑设计思想**](https://www.amazon.cn/IP核芯志-数字逻辑设计思想-吴涛/dp/B0153NAS28/ref=sr_1_1?ie=UTF8&qid=1460103068&sr=8-1&keywords=IP核芯志-数字逻辑设计思想)》、《Altera FPGA/CPLD设计》第二版的[**基础篇**](http://bbs.eetop.cn/thread-236729-1-1.html?tid=236729&extra=page%3D1&page=1)和[**高级篇**](http://bbs.eetop.cn/thread-429297-1-1.html)两本。学会加快编译速度（增量式编译、LogicLock），静态[**时序分析**](http://blog.csdn.net/k331922164/article/details/48687161)（[**timequest**](http://wenku.baidu.com/link?url=f55u1aL6d5XKysDuD4keltciKNk46aUXn39IF0hINNjaTqHxOeh34PfZDkudvU3JuI1RuwT9DPsow_kRFe9qhWYZbG7uWVo-AoAAylvlLP3)），嵌入式逻辑分析仪（[**signaltap**](http://blog.csdn.net/k331922164/article/details/47623501)）就算是通关了。如果有不懂的地方可以暂时跳过，因为这部分还需要足量的实践，才能有较深刻的理解。

**四、学会提高开发效率。**

​    因为Quartus和ISE的编辑器功能太弱，影响了开发效率。所以建议使用**[Sublime text编辑器](http://blog.csdn.net/k331922164/article/details/48092291)**中代码片段的功能，以减少重复性劳动。Modelsim也是常用的仿真工具，学会TCL/TK以编写适合自己的**[DO文件](http://blog.csdn.net/k331922164/article/details/50001035)**，使得仿真变得自动化，推荐的教材是《[**TCL/TK入门经典**](http://www.jb51.net/books/304937.html)》。你可能会手动备份代码，但是专业人士都是用版本控制器**[Git](https://www.runoob.com/git/git-tutorial.html)**的，可以提高工作效率。文件比较器[**Beyond Compare**](http://www.beyondcompare.cc/xiazai.html)也是个比较常用的工具，Git也有比较功能。此外，你也可以使用[**System Verilog**](http://bbs.eetop.cn/thread-387263-1-4.html)来替代testbench，这样效率会更高一些。如果你是做IC验证的，就必须掌握System Verilog和验证方法学（UVM）。推荐的教材是《[**Writing Testbenches using SystemVerilog**](http://bbs.eetop.cn/thread-587167-1-1.html)》、《[**The UVM Primer**](http://bbs.eetop.cn/thread-479340-1-1.html)》、《[**System Verilog1800-2012语法手册**](http://bbs.eetop.cn/thread-387263-1-1.html)》。

​     掌握了TCL/TK之后，可以学习[**虚拟Jtag**](http://blog.csdn.net/k331922164/article/details/52093292)（ISE也有类似的工具）制作属于自己的调试工具，此外，有时间的话，最好再学个python。脚本，意味着一劳永逸。

**五、增强理论基础。**

​    这个时候，你已经会使用FPGA了，但是还有很多事情做不了（比如，FIR滤波器、[**PID算法**](http://blog.csdn.net/k331922164/article/details/51146507)、OFDM等），因为理论没学好。我大概地分几个方向供大家参考，后面跟的是要掌握的理论课。

1、信号处理——**[信号与系统](https://blog.csdn.net/k331922164/article/details/55006763)**、数字信号处理、数字图像处理、现代数字信号处理、盲信号处理、自适应滤波器原理、雷达信号处理

2、接口应用——如：[**UART**](http://blog.csdn.net/k331922164/article/details/51429544)、**[SPI](https://wenku.baidu.com/view/1d162f7187c24028915fc3da.html?from=search)**、[**IIC**](https://wenku.baidu.com/view/8f9df95f804d2b160b4ec0b3.html?from=search)、[**USB**](http://blog.csdn.net/k331922164/article/details/53349360)、[**CAN**](https://wenku.baidu.com/view/f6cf8081d4d8d15abe234ecb.html)、[**PCIE**](http://bbs.eetop.cn/thread-600329-1-1.html)、[**Rapid IO**](http://www.rapidio.org/)、[**DDR**](https://wenku.baidu.com/view/c1609388d4d8d15abe234e6f.html)、[**TCP/IP**](http://www.jb51.net/books/66960.html)、**[SPI4.2](https://wenku.baidu.com/view/8093e3edf8c75fbfc77db296.html)**(10G以太网接口)、[**SATA**](http://bbs.eetop.cn/thread-594135-1-1.html)、光纤、**[DisplayPort](http://bbs.eetop.cn/thread-315534-1-1.html)**、HDMI

3、无线通信——信号与系统、数字信号处理、通信原理、移动通信基础、随机过程、信息论与编码

4、CPU设计——计算机组成原理、**[单片机](http://blog.csdn.net/k331922164/article/details/44681093)**、计算机体系结构、编译原理、**[RISC-V](https://riscv.org/risc-v-cores/)**

5、仪器仪表——模拟电子技术、高频电子线路、电子测量技术、智能仪器原理及应用

6、控制系统——自动控制原理、现代控制理论、过程控制工程、模糊控制器理论与应用

7、压缩、编码、加密——数论、抽象代数、现代编码技术、信息论与编码、数据压缩导论、应用密码学、音频信息处理技术、数字视频编码技术原理

​    现在你发现，原来FPGA会涉及到那么多知识，你可以选一个感兴趣的方向，但是工作中很有可能用到其中几个方向的知识，所以理论还是学得越多越好。如果你要更上一层，数学和英语是不可避免的。

**六、学会使用MATLAB仿真。**

​    设计FPGA算法的时候，多多少少都会用到MATLAB，比如[**CRC**](http://blog.csdn.net/k331922164/article/details/51648707)的系数矩阵、数字滤波器系数、各种表格和文本处理等。此外，MATLAB还能用于调试HDL（用MATLAB的计算结果跟用HDL算出来的一步步对照，可以知道哪里出问题）。推荐的教材是《[**MATLAB宝典**](http://www.jb51.net/books/104042.html)》和杜勇的《[**数字滤波器的MATLAB与FPGA实现**](http://search.dangdang.com/?key=%CA%FD%D7%D6%C2%CB%B2%A8%C6%F7%B5%C4MATLAB%D3%EBFPGA%CA%B5%CF%D6&act=input)》。

**七、足量的实践。**

​    这个时候你至少读过几遍芯片手册（**[官网](https://www.altera.com/support/literature/lit-index.html)**有），然后可以针对自己的方向，做一定量的实践了（期间要保持良好的[**代码风格**](http://blog.csdn.net/k331922164/article/details/52166038)，[**增加元件例化语句的可读性**](http://blog.csdn.net/k331922164?viewmode=list)，绘制[**流程图/时序图**](http://blog.csdn.net/k331922164/article/details/50541541)，[**撰写文档**](http://blog.csdn.net/k331922164/article/details/50539863)的习惯）。比如：通信类的可以做调制解调算法，仪表类的可以做总线分析仪等等。不过这些算法，在书上只是给了个公式、框图而已，跟实际的差距很大，你甚至会觉得书上的东西都很肤浅。那么，你可以在[**知网**](http://www.cnki.net/)、**[百度文库](http://wenku.baidu.com/)**、**[EETOP论坛](http://bbs.eetop.cn/)**、**[opencores](http://opencores.org/projects)**、**[ChinaAET](http://blog.chinaaet.com/)**、**[SCI-HUB](http://tool.yovisun.com/scihub/)**、Q群共享、博客上面找些相关资料（校外的朋友可以在淘宝买个知网账号）。其实，当你到了这个阶段，你已经达到了职业级水平，有空就多了解一些前沿技术，这将有助于你的职业规划。

​    在工作当中，或许你需要关注很多协议和行业标准，协议可以在EETOP上面找到，而标准（如：国家标准GB和GB/T，国际标准ISO）就推荐《[**标准网**](http://www.biaozhuns.com/)》和《[**标准分享网**](http://www.bzfxw.com/)》。

**八、图像处理。**（这部分只写给想学图像处理的朋友，也是由浅入深的路线）

1、Photoshop。花一、两周的时间学习PS，对图像处理有个大概的了解，知道各种图片格式、直方图、色相、通道、滤镜、拼接等基本概念，并能使用它。这部分是0基础，目的让大家对图像处理有个感性的认识，而不是一上来就各种各样的公式推导。推荐《**[Photoshop CS6完全自学教程](http://www.jb51.net/books/100972.html)**》。

2、基于MATLAB或OpenCV的图像处理。有C/C++基础的可以学习OpenCV，否则的话，建议学MATLAB。这个阶段下，只要学会简单的调用函数即可，暂时不用深究实现的细节。推荐《**[数字图像处理matlab版](http://bbs.eetop.cn/thread-305112-1-1.html)**》、《**[学习OpenCV](http://www.jb51.net/books/86684.html)**》。

3、图像处理的基础理论。这部分的理论是需要高数、复变、线性代数、信号与系统、数字信号处理等基础，基础不好的话，建议先补补基础再来。看不懂的理论也可以暂时先放下，或许学到后面就自然而然地开窍了。推荐《**[数字图像处理](http://bbs.eetop.cn/thread-252932-1-1.html)**》。

4、基于FPGA的图像处理。把前面学到的理论运用到FPGA上面，如果这时你有前面第七个阶段的水平，你将轻松地独立完成图像算法设计（图像处理是离不开接口的，上面第五个阶段有讲）。推荐《**[基于FPGA的嵌入式图像处理系统设计](https://www.amazon.cn/dp/B00BPXFUVK/ref=wl_it_dp_o_pd_nS_ttl?_encoding=UTF8&colid=1KD25DEC6Q598&coliid=I1BCWMH1TX87Q5)**》、《[**基于FPGA的数字图像处理原理及应用**](http://product.dangdang.com/24171633.html#preface)》。

5、进一步钻研数学。要在算法上更上一层，必然需要更多的数学，所以这里建议学习[**实分析**](http://blog.csdn.net/k331922164/article/details/52842206)、**[泛涵分析](http://wenku.baidu.com/link?url=Fk0k8pCAe8PlvAk35gVwQgYUbMaQ8FILvJXINDJA-1jyB1bDaMvRi-D-e3zl4-CUdHwdpEqasojlnPeA2_cW5UbbF1l0Ig2OM1bTd_6pn_q)**、**[小波分析](http://wenku.baidu.com/link?url=sIxDcV7Aju1bNj0eqZj1-1zJrs0P1ZCg558bXfyO5NDNo6oRWz5QHl3fcAoe41yxi_oH9k0DuPy_7qznsF7QLEMNUh8ELJR-cFuzpZavrve)**等。

下面这两个阶段是给感兴趣的朋友介绍的。

**九、数电的尽头是模电。**

​    现在FPGA内部的事情是难不到你的，但是信号出了FPGA，你就没法控制了。这个时候必须学好模电。比如：电路分析、模拟电子技术、高频电子线路、PCB设计、EMC、SI、PI等等，能设计出一块带两片DDR3的FPGA开发板，就算通关了。具体的学习路线可以参考本博客的《**[如何学习硬件设计——理论篇](http://blog.csdn.net/k331922164/article/details/45102489)**》和《**[如何学习硬件设计——实践篇](http://blog.csdn.net/k331922164/article/details/46844339)**》。

**十、学无止境。**

​    能到这个境界，说明你已经很厉害了，但是还有很多东西要学的，因为FPGA常常要跟CPU交互，也就是说你得经常跟软件工程师交流，所以也得懂点软件方面的知识。比如ARM（Xilinx的ZYNQ和Altera的SOC会用到ARM的硬核，请参考本博客的《**[如何学习嵌入式软件](http://blog.csdn.net/k331922164/article/details/50629131)**》）、**[DSP](http://blog.csdn.net/k331922164/article/details/78734859)**、Linux、安卓、上位机（[**QT**](http://blog.csdn.net/k331922164/article/details/52729675)、C#、JAVA）都可以学一下，反正学无止境的。

**十一、其它问题。**

a、为什么不推荐学习NIOS II和MicroBlaze等软核？

   1、性价比不高，一般的软核性能大概跟Cortex M3或M4差不多，用FPGA那么贵的东西去做一个性能一般的CPU，在工程上是非常不划算的。不如另外加一块M3。

   2、加上软核，可能会影响到其它的逻辑的功能。这是在资源并不十分充足的情况下，再加上软核，导致布局布线变得相当困难。

   3、软核不开源，出现Bug的时候，不容易调试。

   4、工程上很少使用，极有可能派不上用场。

b、为什么不推荐0基础学习ZYNQ或SOC？

   1、容易让人有傍同心理。傍同心理是指一个人通过渲染与自己有亲近关系的人的杰出，来掩盖和弥补自己在这方面的不足，从而获得心理上的平衡。自己在学习很厉害的东西，然后也感觉自己很厉害，但这只是错觉而已。

   2、入门应该学习尽量简单的东西，要么专心学习ARM，要么专心学习FPGA。这样更容易有成就感，增强信心。

   3、ZYNQ和SOC的应用领域并不广，还有很多人没听过这种东西，导致求职的不利。

   4、开发工具编译时间长，浪费较多时间。

   5、绝大多数工作，都只是负责一方面，也就是说另一方面，很有可能派不上用场。

c、为什么已经存在那么多IP核，仍然需要写HDL？

   1、问这种问题的，一般是学生，他们没有做过产品，没有遇到过工程上的问题。

   2、IP核并非万能，不能满足所有需求。

   3、尽量少用闭源IP核，一旦出问题，这种黑匣子很可能让产品难产。

   4、深入理解底一层次，可以更好地使用高一层次。该法则可以适用于所有编程语言。

d、推荐一些微电子的教学视频。

   可以参考本博客的《**[微电子教学视频–Silicon Run等](https://blog.csdn.net/k331922164/article/details/85047746)**》。

[点这里可以跳转到人工智能网站](https://www.captainbed.net/lhyd)