[Vulkan Tutorial)](https://vulkan-tutorial.com/Introduction)

[Vulkan Tutorial-CN](https://zhuanlan.zhihu.com/p/56338417)

  

Vulkan学习指南

[Learning Vulkan](https://github.com/PacktPublishing/Learning-Vulkan)

  

[Vulkan: Examples](https://github.com/SaschaWillems/Vulkan)

  

[Vulkan从入门到精通1 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/430397192)

[Vulkan 学习指南 - 知乎 (zhihu.com)](https://www.zhihu.com/column/c_1033291907413250048)

[Vulkan文章汇总 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/616082929)
 

## Swap Chain(Mulit Buffer)

交换链或多缓冲

在单缓冲的情况下，画面时逐行逐像素的刷新，同一个缓冲一边显示一边用来处理数据，会导致最终画面出现**闪烁、撕裂**等

因此，采用双缓冲，缓冲A用来呈现画面，缓冲B用来后台处理数据，数据处理完成后，交换AB，使得始终有缓冲用来处理数据，不会停下来，同时保证呈现画面始终的处理完数据的画面，而不是处理一半的画面

In [computer science](https://en.wikipedia.org/wiki/Computer_science), **multiple buffering** is the use of more than one [buffer](https://en.wikipedia.org/wiki/Buffer_(computer_science)) to hold a block of data, so that a "reader" will see a complete (though perhaps old) version of the data, rather than a partially updated version of the data being created by a ["writer"](https://en.wikipedia.org/wiki/Readers-writers_problem). It is very commonly used for computer display images. It is also used to avoid the need to use [dual-ported RAM](https://en.wikipedia.org/wiki/Dual-ported_RAM) (DPRAM) when the readers and writers are different devices.

[多重缓冲 - 维基百科 (wikipedia.org)](https://en.wikipedia.org/wiki/Multiple_buffering)

## 思路：

vulkan需要显示的指定所有配置，绘制一个三角形需要1500+行代码，初学容易迷失在细节中

暂时不去过多了解部件的具体细节，先建立整体流程框架的认知，再填充细节

## 基本概念

#### 快速建立体系认知

[30分钟入门](https://www.zhihu.com/search?type=content&q=gpu%20vulkan)

[(18 封私信 / 17 条消息) 如何正确的入门Vulkan？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/424430509/answer/1632072443)

创建Vulkan对象的函数参数的一般形式就是：

- 一个包含了创建信息的结构体指针
    
- 一个自定义的分配器回调函数，在本教程，我们没有使用自定义的内存分配器，总是将它设置为nullptr
    
- 一个指向新对象句柄存储位置的指针
    
- 返回一个result，反馈是否创建成功
    

creat：创建对象

destory：销毁对象

CreateInfo：包含创建信息的结构体

**Enumerate**xxx**Properties**：用来查询


```
uint32_t layerCount;  
vkEnumerateInstanceLayerProperties(&layerCount, nullptr);   //查询数量  

std::vector<VkLayerProperties> availableLayers(layerCount); //根据数量分配内存  
vkEnumerateInstanceLayerProperties(&layerCount,availableLayers.data()); //查询属性
```


**QueueFamily**：队列族，Graphic：图形队列族，Compute计算队列族、 Transfer传输数据队列族；每个队列族都有多个队列

**Queue**：对硬件执行流水线的抽象，通过队列来提交命令，并且每个队列只能有一个线程访问，且命令的执行在队列上按顺序执行

vkCmdxx：录制命令，push_back到command buffer中

vkQueuexx：提交命令

现代API将命令分成录制和提交，本质上为了更好的实现多线程

**Buffer**：Vulkan主要有两种 Buffer 和 Image，Buffer一般用于vertex、index以及uniform，Image用于位图数据，也就是贴图。而Buffer要真正使用起来还得配备一个 VkDeviceMemory，下面会说。

1. **Mesh**：这个比较好理解，vertex、index各自对应一个buffer对象和一个memory对象组成一个Mesh，tutorial里也算写的很清楚。
    
2. **Texture**：VkImage 除了 VkDeviceMemory 还需要 VkImageView 和 VkImageSampler，VkImageView 相当于一个 accessor，具体操作都需要操作这个accessor，所以基本上VkImage和VkImageView就是个强绑定关系。VkImageSampler是采样器，anisotropy、mips等在这里设置，可以多个图共享一个采样器。
    
3. **Uniform**：类似Mesh，只不过这个Buffer对应的**结构体一定一定要注意内存对齐的问题！！！**不同编译器不同平台编出来的内存对齐模式都可能不一样，而vulkan对传过来的数据是有对齐要求的，这个问题我吃过药，tutorial那一章最后写过，当时给我跳过去了，结果shader里拿到的数据怎么都不对，更要命的是这个问题是validation layer无法检查出来的！
    

**Memory**：VK中开放GPU的内存分配，VkDeviceMemory ；并且，可分配次数有限，需要suballocation（vulkan memory allocator）

**Synchronization**：

vulkan不光让你自己管理内存，同步也需要手工做，cpu提交之后如何知道gpu完成任务了呢，所以有fence,充当cpu和gpu之间的同步工具，cpu通过等待fence知道gpu完成任务。

#### 创建实例

创建一个Vulkan实例(VkInstance)来完成Vulkan的初始化；
```
 VkApplicationInfo appInfo = {};  
        appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;  
        appInfo.pNext = nullptr;  
        appInfo.pApplicationName = "Hello Triangle";  
        appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0);  
        appInfo.pEngineName = "No Engine";  
        appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);  
        appInfo.apiVersion = VK_API_VERSION_1_0;
```

**创建VkInstance实例**时，指定需要使用的验证层（layer）和拓展(extension);如果不知道有哪些层(layer)或扩展可以使用，可以使用查询函数来枚举可用的层(layer)和扩展

创建VkInstance后，检测可用的GPU设备，返回VkPhysicalDevice类型的句柄。通过GPU设备的句柄，可以查询GPU设备属性并创建一个VkDevice

vkCreateInstance() → vkEnumeratePhysicalDevices() → vkCreateDevice()

#### 图像和缓冲

通过VkDevice创建VkImage和VkBuffer，指定其类型、储存形式、大小；VkImage通过VkImageView来访问，VkBuffer仅在着色器中需要VkBufferView

#### 分配GPU内存

缓冲和图像在创建后并没有实际为它们分配内存，需要手动分配

vkGetPhysicalDeviceMemoryProperties读取可分配内存信息，一般有系统内存和GPU内存

不同类型内存的性质不同，有的可被CPU直接访问，有的在CPU和GPU保持数据一致性，有的需要进行同步

内存分配需要调用vkAllocateMemory函数。调用它需要使用VkDevice和一个描述内存分配信息的结构体作为参数

#### 绑定内存

VkBuffer和VkImage的内存需求可以通过调用vkGetBufferMemoryRequirements函数和vkGetImageMemoryRequirements函数获取

通常，实践中由于内存分配的总次数有一定限制；我们可以一次分配一大块内存，然后将这一大块内存通过使用不同的偏移值分配给多个图像或缓冲使用。分配的偏移值需要满足图像或缓冲的对齐需求。是这样做来减少内存分配次数。

#### 指令缓冲和提交指令

#### VkImage

由像素组成，访问频繁，需要快速访问，尽量存储在GPU，可视作对于Buffer的进一步封装，增加特性

#### VkBuffer

在gup中分配的一小块内存空间，显卡访问GPU的速度，要比访问CPU的速度快很多；

①减少对cpu的访问次数，将数据一次性存放在buffer中，直接在buffer中读取

②buffer创建时，还存在cpu中，bind就是转移到具体buffer类型中

#### VkDevice:

physical device的逻辑形式，用来创建队列，管理内存，同步数据，xxxx生万物

#### extension

通俗理解，对Vulkan API的一种补充（结构体、函数等）

例如，Vulkan是平台无关的API，所以需要一个和窗口系统（glfw）交互的扩展

uint32_t glfwExtensionCount = 0;  
const char** glfwExtensions;  
​  
glfwExtensions = glfwGetRequiredInstanceExtensions(&glfwExtensionCount);  
​  
createInfo.enabledExtensionCount = glfwExtensionCount;  
createInfo.ppEnabledExtensionNames = glfwExtensions;

[What is a "Vulkan Extension"? - Computer Graphics Stack Exchange](https://computergraphics.stackexchange.com/questions/8170/what-is-a-vulkan-extension#:~:text=Vulkan extensions are simply additional features that Vulkan,implementation supports these extensions before creating an instance%2Fdevice.)

**KHR：Vulkan拓展

#### 验证层

Vulkan API的设计围绕最小化驱动程序开销，所以默认提供的错误检查非常有限；并且Vulkan需要用户大量的显示调用，容易出错

验证层是一个可选的，附加在Vulkan函数调用时，来进行附加操作，由Vulkan SDK中提供，主要功能如下：

- 检测参数值是否合法
    
- 追踪对象的创建和清除操作，发现资源泄漏问题
    
- 追踪调用来自的线程，检测是否线程安全。
    
- 将API调用和调用的参数写入日志
    
- 追踪API调用进行分析和回放
    

example：检查入参是否为空

VkResult vkCreateInstance(  
    const VkInstanceCreateInfo* pCreateInfo,  
    const VkAllocationCallbacks* pAllocator,  
    VkInstance* instance) {  
​  
    if (pCreateInfo == nullptr || instance == nullptr) {    //  
        log("Null pointer passed to required parameter!");  
        return VK_ERROR_INITIALIZATION_FAILED;  
    }  
​  
    return real_vkCreateInstance(pCreateInfo, pAllocator, instance);    //真正的实例化  
}

Vulkan可以使用两种不同类型的校验层：**实例校验层**和**设备校验层**。

实例校验层只检查和全局Vulkan对象相关的调用，比如Vulkan实例。

设备校验层只检查和特定GPU相关的调用。设备校验层现在已经不推荐使用

##### 验证层的使用

调试时开启，发布时关闭
```
#ifdef NDEBUG  
    const bool enableValidationLayers = false;  
#else  
    const bool enableValidationLayers = true;  
#endif

bool checkValidationLayerSupport() {  
    uint32_t layerCount;  
    vkEnumerateInstanceLayerProperties(&layerCount, nullptr);unt); //开辟属性储存空间  
    vkEnumerateInstanceLayerProperties(&layerCount, //读取验证层信息  
                    availableLayers.data());  
​  
    return false;  
}
```


![image-20220817165418747.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231719275.png)



剩下内容，后续补充

[Vulkan编程指南(章节6-校验层) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/56604991)

# Vulkan编程模型
![image-20220819160646115.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231732519.png)


![image-20220817154625736.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231731337.png)


## 硬件初始化

应用程序启动后，与加载器进行通讯，定位系统中的Vulkan驱动（**平台无关性**），加载驱动

![image-20220819161242216.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231732155.png)
当应用程序与API链接后：

![image-20220819161856681.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231733874.png)
## 窗口展示表面

Vulkan通过WSI（Windows system interesting）来创建窗口系统，这是跨平台的，WSI作为显示设备和应用程序之间的接口

![image-20220819162311808.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231733212.png)
引入**交换链**，使得显示和图像处理被分开；

应用程序处理第一幅图时，将第二幅图交给设备显示；处理完第一幅图时，交给图像显示，获取第二幅图去处理

![image-20220819162616674.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231736797.png)

![image-20220819162732631.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231736730.png)

## 资源设置

Vulkan中需要显示的分配管理内存

Vulkan中对GPU内存的分配次数是有限的，鼓励进行**suballocation**，物理内存的分配开销很大

因此，可以**先申请一大块物理内存（物理地址），创建资源对象后，给资源对象先分配一个逻辑地址，最后将逻辑地址绑定到物理内存上（物理地址+偏移量）**

![image-20220819164037382.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231736190.png)


tips:物理地址是硬件上实际的地址，应用程序无法直接访问；逻辑地址，是由应用程序分配的虚拟地址，程序可访问（背后发生的是绑定到物理地址）;类似，物理地址xx栋xx层，逻辑地址人为规定1号、2号、3号 与实际地址有一个对应关系
![image-20220819164107741.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231737960.png)


## 流水线设置

# Hello World伪代码

## 初始化

![image-20220819170019256.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231737545.png)


### 枚举layer和extension

Vulkan API的设计围绕最小化驱动程序开销，所以默认提供的错误检查非常有限；并且Vulkan需要用户大量的显示调用，容易出错

**验证层是一个可选的**（debug开启，release关闭），附加在Vulkan函数调用时，来进行附加操作，由Vulkan SDK中提供

目前，验证层已经不建议对设备进行验证，**更多的用于对创建的实例对象进行验证**

**extension**是Layer对VulkanAPI的补充

### VkInstance创建

VkInstance创建时，通过传入一个Info结构指定其属性，以及开启layer和extension

### VkDevice创建

枚举可用的物理设备，并创建VkDevice作为逻辑设备

VkDevice被用来**获取物理设备的信息，访问可用的队列族，申请分配物理设备内存**

## 交换链初始化

WSI创建本地窗口，创建VkSurface作为本地窗口的逻辑对象；根据surface查找队列

### Command buffer 初始化

根据surface查找到的图形队列句柄，来创建指令池，在指令池中分配command buffer

### 资源管理——VkImage&VkBuffer

## 着色器支持

通过vkCreateShaderModule()来创建着色器实例，将文本格式转化为Vuilkan可识别的SPIR-V

# 连接硬件设备

#### MyLayerAndExtension

获取instance的layer和 extension

获取device的layer和extension

#### MyInstance

createInstance：输入要开启的layer和extension，来实例化Instance

#### MyDevice

createDevice：根据gpu(物理设备)实例化Device

getPhysicalDeviceQueuesAndProperties：获取gpu上的队列族数量和属性

getGrahicsQueueHandle：获取图形队列在队列族中的索引

#### MyApplication

GetInstance：重新实例化对象，并覆盖原有对象，查找实例的layers和extensions

createVulkanInstance：为成员变量my_stance实例化instance

enumeratePhysicalDevices：枚举物理设备

handShakeWithDevice：传入gpu给MyDevice; 查找gpu的layers和extensions；获取Gpu属性；获取GPU内存属性；查找GPU支持队列；查找GPU图形队列索引；根据GPU信息，实例化相关device（属性、内存属性、队列信息、图形队列）

# Vulkan调试

## 借助SDK

**vkconfig.exe**

demo演示：[https://www.youtube.com/watch?v=KbL2vBzQ8Fc&feature=youtu.be](https://www.youtube.com/watch?v=KbL2vBzQ8Fc&feature=youtu.be)

文档说明：[Intro-to-Vulkan-Configurator_08_2020.pdf (lunarg.com)](https://www.lunarg.com/wp-content/uploads/2020/08/Intro-to-Vulkan-Configurator_08_2020.pdf)

具体使用

## 代码配置

bool debugFlag：debug时开启验证，release时关闭验证，提高性能

添加验证层名称

```
std::vector<const char *> instanceExtensionNames = {  
    VK_KHR_SURFACE_EXTENSION_NAME,  
    VK_KHR_WIN32_SURFACE_EXTENSION_NAME,  
    VK_EXT_DEBUG_REPORT_EXTENSION_NAME,  
};  
​  
std::vector<const char *> layerNames = {  
    "VK_LAYER_GOOGLE_threading",       
    "VK_LAYER_LUNARG_parameter_validation",  
    "VK_LAYER_LUNARG_object_tracker",  
    "VK_LAYER_LUNARG_image",           
    "VK_LAYER_LUNARG_core_validation",  
    "VK_LAYER_LUNARG_swapchain",  
    "VK_LAYER_GOOGLE_unique_objects",   
    "VK_LAYER_LUNARG_standard_validation"  
};
```


areLayersSupported：检测instance支持的layers

写入instance配置
```
   // Specify the list of layer name to be enabled.     //开启layers和extensions  
    instInfo.enabledLayerCount = layers.size();   
    instInfo.ppEnabledLayerNames   = layers.data();  
    instInfo.enabledExtensionCount = extensions.size();  
    instInfo.ppEnabledExtensionNames = extensions.data();  
​  
    VkResult res = vkCreateInstance(&instInfo, NULL, &instance);
```


vkGetInstanceProcAddr：查询API并进行动态连接，返回函数指针

createDebugReportCallback()：检索并创建、销毁函数指针

# 命令缓冲区以及内存管理

先创建缓冲池，再从缓冲池中分配缓冲命令：（减少多次申请缓冲的开销）
![image-20220824145607350.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231738137.png)


## 内存管理

- Host Local Memory，只对Host可见的内存，通常称之为普通内存（**CPU**）
    
- Device Local Memory，只对Device可见的内存，通常称之为显存（**GPU**）
    
- Host Local Device Memory，由Host管理的，对Device看见的内存
    
- Device Local Host Memory，由Device管理的，对Host可见的内存
    

通俗来讲，CPU容量更大，GPU对物理设备可见，访问更快

CPU内存管理：通过VkAllocationCallbacks来配置属性

### GPU内存管理：

VkGetPhysicalDeviceMemoryProperties：查询物理设备的内存类型、内存堆的属性和数量

vkAllocateMemory：分配GPU内存

vkFreeMemory：释放GPU内存

vkMapMemory：映射GPU内存，使其对cpu可见

vkUnMapMemory:结束映射，内容更新到GPU内存

# 图像

#### Image

存在GPU中的资源，主要有一下三种形式：

- 可写的 color attachment，打算将渲染的结果画到这个 Image 中。
    
- 可读的传统贴图，在 shader 中 配合 sampler 进行读取
    
- 可读可写，image load/store
    

#### image layout

image的储存形式，不同存储形式，特性不同；因此，**image需要在创建时，指定其用途**

**LINEAR** ：按照连续的方式存放，对人友好，对GPU不友好

**OPTIMAL**：让显存尽可能优化地管理 image，具体为一个黑盒

#### image view

image 是不能直接用的，需要通过 **ImageView**

### 图像创建流程

![image-20220824171749027.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231738230.png)


#### 创建 Image对象

根据Info信息，实例化逻辑对象

Info信息繁琐，需要时查表

#### 分配image内存

**计算内存需求:**

size:所需物理内存大小

alignment：访问时的偏移量（image地址 = 物理内存堆+alignment）

**memoryTypeBits**： ？？？看不懂

**申请分配GPU内存**

vkAlloactionMemory

**绑定内存**

![image-20220824174309495.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231739777.png)


#### 设置imagelayout

VkImageLayoyt枚举量来定义，Info

同步屏障

#### 创建ImageView:

# 交换链

图元处理机制：使得显示和图像处理被分开，较少渲染等待时间开销

应用程序处理第一幅图时，将第二幅图交给设备显示；处理完第一幅图时，交给图像显示，获取第二幅图去处理

图像的更新频率：①后台绘制完成，主动触发更新；②垂直刷新间隔VBI（vertical blanking interval）

交换链时VK拓展功能，需要开启**VK_KHR_SWAPCHAIN_EXTENSION_NAME**

工作流程：

交换链的颜色图像由其自己管理，无法应用ImageLayout

![image-20220825114758363.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231739801.png)


Application可以拥有多个渲染窗口

Render：管理一个渲染窗口，及其相关的渲染资源（设备、交换链、管线）

![image-20220825134401888.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306231739924.png)


# GPU架构：

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


  

# Hello World

![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306241653770.png)

  

## Instance

通过info来创建，用来控制vulkan的版本(可向前兼容)等信息

设置开启的layer和extension

  

## Layer & Extension

layer：用来调试、验证

extension：用来开启vulkan的扩展特性，包括RayTracing、SwapChain等

  

## PhysicalDevice

物理设备，往往指显卡，常用来查找或get一些信息

注：可通过枚举所有物理设备，选取符合相应特性的设备

## Device

逻辑设备，物理设备一般不与程序直接交互，通过对物理设备的映射，得到逻辑设备，用来和程序交互

  

## Queue

vulkan通过command来对绘制指令、内存申请进行==异步==执行，提交的对象就是命令族，图形、计算、内存拥有各自的命令族

  

## Surface & SwapChain & presentation

vulkan本身是与平台无关的，需要利用原生平台的窗口系统来显示渲染的内容，vulkan通过surfece抽象各平台的窗口系统，例如windows下，需要hwnd和hinstance句柄来创建surface

  

### Swap Chain(Mulit Buffer)

  

交换链或多缓冲

  

在单缓冲的情况下，画面时逐行逐像素的刷新，同一个缓冲一边显示一边用来处理数据，会导致最终画面出现**闪烁、撕裂**等

  

因此，采用双缓冲，缓冲A用来呈现画面，缓冲B用来后台处理数据，数据处理完成后，交换AB，使得始终有缓冲用来处理数据，不会停下来，同时保证呈现画面始终的处理完数据的画面，而不是处理一半的画面

  

In [computer science](https://en.wikipedia.org/wiki/Computer_science), **multiple buffering** is the use of more than one [buffer](https://en.wikipedia.org/wiki/Buffer_(computer_science)) to hold a block of data, so that a "reader" will see a complete (though perhaps old) version of the data, rather than a partially updated version of the data being created by a ["writer"](https://en.wikipedia.org/wiki/Readers-writers_problem). It is very commonly used for computer display images. It is also used to avoid the need to use [dual-ported RAM](https://en.wikipedia.org/wiki/Dual-ported_RAM) (DPRAM) when the readers and writers are different devices.

  

[多重缓冲 - 维基百科 (wikipedia.org)](https://en.wikipedia.org/wiki/Multiple_buffering)

  

### presentation

将绘制内容，呈现到surface

  

  

注：surface、swapchain都属于extension，需要在Instance中开启

  

  

## image & frame buffer

  

通过image view指定对swap chain输出内容的解析格式

  

注：输出内容不变，仅是按照不同格式去解析 RGBA，RGB？

  

  

frame buffer: pipeline最终输出到buffer

  

  

在gup中分配的一小块内存空间，显卡访问GPU的速度，要比访问CPU的速度快很多；

  

  

①减少对cpu的访问次数，将数据一次性存放在buffer中，直接在buffer中读取

  

  

②buffer创建时，还存在cpu中，bind就是转移到具体buffer类型中

  

  

## pipeline

  

光栅化Pipeline

  

RayTraceing Pipeline

  

  

TODO：加链接

  

# 概述

## 函数调用

函数仅返回bool或VkResult，用来判断函数的结果

get类函数，都是先在外申请变量，分配好内存，再作为参数传入函数来读取结果

这样做的好处是，所有函数的形式统一，不必返回类似(bool, result)这样的形式；且方便对函数的调用是否正确进行判断

  

## 属性查询

```cpp

uint32_t layerCount;

vkEnumerateInstanceLayerProperties(&layerCount, nullptr); //查询可用数量

  

std::vector<VkLayerProperties> availableLayers(layerCount); //根据数量分配内存

vkEnumerateInstanceLayerProperties(&layerCount,availableLayers.data()); //查询属性

```

  
  

# Set up

## Instance

instance 通过createInstance(createInfo info)创建

  

info中指定了开启的layer、extension、appInfo(版本信息)

  

![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306261037456.png)

  
  

## Layers

  

![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306261055753.png)

[Overview of Vulkan Loader and Layers - LunarG](https://www.lunarg.com/tutorial-overview-of-vulkan-loader-layers/#:~:text=Layers%20are%20optional%20components%20that%20augment%20the%20Vulkan,by%20application%20request%29%20and%20are%20loaded%20during%20CreateInstance.)

  

![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306261100731.png)

  

- layers被插入在loader中间，用来拦截，修改API

- layers作为可选项，debug开启，release关闭，==Vulkan API的设计是紧紧围绕最小化驱动程序开销进行的==

- layers常用功能：

校验API

debug、log

覆盖API调用

  

## Extension

用来开启vulkan的扩展特性API，包括RayTracing、SwapChain等

  
  

## Validation layers

验证层工作示例：

```cpp

VkResult  vkCreateInstance(

const  VkInstanceCreateInfo*  pCreateInfo,

const  VkAllocationCallbacks*  pAllocator,

VkInstance*  instance)

{

  

if (pCreateInfo ==  nullptr  || instance ==  nullptr) {

log("Null pointer passed to required parameter!");

return VK_ERROR_INITIALIZATION_FAILED;

}

  

return  real_vkCreateInstance(pCreateInfo, pAllocator, instance);

}

```

  

- checkValidationLayerSupport：检查所有需要开启的layer是否支持

- getRequiredExtensions()：获取需要开启的extension，glfw、验证层layer等

  

VkDebugUtilsMessengerCreateInfoEXT：用来打印报错信息

```cpp

VkDebugUtilsMessengerCreateInfoEXT createInfo = {};

createInfo.sType  = VK_STRUCTURE_TYPE_DEBUG_UTILS_MESSENGER_CREATE_INFO_EXT;

createInfo.messageSeverity  = VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT |

VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT |

VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT;

createInfo.messageType  = VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT |

VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT |

VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT;

createInfo.pfnUserCallback  = debugCallback;

createInfo.pUserData  =  nullptr; // Optional

```

- sType：指定消息级别，用来过滤一些消息

- VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT：诊断信息

- VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT：资源创建之类的信息

- VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT：警告信息

- VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT：不合法和可能造成崩溃的操作信息

  

- messageSeverity：指定消息的错误级别

- VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT：发生了一些与规范和性能无关的事件

- VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT：出现了违反规范的情况或发生了一个可能的错误

- VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT：进行了可能影响Vulkan性能的行为

  

- pfnUserCallback：指向VkDebugUtilsMessengerCallbackDataEXT结构体的指针，包含错误信息字符串等

  

- pUserData：传入的回调函数，返回值用来控制发生错误时，是否终止运行

```cpp

static VKAPI_ATTR VkBool32 VKAPI_CALL debugCallback(

VkDebugUtilsMessageSeverityFlagBitsEXT messageSeverity,

VkDebugUtilsMessageTypeFlagsEXT messageType,

const VkDebugUtilsMessengerCallbackDataEXT* pCallbackData,

void* pUserData) {

  

std::cerr <<  "validation layer: "  <<  pCallbackData->pMessage

<< std::endl;

  

return VK_FALSE;

}

```

  

注：具体代码有点复杂，大致自洽，使用为主

  

## PhysicalDevice

物理设备仅用于查找属性，包含多个队列族

  

pickPhysicalDevice：获取第一个可用的GPU

isDeviceSuitable：检查物理设备是否满足属性和特性需求

vkGetPhysicalDeviceProperties：设备属性

vkGetPhysicalDeviceFeatures：功能特性

  

## QueueFamily

command会被提交到queue中，queue会按照类型存放在相应的队列族中

  

findQueueFamilies：查找队列族，找到第一个合适的队列族并返回其==索引==

ps：此处队列族列表好像没有被保存在本地，之后传队列族索引到vulkan？

  

## Device

物理设备仅用于查找属性，需要通过逻辑设备来与物理设备进行交互

用多个逻辑设备来映射同一个GPU，使得每个逻辑设备仅包含一类队列

VkDeviceCreateInfo

sType: VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO

pQueueCreateInfos: 队列Info列表

queueCreateInfoCount: 队列数量

pEnabledFeatures：

enabledExtensionCount：扩展数量

  

## Queue

队列属于逻辑设备，用于应用层和物理设备间的通讯

应用层通过提交command到队列，物理设备读取队列并异步执行

  

队列的创建，隐式的创建在逻辑设备的创建过程中

![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306271144725.png)

ps: queuePriority是一个0到1之间的浮点数，用来控制命令的优先级

  

vkGetDeviceQueue：从device中获取queque句柄

  
  
  

# Presentation

## surface

vulkan本身是与平台无关的，需要利用原生平台的窗口系统 ==WSI== 来显示渲染的内容，vulkan通过surfece抽象各平台的窗口系统，例如windows下，需要hwnd和hinstance句柄来创建surface

  

教程中通过glfwCreateWindowSurface来创建surface

  

surfaceKHR属于extension，需要在物理设备中进行检查

graphic：queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT

persentation：vkGetPhysicalDeviceSurfaceSupportKHR

  

并且，绘制的队列族和支持呈现的队列族，有时不重叠，因此需要单独维护一个支持呈现的队列族，和graphic 队列族存放在一个列表中，用于创建逻辑设备

  

## swap chain

在单缓冲的情况下，画面时逐行逐像素的刷新，同一个缓冲一边显示一边用来处理数据，会导致最终画面出现**闪烁、撕裂**等

  

因此，采用双缓冲，缓冲A用来呈现画面，缓冲B用来后台处理数据，数据处理完成后，交换AB，使得始终有缓冲用来处理数据，不会停下来，同时保证呈现画面始终的处理完数据的画面，而不是处理一半的画面

[多重缓冲 - 维基百科 (wikipedia.org)](https://en.wikipedia.org/wiki/Multiple_buffering)

  

交换链的本质上就是包含了若干等待呈现的图像的队列，应用层通过交换链获取图像用于渲染数据，再将图像丢回队列

  

### Checking for swap chain support

交换链的支持，需要在extension中开启VK_KHR_swapchain

在 isDeviceSuitable中调用checkDeviceExtensionSupport，来检查物理设备是否支持该扩展

之后在物理设备的创建参数中，加入扩展信息

  

### Querying details of swap chain support

- 表面格式(像素格式，颜色空间)

- 基础表面特性(交换链的最小/最大图像数量，最小/最大图像宽度、高度)

- 可用的呈现模式

  

```cpp

struct  SwapChainSupportDetails {

VkSurfaceCapabilitiesKHR capabilities;

std::vector<VkSurfaceFormatKHR> formats;

std::vector<VkPresentModeKHR> presentModes;

};

```

vkGetPhysicalDeviceSurfaceCapabilitiesKHR

vkGetPhysicalDeviceSurfaceFormatsKHR

vkGetPhysicalDeviceSurfacePresentModesKHR

查询对应属性

  

### Choosing the right settings for the swap chain

设置交换链的属性，如果不支持，则设置一个其他属性

  

#### Surface format

format：像素格式

VK_FORMAT_B8G8R8A8_UNORM：BGRA，四通道，都是u8

.....

color space：颜色空间

VK_COLOR_SPACE_SRGB_NONLINEAR_KHR：标志是否支持SRGB

  

chooseSwapSurfaceFormat：用来查找是否支持期望的format属性，不支持的话 则设置formats[0]

  

#### Presentation mode

呈现模式用来指定什么情况下，图像被呈现到屏幕

  

VK_PRESENT_MODE_IMMEDIATE_KHR：应用程序提交的图像会被立即传输到屏幕上，可能会导致撕裂现象

VK_PRESENT_MODE_FIFO_KHR：先进先出，队列头用于呈现，当队列满时，应用层阻塞

VK_PRESENT_MODE_FIFO_RELAXED_KHR：先进先出，但是如果应用程序延迟，导致交换链的队列在上一次垂直回扫时为空，那么，如果应用程序在下一次垂直回扫前提交图像，图像会立即被显示。这一模式可能会导致撕裂现象

VK_PRESENT_MODE_MAILBOX_KHR：先进先出，但是队列满时，不会阻塞应用层，直接替换队列中的图像

  

chooseSwapPresentMode：按优先级选取呈现模式，3->4->1->2

  

#### Swap extent

对于部分窗口系统，会使用特殊值，例如u32::MAX，此时表示是否支持对当前的分辨率属性进行设置

  

chooseSwapExtent：

检测系统是否支持自定义当前分辨率

不支持：直接返回

支持自定义：

初始分辨率为glfw创建时的窗口屏幕分辨率

对初始分辨率在swap chain支持的分辨率min、max中，做clomp

  
  

### Creating the swap chain

查询swap chain属性

选择swap chain属性

配置 creaetInfo，用到上述选择的属性，此外还有一些简单的属性配置

  

如果graphic队列族和persentation队列族不是同一个，此时，**images的计算和呈现在不同队列族中**

在Info中指定imageSharingMode，性能开销明显

- VK_SHARING_MODE_EXCLUSIVE：一张图像同一时间只能被一个队列族所拥有，在另一队列族使用它之前，必须显式地改变图像所有权。这一模式下性能表现最佳。

- VK_SHARING_MODE_CONCURRENT：图像可以在多个队列族间使用，不需要显式地改变图像所有权

preTransform：对呈现的image进行transform

  

注：swap chain创建参数过多，以后需要再查手册

  

### Retrieving the swap chain images

```cpp

std::vector<VkImage> swapChainImages;

```

vkGetSwapchainImagesKHR：获取swap chain的Images

  

## Image views