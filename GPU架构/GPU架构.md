
## Fermi硬件架构

![image-20220902135505719.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231739971.png)


### **Host Interface**

### **Input Assembler**

### GPC（Graphics Processing Cluster，图形处理簇）

组成：1*Raster Engine，4 *SM；

通过Crossbar来连接，GPC1处理完的数据，通过Crossbar分配给GPC2

![image-20220902135453986.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231740069.png)


#### Raster Engine(光栅化引擎)

![image-20220902145826120.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231740881.png)


#### SM（Stream Multiprocessor，流多处理器）

SM为**计算单元**

![image-20220902135106924.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231740783.png)
	

##### **Instruction Cache**

指令缓存。存放将要执行的指令，通过**Dispatch Units**填装到每个**运算核心（Core）**进行运算

##### Warp Scheduler

Warp调度模块。Warp的概念其实就是一组线程**Thread**，通常由32个线程组成，对应着32个运算核心。

Warp调度器的指令通过**Dispatch Units**送到**运算核心（Core）**执行

##### **Register File**

寄存器堆。存放将要处理的数据

##### LD/ST(Load/Store Unit)

加载/存储模块（Load/Store）。辅助一个Warp（线程组）从**Share Memory**或显存、加载(Load)或存储(Store)数据

##### SFU（Special function units）

特殊函数单元。与Adreno GPU中的初等函数单元（Elementary Function Unit，EFU）类似，执行特殊数学运算。由于其数量少，在高级数学函数使用较多时有明显瓶颈。

特殊函数例如以下几类：

- 幂函数：pow(x, a)、sqrt(x)
    
- 对数函数：log(x)、log2(x)
    
- 三角函数：sin(x)、cos(x)、tan(x)
    
- 反三角函数：asin(x)、acos(x)、atan(x)
    

##### Core（SP：stream processor）

运算核心；每个SM由32个运算核心组成。由**Warp Scheduler**调度，接收**Dispatch Units**的指令并执行

**Dispath Port**：接受分发单元的指令

**FPU**（floating point unit ）：浮点数单元

**Nnteger ALU**（arithmetic logic unit）:整数逻辑运算单元

![image-20220902143439454.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231740224.png)


##### **Interconnect Network**

内部链接网络

##### L1 Cache

L1缓存，不同GPU架构不一样，有些L1缓存和**Shared Memory**共用，有的L1缓存和**Texture Cache**共用

##### Uniform Cache

全局统一内存缓存

##### Tex Unit

纹理读取单元，Fermi有4个**Texture Units**，每个**Texture Unit**在一个运算周期最多可取4个采样器，这时刚好喂给一个**线程束（Warp）**（的16个车道），每个**Texture Uint**有16K的**Texture Cache**，并且在往下有**L2 Cache**的支持

##### Texture Cache

纹理缓存

##### PolyMorph Engine

多边形变形引擎。负责处理和多边形顶点相关的工作，包括以下模块。

- - **Vertex Fetch**：顶点处理前期的通过三角形索引取出三角形数据。
        
    - **Tesselator**：曲面细分
        
    - **Viewport Transform**：顶点的视口变换，三角形会被裁剪准备栅格化
        
    - **Attribute Setup**：属性装配，负责顶点的插值运算并输出给后续像素处理阶段使用
        
    - **Stream Output**：对应着DX10引入的新特性Stream Output
        

![image-20220902144735685.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231740142.png)


## GPU内存架构

![image-20220904222645939.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231741157.png)


Global Memory: 全局内存，显存；放置在GPU芯片的外部

L2 cache：GPU芯片内部，用来跨GPC

SM内：**L1 Cache/Shared Memory**、**Uniform Cache**、**Tex Unit**和**Texture Cache**以及**寄存器**都是存在于SM内部的

**存取速度**从上到下，依次变慢

![image-20220902150308378.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231741595.png)


## GPU逻辑管线

### The Application（应用程序阶段）

![image-20220902151653214.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231741802.png)


主要是CPU在准备数据，包括图元数据、渲染状态等，将数据传递给GPU的过程，图中显示数据如何进入GPU进行处理

CPU和GPU之间的数据传输是一个异步的过程，类似于CPU生产命令，GPU执行命令

#### Graphics API

CPU端通过调用图形API（OpenGL、Vulkan），将操作封装为一个个的**命令**，

#### PushBuffer

命令被推送到驱动程序，驱动程序检查命令合法性后，命令被存放到**命令队列中（FIFO Push Buffer）**，即上图中的**PushBuffer**（打包传递，减少开销）

#### Host Interface

当buffer写满或者显示调用Flush后，驱动程序把PushBuffer提交给GPU，并在命令队列末尾压入一条改变**Fence**值的命令,GPU通过Host Interface(主机接口)来接受这些命令

#### Front End

驱动进行调度，轮到这条队列时，将buffer中的数据存放到内存中的RingBuffer(内存大小固定)，RingBuffer好比一个旋转的水车，将命令一点一点“搬运”到GPU的**前端（Front End）**，当RingBuffer满载时，对**CPU进行堵塞**，直到读取到队列的最后一条命令改变Fence值，CPU解除堵塞

![image-20220819111709550.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231741983.png)


#### **Primitive Distributor**

**图元装配器（Primitive Distributor）**根据图元类型、顶点索引以及图元装配命令，开始分配渲染工作，提交上来n个三角形，分配给这几个PGC同时处理

①剔除重复的vertices index

②重新装配面片索引

③ 分成批次(batches)：至多32个顶点或32个图元（warp线程数32），然后发送给多个PGCs

![image-20220905103013831.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231741233.png)


tips：对于一些被复用的顶点，可能会被拆分到不同的batch中，这些顶点会被重复计算，但是在GPU的大规模并行中，这点消耗是值得的

### 顶点着色器

1、**PolyMorph Engine**的**Vertex Fetch模块**通过三角形索引(**Primitive Distributor**)，将数据从**DRAM显存**中取得三角形数据，传入SM的**L1 Cache**中

Tips:**Primitive Distributor**传入顶点索引，**Vertex Fetch**传入顶点数据

2、数据进入SM后，SM中的**线程调度器（Warp Scheduler）**为每个Shader核心函数（VS/GS/PS等）创建一个线程，并在一个**运算核心（Core）**上执行该线程。根据Shader需要的寄存器数量，在**寄存器堆（Register File）**中为每个线程分配指定数量的寄存器

Warp是**SIMT(单指令多线程)**的实现，即32个Thread执行的命令是相同的，只是线程数据不同，GPU的并行机制

3、SM的**Warp Scheduler**会按照顺序分发指令给整个warp，单个warp中的线程会**锁步(lock-step)**执行各自的指令，如果线程碰到不激活执行的情况也会被**遮掩(be masked out)**。

==详细分析见逻辑控制语句==

4、warp中的指令可以被一次完成，也可能经过多次调度，例如通常SM中的LD/ST(加载存取)单元数量明显少于基础数学操作单元

5、由于某些指令比其他指令需要更长的时间才能完成，特别是内存加载，warp调度器可能会简单地切换到另一个没有内存等待的warp，这是GPU如何克服内存读取延迟的关键，只是简单地切换活动线程组。为了使这种切换非常快，调度器管理的所有warp在寄存器文件中都有自己的寄存器。这里就会有个矛盾产生，shader需要越多的寄存器，就会给warp留下越少的空间，就会产生越少的warp，这时候在碰到内存延迟的时候就会只是等待，而没有可以运行的warp可以切换

![image-20220902150453304.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231742479.png)


### TCS/Hull

细分着色器

1. Tessellation Control Shader（hull shader）：决定每个patch要**细分到什么程度**
    
2. Tessellation Primitive Generation（tessellator）：根据Tessellation Control Shader传出的两个参数，**执行内置的一套细分规则**
    
3. Tessellation Evaluation Shader（domain shader）：本质上和顶点着色器一样，只不过处理的是细分后的所有顶点（**包含无中生有出来的顶点**）
    

### Polymorph Engine

①剔除

视锥剔除：三角形裁剪

与近平面相交：裁剪，计算简单

与其他面相交：不裁剪，计算麻烦

与其他面相交&数值过大：顶点坐标超过了硬件处理范围，必须裁剪

②透视除法

xyz同除w，统一坐标空间**NDC**（Normalized Device Coordinates），NDC的范围 **[-1,1]** * **[-1,1]** * **[-1,1]**

③视口变换

被Viewport Transform模块处理,投影到屏幕uv坐标，GPU会使用L1和L2缓存来进行vertex-shader和pixel-shader的数据通信

TIPS：屏幕坐标为小数，像素坐标为整数下标，屏幕坐标转化为像素坐标在光栅化完成

![image-20220902172843646.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231742403.png)


### 像素着色器

接下来这些三角形将被分割，再分配给多个GPC，三角形的范围决定着它将被分配到哪个光栅引擎(raster engines)，每个raster engines覆盖了多个屏幕上的tile，这等于把三角形的渲染分配到多个tile上面。也就是**像素阶段就把按三角形划分变成了按显示的像素划分**了；数据再次**并行**处理

![image-20220902173004606.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231742364.png)


3、SM上的Attribute Setup保证了从vertex-shader来的数据经过插值后是pixel-shade是可读的???

**PolyMorph Engine** TO **raster engine**

GPC上的光栅引擎(raster engines)在它接收到的三角形上工作，来负责这些这些三角形的像素信息的生成（同时会处理裁剪Clipping、背面剔除和Early-Z剔除）

5、32个像素线程将被分成一组，或者说8个2x2的像素块，这是在像素着色器上面的最小工作单元，在这个像素线程内，如果没有被三角形覆盖就会被遮掩，SM中的warp调度器会管理像素着色器的任务

6、接下来的阶段就和vertex-shader中的逻辑步骤完全一样，但是变成了在像素着色器线程中执行。 由于不耗费任何性能可以获取一个像素内的值，导致锁步执行非常便利，所有的线程可以保证所有的指令可以在同一点

![image-20220902181826096.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231742631.png)


最后一步，现在像素着色器已经完成了颜色的计算还有深度值的计算，在这个点上，我们必须考虑三角形的原始api顺序，然后才将数据移交给ROP(render output unit，渲染输入单元)，一个ROP内部有很多ROP单元，在ROP单元中处理**深度测试**，和framebuffer的混合，深度和颜色的设置必须是原子操作，否则两个不同的三角形在同一个像素点就会有冲突和错误

## 重要概念

### SIMD & SIMT

#### SIMD（single instruction multiple date）

单指令多数据：在Core的ALU单元内，一条指令可以处理多维向量

vec4 c = a + b;  
//SIMD  
SIMD_ADD c, a, b

SISD 单指令单数据：四维向量的运算，需要四条指令

vec4 c = a + b;  
//SISD  
ADD c.x, a.x, b.x  
ADD c.y, a.y, b.y  
ADD c.z, a.z, b.z  
ADD c.w, a.w, b.w

#### SIMT(single instruction multiple thread)

单指令多线程：给SM一个指令，让其中多个Core同时执行该指令，可以读取不同的值（每个Core所分配的寄存器）

![image-20220904143740337.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231742827.png)


### **co-issue**

充分利用SIMD的运算单元

![image-20220904144233406.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231743393.png)


![image-20220904144240959.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231743313.png)


**详情见：标量指令着色器**（Scalar Instruction Shader）

### ==条件分支==

#### CPU & GPU

![image-20220904221815203.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231743616.png)


CPU中有大量的存储单元和控制单元，相比GPU，ALU计算单元很少；

CPU不擅长进行大规模并行计算，更擅长逻辑控制

相反，**GPU适合大规模并行运算，不擅长逻辑控制**，避免在shader中进行大量分支结构

#### CPU：分支预测

CPU擅长逻辑控制的原理来自于CPU的分支预测机制

一条指令在CPU上执行，需要经历以下四个阶段：

1. fetch（获取指令）
    
2. decode（解码指令）
    
3. execute（执行指令）
    
4. write-back（写回数据）
    

执行效率最低：等A指令的四个阶段都执行完，再执行B指令

**CPU的执行策略**

![image-20220904152140361.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231743204.png)


A指令fetch后，A继续decode，B此时进行fetch

如果是**执行判断分支**

if （condition）//A
{
	X=X+1;		//B
} else
	X=X-1		//c

![image-20220904152235330.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231743965.png)


当if的条件判断fetch后，还未到execute，无法知道条件判断的结果

此时，编译器不清楚该执行B，还是执行C，CPU的**预测分支**机制就排上用场了

分支预测：根据历史记录，先执行B或C分支

如果猜测对了，那就赚了，节省了等待if执行完的时间开销

如果猜测错了，反正不亏，先回滚，再执行另外的分支

#### GPU：遮掩

GPU中没有那么多控制单元，无法进行分支预测；

GPU中通过**lock-step**和**遮掩**机制来处理判断分支

**lock-step**：GPU中执行指令的方式以lock-step进行锁步操作，所有Core在同一时间执行的指令都是相同的，当所有Core执行完当前指令后，Dispatch unit才会分配下一条指令

**遮掩**

![image-20220904155614676.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231744755.png)


图中有8个ALU来执行指令，黄色的代表ture分支的执行语句，蓝色的代表false分支的执行语句

不同的ALU执行相同的语句，但是进行条件判断的值是不同相同的，这就导致有的ALU执行ture分支，有的ALU执行falese分支；并且这些分支是lock-step，同意时间只能执行相同的语句

因此，**遮掩**机制让所有ALU按顺序执行:

先执行true分支，条件判断为true的分支正常执行；条件判断为false的分支**被遮掩**，不执行，处于等待状态

再执行false分支，条件判断为ture的分支**被遮掩**；条件判断为false的分支正常执行

遮掩和lock-step机制使得GPU在并行运算的过程中，也能处理逻辑运算

BUT！！!

GPU中的逻辑运算 更像是 顺序执行，被遮掩的代码需要消耗相同的执行时间去等待

因此，shader中有大量**if-else**语句，会白白浪费很多时间; **for循环**同理，有的ALU的分支执行3次循环，有的ALU执行5次循环；3次循环的ALU后期被遮掩，等待所有ALU执行完毕

**Shader里不要用不固定的数值来控制逻辑执行**

### Early-Z

![image-20220904221433696.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231744573.png)


### 统一逻辑架构

早期，VS、PS各自适用不同类型的core；这导致，在顶点任务多时，像素core处于闲置状态

因此，现在所有的着色器都适配同一种core

### 像素块(**Pixel Quad**)

32个像素线程将被分成一组，或者说8个**2x2的像素块**，这是在**像素着色器上面的最小工作单元**，在这个像素线程内，如果没有被三角形覆盖就会**被遮掩**，SM中的warp调度器会管理像素着色器的任务

在像素着色器中，会将相邻的四个像素作为不可分割的一组，送入SM的4个core中进行处理

**缺点**

![image-20220904222037332.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231744369.png)


为了绘制绿色的三个像素，需要将这三个像素及其像素块进行处理，即：绘制3个绿色像素，调用了12个core绘制12次，造成大量资源浪费

**优点**

1、简化和加速像素分派的工作

2、精简SM的架构，减少硬件单元数量和尺寸

3、降低功耗，提高效能比。

4、无效像素虽然不会被存储结果，但可辅助有效像素求导函数

### Warp（线程束）

**线程调度器**会将所有线程分为若干个组，每一个组叫做一个**线程束（Warp）**，它又包含了32个线程。

因为GPU是**lock-step**的方式执行，为保持指令一致，所以**线程调度器**调度的单位是**线程束（Warp）**

但是，每当当前**线程束（Warp）**遇到费时操作，它就会被**阻塞（Stall）**。为了降低延迟，GPU的**线程调度器**会采用一种简单而有效的策略，就是切换另一组线程来运行。

![v2-009fe452d12c40b901111457a371f481_720w.jpg](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231744653.jpg)


## 参考文献

[Life of a triangle - NVIDIA's logical pipeline | NVIDIA Developer](https://developer.nvidia.com/content/life-triangle-nvidias-logical-pipeline)

[深入GPU硬件架构及运行机制 - 0向往0 - 博客园 (cnblogs.com)](https://www.cnblogs.com/timlly/p/11471507.html)

[GPU架构探秘之旅 - 知乎 (zhihu.com)](https://www.zhihu.com/column/c_1351502583832354816)

[GPU架构和渲染 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/441610596)


  


