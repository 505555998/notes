<!---title:设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理-->
<!---keywords:DSP,FPGA-->
<!---date:old-->

硬件系统的构建如下图：

![][DSP_FPGA_System]

采用高速的C6000浮点系列DSP，方便做浮点运算。FPGA用于采集信号，DSP用于处理信号，分工合作。

去年设计的PCB，到现在，通过半年左右的适用时间，除了调试发现一些问题，还发现许多需要注意的问题。其中一些已经在文章

1. [高频总线上的串阻问题](./高频总线上的串阻问题.md)
2. [FPGA的复位](./FPGA的复位.md)
3. [DSP连接不上CCS3.3的问题讨论](./DSP连接不上CCS3.3的问题讨论.md)

等一些文章中进行过讨论。本文将特定对其它一些硬件设计上应该注意的问题进行整理，以防以后再出错。

### DSP部分

- DSP复位

TMS320C6713B内部没有上拉电阻，手册上推荐添加在复位管脚添加上拉电阻，上拉电阻放在复位管脚附近。

- DSP中断信号

不使用的DSP中断管脚使用下拉电阻配成默认非触发模式。

![][DSP-INT]

- EMIF接口

	- 串联33欧姆电阻
	- ECLKOUT添加测试点
	- 若使用Flash启动，Flash只能连接到CE1地址空间
	- 

### FPGA部分

- FPGA的PIN1与PIN2

特别注意FPGA的特殊管脚：PIN1与PIN2。PIN1是与EPCS通信的数据管脚，PIN2是FPGA的片选管脚，这两个管脚都将连接到AS下载口和EPCS芯片上。不能将这两个管脚连接到其它地方作他用。

![][FPGA12]

Altera官方的OrCAD库没将1、2管脚特殊标注，弄得设计时当做普通管脚用了，最后只能飞线，这也是这次设计最失败的地方！

- FPGA别忘了设计一个硬件复位信号

在设计FPGA的电路时，问一用过FPGA的同学关于FPGA最小系统需要复位吗？同学答：下面的电路就是复位，按一下就可以重新配置FPGA。于是我只设计了下面的电路。

![][FPGA-RESET]

直到写FPGA的程序，才发现，FPGA的nCFG信号是无法配置成Verilog中的reset信号的，我还缺少一个用于初始化Verilog变量的Reset信号，这个信号需要通过外部FPGA的普通管脚提供。

因此，设计电路时：请将FPGA的一个普通管脚映射为Verilog中的复位信号，复位电路与上面的电路一致。

- 将FPGA的某个管脚连接到DSP的中断管脚

在DSP与FPGA的通信过程中，为提高效率，用FPGA触发DSP的外部中断将是一种较好的方式。

- FPGA通过EMIF与DSP通信需要的信号

	- CE2 或 CE3, FPGA片选信号，都接到FPGA也没关系
	- AOE，DSP读FPGA信号
	- AWE，DSP写FPGA信号
	- ECLKOUT，时钟信号
	- EA，地址信号，EA[1:0]=00
	- ED，数据信号

- FPGA的配置信号

![][FPGA-Config]

### 其它

- 插入测试点

为电源信号、DSP的ECLKOUT信号添加测试点

- 电源

![][电源]

这种错误以后就别再犯傻了！

- 二极管是存在压降的

在设计电路时，为了防反插，添加了下面的电路，

![][SS14使用注意]

然后焊上，一测，输出端5V只有4.5V了。以后注意：任何二极管都存在压降的，SS14防反插只用在稳压芯片之前，并且得考虑SS14的压降是否会对稳压芯片的输入范围产生影响。



[DSP_FPGA_System]:../images/设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理/DSP_FPGA_System.png
[DSP-INT]:../images/设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理/DSP-INT.png
[FPGA12]:../images/设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理/FPGA12.png
[电源]:../images/设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理/电源.png
[FPGA-RESET]:../images/设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理/FPGA-RESET.png
[SS14使用注意]:../images/设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理/SS14使用注意.png
[FPGA-Config]:../images/设计DSP与FPGA数据采集硬件系统时遇到的一些问题整理/FPGA-Config.png