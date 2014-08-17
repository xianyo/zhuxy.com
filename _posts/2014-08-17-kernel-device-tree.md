---
layout: post
title: Kernel Device Tree
category: learn
tags: [linux kernel]

---


Device Tree在Linux内核驱动中的使用源于2011年3月17日Linus Torvalds在ARM Linux邮件列表中的一封邮件，他宣称“this whole ARM thing is a f\*cking pain in the ass”，并提倡学习PowerPC等其他架构已经成熟使用的Device Tree技术。自此，Device Tree正式进入ARM社区的视野中。

**本文章资源多取自[Device Tree Usage](http://devicetree.org/Device_Tree_Usage)**

<!--break-->

### 1. 作用

Device Tree是一种用来描述硬件的数据结构，类似板级描述语言，起源于OpenFirmware(OF)。在目前广泛使用的Linux kernel 2.6.x版本中，对于不同平台、不同硬件，往往存在着大量的不同的、移植性差的板级描述代码，以达到对这些不同平台和不同硬件特殊适配的需求。但是过多的平台、过的的不同硬件导致了这样的代码越来越多，最终引发了Linux创始人Linus的不满，以及强烈呼吁改变。Device Tree的引入给驱动适配带来了很大的方便，一套完整的Device Tree可以将一个PCB摆在你眼前。Device Tree可以描述CPU，可以描述时钟、中断控制器、IO控制器、SPI总线控制器、I2C控制器、存储设备等任何现有驱动单位。对具体器件能够描述到使用哪个中断，内存映射空间是多少等等。

###2. 基本数据格式

Device Tree由节点和属性构成。属性为key-value对，节点包括了各种属性，也可以包含子节点。下边列举一个简单的dts文件：

    / {
        node1 {
            a-string-property = "A string";
            a-string-list-property = "first string", "second string";
            a-byte-data-property = [0x01 0x23 0x34 0x56];
            child-node1 {
                first-child-property;
                second-child-property = <1>;
                a-string-property = "Hello, world";
            };
            child-node2 {
            };
        };
        node2 {
            an-empty-property;
            a-cell-property = <1 2 3 4>; /* each number (cell) is a uint32 */
            child-node1 {
            };
        };
    };

这个文件实际上没有任何意义，但却包含了基本所有要素：

- 1 唯一的根节点 “/”
- 2 一些节点：node1 node2
- 3 子节点 node1的子节点child-node1和child-node2
- 4 一群分散的属性

属性都是简单的key-value对，其中value也可以是空的或包含任意的byte流。以下是一些属性的基本数据结构：

- 1 双引号包含的字符信息

        string-property = "a string";
- 2 cells单位信息是32位无符号整型数据

        cell-property = <0xFF01 412 0x12341283>;
- 3 二进制数据流

        binary-property = [0x01 0x02 0x03 0x04];
- 4 混合数据用逗号隔开

        mixed-property = "a string", [0x01 0x02 0x03 0x04], <0xFF01 412 0x12341283>;
- 5 字符列表

        string-list = "string test1", "string test2";


###3. 一些基本概念

- 每个完整的dts文件必须拥有一个根节点
- dtsi文件一般为通用文件（类似C语言的头文件），可被其他文件include
后边的名字涵盖的范围更加广泛，如果可以匹配到，同样会以这个dts为基础进行初始化并启动。
- 父节点名应该取类型名，而不是IC名。节点名的命名规则一般是 name@address，也可以只有name而没有@之后的内容，但是要确保name不能重名。如果加了@以及地址，那么name可以相同，只要address不同即可。
- 每一个设备节点都要有一个compatible属性
- compatible的内容是用来匹配驱动的，组成方式为"manufacturer, model"，加入厂商名是为了避免重名。有的时候后边还会跟一个名字，如：

        compatible = "acme,coyotes-revenge", "acmd-board";

###4. 工作方式

#####a. 地址

设备的地址特性根据一下几个属性来控制：

- reg
- \#address-cells
- \#size-cells

reg意为region，区域。格式为：

    reg = <address1 length1 [address2 length2] [address3 length3]>;
父类的address-cells和size-cells决定了子类的相关属性要包含多少个cell，如果子节点有特殊需求的话，可以自己再定义，这样就可以摆脱父节点的控制。
address-cells决定了address1/2/3包含几个cell，size-cells决定了length1/2/3包含了几个cell。本地模块例如：

    spi@10115000 {
            compatible = "arm,pl022";
            reg = <0x10115000 0x1000 >;
    };
位于0x10115000的SPI设备申请地址空间，起始地址为0x10115000，长度为0x1000，即属于这个SPI设备的地址范围是0x10115000~0x10116000。

实际应用中，有另外一种情况，就是通过外部芯片片选激活模块。例如，挂载在外部总线上，需要通过片选线工作的一些模块：

    external-bus {
        #address-cells = <2>
        #size-cells = <1>;

        ethernet@0,0 {
            compatible = "smc,smc91c111";
            reg = <0 0 0x1000>;
        };

        i2c@1,0 {
            compatible = "acme,a1234-i2c-bus";
            #address-cells = <1>;
            #size-cells = <0>;
            reg = <1 0 0x1000>;
            rtc@58 {
                compatible = "maxim,ds1338";
                reg = <58>;
            };
        };

        flash@2,0 {
            compatible = "samsung,k8f1315ebm", "cfi-flash";
            reg = <2 0 0x4000000>;
        };
    };
external-bus使用两个cell来描述地址，一个是片选序号，另一个是片选序号上的偏移量。而地址空间长度依然用一个cell来描述。所以以上的子设备们都需要3个cell来描述地址空间属性——片选、偏移量、地址长度。在上个例子中，有一个例外，就是i2c控制器模块下的rtc模块。因为I2C设备只是被分配在一个地址上，不需要其他任何空间，所以只需要一个address的cell就可以描述完整，不需要size-cells。

当需要描述的设备不是本地设备时，就需要描述一个从设备地址空间到CPU地址空间的映射关系，这里就需要用到**ranges**属性。还是以上边的external-bus举例：

    #address-cells = <1>;
    #size-cells = <1>;
    ...
    external-bus {
        #address-cells = <2>
        #size-cells = <1>;
        ranges = <0 0  0x10100000   0x10000     // Chipselect 1, Ethernet
                  1 0  0x10160000   0x10000     // Chipselect 2, i2c controller
                  2 0  0x30000000   0x1000000>; // Chipselect 3, NOR Flash
    };
ranges属性为一个地址转换表。表中的每一行都包含了子地址、父地址、在自地址空间内的区域大小。他们的大小（包含的cell）分别由子节点的address-cells的值、父节点的address-cells的值和子节点的size-cells来决定。以第一行为例：

- 0 0 两个cell，由子节点external-bus的address-cells=2决定；
- 0x10100000 一个cell，由父节点的address-cells=1决定；
- 0x10000 一个cell，由子节点external-bus的size-cells=1决定。
最终第一行说明的意思就是：片选0，偏移0（选中了网卡），被映射到CPU地址空间的0x10100000~0x10110000中，地址长度为0x10000。

#####b. 中断

描述中断连接需要四个属性：
1. interrupt-controller 一个空属性用来声明这个node接收中断信号；
2. \#interrupt-cells 这是中断控制器节点的属性，用来标识这个控制器需要几个单位做中断描述符；
3. interrupt-parent 标识此设备节点属于哪一个中断控制器，如果没有设置这个属性，会自动依附父节点的；
4. interrupts 一个中断标识符列表，表示每一个中断输出信号。

如果有两个，第一个是中断号，第二个是中断类型，如高电平、低电平、边缘触发等触发特性。对于给定的中断控制器，应该仔细阅读相关文档来确定其中断标识该如何解析。

#####c. 其他

除了以上规则外，也可以自己加一些自定义的属性和子节点，但是一定要符合以下的几个规则：

1. 新的设备属性一定要以厂家名字做前缀，这样就可以避免他们会和当前的标准属性存在命名冲突问题；
2. 新加的属性具体含义以及子节点必须加以文档描述，这样设备驱动开发者就知道怎么解释这些数据了。描述文档中必须特别说明compatible的value的意义，应该有什么属性，可以有哪个（些）子节点，以及这代表了什么设备。每个独立的compatible都应该由单独的解释。
3. 新添加的这些要发送到devicetree-discuss@lists.ozlabs.org邮件列表中进行review，并且检查是否会在将来引发其他的问题。

###5. 进阶例子


    pci@0x10180000 {
            compatible = "arm,versatile-pci-hostbridge", "pci";
            reg = <0x10180000 0x1000>;
            interrupts = <8 0>;
            bus-ranges = <0 0>;

            #address-cells = <3>
            #size-cells = <2>;
            ranges = <0x42000000 0 0x80000000 0x80000000 0 0x20000000
                      0x02000000 0 0xa0000000 0xa0000000 0 0x10000000
                      0x01000000 0 0x00000000 0xb0000000 0 0x01000000>;
    };

像之前描述过的本地总线一样，PCI地址空间与CPU地址空间是完全分离的，所以这里需要通过定义ranges属性进行地址转化。
\#address-cells定义PCI使用3个cell，并且PCI的地址范围通过两个单位就可以解读。所以，首先的问题就是，为什么需要用3个32位的cell来描述一个PCI地址。

**这三个cell分别代表物理地址高位、中位、低位**：

- 1 phys.high cell : npt000ss bbbbbbbb dddddfff rrrrrrrr
- 2 phys.mid cell :  hhhhhhh hhhhhhhh hhhhhhhh hhhhhhh
- 3 phys.low cell :  llllllll llllllll llllllll llllllll

PCI地址为64位宽度，编码在phys.mid和phys.low中。真正重要的东西在于phys.high这一位空间中：

n：代表重申请空间标志（这里没有使用）
p：代表预读空间（缓存）标志
t：别名地址标志（这里没有使用）
ss：空间代码
  00： 设置空间
  01：IO空间
  10：32位存储空间
  11：64位存储空间

bbbbbbbb： PCI总线号。PCI有可能是层次性架构，所以我们可能需要区分一些子-总线
ddddd：设备号，通常由初始化设备选择信号IDSEL连接时申请。
fff：功能序号，有些多功能PCI设备可能用到。
rrrrrrrr：注册号，在设置周期使用。

    ranges = <0x42000000 0 0x80000000 0x80000000 0 0x20000000
              0x02000000 0 0xa0000000 0xa0000000 0 0x10000000
              0x01000000 0 0x00000000 0xb0000000 0 0x01000000>;
回头再看这个ranges分表代表了什么。父节点address-cells为1，子节点address-cells为3， 子节点size-cells为2。则第一行可以这样划分：

    0x42000000 0 0x80000000 子节点地址| 0x80000000 父节点地址| 0 0x20000000 地址空间长度|
0x42000000为phys.high，第一位为01000010，则p为1，ss为10，即申请32位存储空间为缓存空间。phys.mid为0，phys.low为0x80000000，他们共同组成了PCI地址，即表示从PCI总线的0x80000000地址处申请出一个32位的存储空间作为缓存。后边的那个cell 0x80000000 0 0x20000000代表到CPU空间后的参数，申请的地址被映射到CPU空间的0x80000000地址处，大小共计0x20000000(512MB)。




本文章转自[Kevin's Blog](http://airk000.github.io/linux%20kernel/2014/02/25/kernel-device-tree/)
