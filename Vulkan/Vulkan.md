[Vulkan Tutorial](https://vulkan-tutorial.com/)  
[VulkanTutorial/code](https://github.com/Overv/VulkanTutorial/tree/main/code)  
[Vulkan Tutorial (Rust)：最后有新内容补充](https://kylemayes.github.io/vulkanalia/overview.html)  
[Vulkan Tutorial-CN](https://zhuanlan.zhihu.com/p/56338417)

[Vulkan笔记：深度+细致](https://zhuanlan.zhihu.com/p/616082929)

Vulkan学习指南.PDF:详细到每个参数

[Vulkan: Examples：进阶](https://github.com/SaschaWillems/Vulkan)


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

![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306261100731.png)  
==instance中存放了所有对象的状态==

instance 通过createInstance(createInfo info)创建  
info中指定了开启的layer、extension、appInfo(版本信息)  
![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306261037456.png)

## Layers

![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306261055753.png)

[Overview of Vulkan Loader and Layers - LunarG](https://www.lunarg.com/tutorial-overview-of-vulkan-loader-layers/#:~:text=Layers%20are%20optional%20components%20that%20augment%20the%20Vulkan,by%20application%20request%29%20and%20are%20loaded%20during%20CreateInstance.)

- layers被插入在loader中间，用来拦截，修改API
    
- layers作为可选项，debug开启，release关闭，==Vulkan API的设计是紧紧围绕最小化驱动程序开销进行的==
    
- layers常用功能：
    

校验API

debug、log

覆盖API调用

## Extension

用来开启vulkan的扩展特性API，包括RayTracing、SwapChain等

## Validation layers

- 根据规范检查参数数值，最终确认是否存与预期不符的情况
- 跟踪对象的创建和销毁，以查找是否存在资源的泄漏
- 跟踪线程的调用链，确认线程执行过程中的安全性
- 将每次函数调用所使用的参数记录到标准的输出中，进行初步的Vulkan概要分析

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

vkCreateDebugUtilsMessengerEXT是拓展函数，需要在instance中检查

```cpp
// 需要通过vkCreateDebugUtilsMessengerEXT来创建，但是这个函数又是个扩展函数。必须检查一下
VkResult CreateDebugUtilsMessengerEXT(VkInstance instance, const VkDebugUtilsMessengerCreateInfoEXT* pCreateInfo, const VkAllocationCallbacks* pAllocator, VkDebugUtilsMessengerEXT* pDebugMessenger) {
    // 可通过vkGetInstanceProcAddr来查找是否有这个函数。
    auto func = (PFN_vkCreateDebugUtilsMessengerEXT) vkGetInstanceProcAddr(instance, "vkCreateDebugUtilsMessengerEXT");
    if (func != nullptr) {
        return func(instance, pCreateInfo, pAllocator, pDebugMessenger);
    } else {
        return VK_ERROR_EXTENSION_NOT_PRESENT;
    }
}
```

注：具体代码有点复杂，大致自洽，使用为主

## PhysicalDevice

GPU用于查找属性，包含多个队列族

pickPhysicalDevice：获取第一个可用的GPU

isDeviceSuitable：检查物理设备是否满足属性和特性需求

vkGetPhysicalDeviceProperties：设备属性

vkGetPhysicalDeviceFeatures：功能特性

## QueueFamily

command会被提交到queue中，queue会按照类型存放在相应的队列族中

findQueueFamilies：查找队列族，找到第一个合适的队列族并返回其==索引==

ps：此处队列族列表没有被保存在本地，之后传队列族索引到vulkan

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

# PresentationSet up
Instance
image.png
instance中存放了所有对象的状态

instance 通过createInstance(createInfo info)创建
info中指定了开启的layer、extension、appInfo(版本信息)
image.png

Layers
image.png

Overview of Vulkan Loader and Layers - LunarG

layers被插入在loader中间，用来拦截，修改API

layers作为可选项，debug开启，release关闭，Vulkan API的设计是紧紧围绕最小化驱动程序开销进行的

layers常用功能：

校验API

debug、log

覆盖API调用

Extension
用来开启vulkan的扩展特性API，包括RayTracing、SwapChain等

Validation layers
根据规范检查参数数值，最终确认是否存与预期不符的情况
跟踪对象的创建和销毁，以查找是否存在资源的泄漏
跟踪线程的调用链，确认线程执行过程中的安全性
将每次函数调用所使用的参数记录到标准的输出中，进行初步的Vulkan概要分析
验证层工作示例：


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

checkValidationLayerSupport：检查所有需要开启的layer是否支持

getRequiredExtensions()：获取需要开启的extension，glfw、验证层layer等

VkDebugUtilsMessengerCreateInfoEXT：用来打印报错信息


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

sType：指定消息级别，用来过滤一些消息

VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT：诊断信息

VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT：资源创建之类的信息

VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT：警告信息

VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT：不合法和可能造成崩溃的操作信息

messageSeverity：指定消息的错误级别

VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT：发生了一些与规范和性能无关的事件

VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT：出现了违反规范的情况或发生了一个可能的错误

VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT：进行了可能影响Vulkan性能的行为

pfnUserCallback：指向VkDebugUtilsMessengerCallbackDataEXT结构体的指针，包含错误信息字符串等

pUserData：传入的回调函数，返回值用来控制发生错误时，是否终止运行


static VKAPI_ATTR VkBool32 VKAPI_CALL debugCallback(

VkDebugUtilsMessageSeverityFlagBitsEXT messageSeverity,

VkDebugUtilsMessageTypeFlagsEXT messageType,

const VkDebugUtilsMessengerCallbackDataEXT* pCallbackData,

void* pUserData) {

  

std::cerr <<  "validation layer: "  <<  pCallbackData->pMessage

<< std::endl;

  

return VK_FALSE;

}

vkCreateDebugUtilsMessengerEXT是拓展函数，需要在instance中检查

// 需要通过vkCreateDebugUtilsMessengerEXT来创建，但是这个函数又是个扩展函数。必须检查一下
VkResult CreateDebugUtilsMessengerEXT(VkInstance instance, const VkDebugUtilsMessengerCreateInfoEXT* pCreateInfo, const VkAllocationCallbacks* pAllocator, VkDebugUtilsMessengerEXT* pDebugMessenger) {
    // 可通过vkGetInstanceProcAddr来查找是否有这个函数。
    auto func = (PFN_vkCreateDebugUtilsMessengerEXT) vkGetInstanceProcAddr(instance, "vkCreateDebugUtilsMessengerEXT");
    if (func != nullptr) {
        return func(instance, pCreateInfo, pAllocator, pDebugMessenger);
    } else {
        return VK_ERROR_EXTENSION_NOT_PRESENT;
    }
}
注：具体代码有点复杂，大致自洽，使用为主

PhysicalDevice
GPU用于查找属性，包含多个队列族

pickPhysicalDevice：获取第一个可用的GPU

isDeviceSuitable：检查物理设备是否满足属性和特性需求

vkGetPhysicalDeviceProperties：设备属性

vkGetPhysicalDeviceFeatures：功能特性

QueueFamily
command会被提交到queue中，queue会按照类型存放在相应的队列族中

findQueueFamilies：查找队列族，找到第一个合适的队列族并返回其索引

ps：此处队列族列表没有被保存在本地，之后传队列族索引到vulkan

Device
物理设备仅用于查找属性，需要通过逻辑设备来与物理设备进行交互

用多个逻辑设备来映射同一个GPU，使得每个逻辑设备仅包含一类队列

VkDeviceCreateInfo

sType: VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO

pQueueCreateInfos: 队列Info列表

queueCreateInfoCount: 队列数量

pEnabledFeatures：

enabledExtensionCount：扩展数量

Queue
队列属于逻辑设备，用于应用层和物理设备间的通讯

应用层通过提交command到队列，物理设备读取队列并异步执行

队列的创建，隐式的创建在逻辑设备的创建过程中

image.png

ps: queuePriority是一个0到1之间的浮点数，用来控制命令的优先级

vkGetDeviceQueue：从device中获取queque句柄

Presentation
surface
vulkan本身是与平台无关的，需要利用原生平台的窗口系统 WSI 来显示渲染的内容，vulkan通过surfece抽象各平台的窗口系统，例如windows下，需要hwnd和hinstance句柄来创建surface

教程中通过glfwCreateWindowSurface来创建surface

surfaceKHR属于extension，需要在物理设备中进行检查

graphic：queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT
persentation：vkGetPhysicalDeviceSurfaceSupportKHR

并且，绘制的队列族和支持呈现的队列族，有时不重叠，因此需要单独维护一个支持呈现的队列族，和graphic 队列族存放在一个列表中，用于创建逻辑设备

swap chain
在单缓冲的情况下，画面时逐行逐像素的刷新，同一个缓冲一边显示一边用来处理数据，会导致最终画面出现闪烁、撕裂等

因此，采用双缓冲，缓冲A用来呈现画面，缓冲B用来后台处理数据，数据处理完成后，交换AB，使得始终有缓冲用来处理数据，不会停下来，同时保证呈现画面始终的处理完数据的画面，而不是处理一半的画面

多重缓冲 - 维基百科 (wikipedia.org)

交换链的本质上就是包含了若干等待呈现的图像的队列，应用层通过交换链获取图像用于渲染数据，再将图像丢回队列

Checking for swap chain support
交换链的支持，需要在extension中开启VK_KHR_swapchain

在 isDeviceSuitable中调用checkDeviceExtensionSupport，来检查物理设备是否支持该扩展

之后在物理设备的创建参数中，加入扩展信息

Querying details of swap chain support
表面格式(像素格式，颜色空间)

基础表面特性(交换链的最小/最大图像数量，最小/最大图像宽度、高度)

可用的呈现模式


struct  SwapChainSupportDetails {

VkSurfaceCapabilitiesKHR capabilities;

std::vector<VkSurfaceFormatKHR> formats;

std::vector<VkPresentModeKHR> presentModes;

};

vkGetPhysicalDeviceSurfaceCapabilitiesKHR

vkGetPhysicalDeviceSurfaceFormatsKHR

vkGetPhysicalDeviceSurfacePresentModesKHR

查询对应属性

Choosing the right settings for the swap chain
设置交换链的属性，如果不支持，则设置一个其他属性

Surface format
format：像素格式

VK_FORMAT_B8G8R8A8_UNORM：BGRA，四通道，都是u8

…

color space：颜色空间

VK_COLOR_SPACE_SRGB_NONLINEAR_KHR：标志是否支持SRGB

chooseSwapSurfaceFormat：用来查找是否支持期望的format属性，不支持的话 则设置formats[0]

Presentation mode
呈现模式用来指定什么情况下，图像被呈现到屏幕

VK_PRESENT_MODE_IMMEDIATE_KHR：应用程序提交的图像会被立即传输到屏幕上，可能会导致撕裂现象

VK_PRESENT_MODE_FIFO_KHR：先进先出，队列头用于呈现，当队列满时，应用层阻塞

VK_PRESENT_MODE_FIFO_RELAXED_KHR：先进先出，但是如果应用程序延迟，导致交换链的队列在上一次垂直回扫时为空，那么，如果应用程序在下一次垂直回扫前提交图像，图像会立即被显示。这一模式可能会导致撕裂现象

VK_PRESENT_MODE_MAILBOX_KHR：先进先出，但是队列满时，不会阻塞应用层，直接替换队列中的图像

chooseSwapPresentMode：按优先级选取呈现模式，优先4 三重缓冲

Swap extent
对于部分窗口系统，会使用特殊值，例如u32::MAX，此时表示是否支持对当前的分辨率属性进行设置

chooseSwapExtent：

检测系统是否支持自定义当前分辨率

不支持：直接返回

支持自定义：

初始分辨率为glfw创建时的窗口屏幕分辨率

对初始分辨率在swap chain支持的分辨率min、max中，做clomp

Creating the swap chain
查询swap chain属性

选择swap chain属性

配置 creaetInfo，用到上述选择的属性，此外还有一些简单的属性配置

如果graphic队列族和persentation队列族不是同一个，此时，images的计算和呈现在不同队列族中

在Info中指定imageSharingMode，性能开销明显

VK_SHARING_MODE_EXCLUSIVE：一张图像同一时间只能被一个队列族所拥有，在另一队列族使用它之前，必须显式地改变图像所有权。这一模式下性能表现最佳。

VK_SHARING_MODE_CONCURRENT：图像可以在多个队列族间使用，不需要显式地改变图像所有权

preTransform：对呈现的image进行transform

注：swap chain创建参数过多，以后需要再查手册

Retrieving the swap chain images

std::vector<VkImage> swapChainImages;

vkGetSwapchainImagesKHR：获取swap chain的Images

Image views
VkImage存放在GPU中，应用层无法直接访问，需要通过ImageView来绑定VkImage，其作为VkImage对外的句柄，描述了访问图像的方式

createInfo:
image.png

Graphic Pipeline Basic
PSO：pipeline state object
PCO: pipeline cache object
Pipeline Layout
Introduction
TODO: link to pipeline

注：此处的pipeline，相较于管线，流水线更加贴切，并且只是流水线模块的状态设置

Shader modules
GL Shader Language（GLSL）基础语法

SPIR-V & GLSL
GLSL是类C代码，写起来比较流畅
SPIR-V是Vulkan支持的二进制代码
SPIR-V更加高效，且不同GPU厂商和平台解析起来也一致，但是不会有人直接去写二进制代码
先写GLSL，再通过vulkan支持的glslangValidator.exe转成SPIR-V

Vertex shader
顶点着色器对每个传入的顶点进行处理，输入坐标和顶点属性，输出裁剪坐标和顶点属性
输出的裁剪坐标，进行透视除法后，得到NDC坐标
顶点属性包括：颜色、纹理坐标。ps：插值也发生在这个阶段

#version 450 
#extension GL_ARB_separate_shader_objects : enable 

out gl_PerVertex { vec4 gl_Position; }; 
vec2 positions[3] = vec2[]( vec2(0.0, -0.5), vec2(0.5, 0.5), vec2(-0.5, 0.5) ); 

void main() { 
	gl_Position = vec4(positions[gl_VertexIndex], 0.0, 1.0); 
}
其中，需要添加extension支持，有的变量为内建变量
in：输入
out：输出
out gl_PerVertex { vec4 gl_Position; }; 顶点着色器的固定输出格式
gl_Position：内建变量，顶点坐标
gl_VertexIndex：内建变量，顶点索引，一般用于在顶点缓冲中读取数据

Fragment shader
#version 450 
#extension GL_ARB_separate_shader_objects : enable 

layout(location = 0) out vec4 outColor; 
void main() { 
	outColor = vec4(1.0, 0.0, 0.0, 1.0);
}
Color interpolation
对顶点颜色进行插值

vertex shader:

#version 450 
#extension GL_ARB_separate_shader_objects : enable 

layout(location = 0) out vec3 fragColor;

out gl_PerVertex { vec4 gl_Position; }; 
vec2 positions[3] = vec2[]( vec2(0.0, -0.5), vec2(0.5, 0.5), vec2(-0.5, 0.5) ); 
vec3 color[3] = vec3[](vec3(1.0, 0.0, 0.0),vec3(0.0, 1.0, 0.0), vec3(0.0, 0.0, 1.0));
void main() { 
	gl_Position = vec4(positions[gl_VertexIndex], 0.0, 1.0); 
	fragColor = colors[gl_VertexIndex];
}
fragment shader:

#version 450 
#extension GL_ARB_separate_shader_objects : enable 

layout(location = 0) in vec3 fragColor;
layout(location = 0) in vec4 outColor;

void main() { 
	outColor = vec4(fragColor, 1.0);
}
对于输入变量和输出变量，通过location来关联，与变量名无关

Create shader
compile
创建shader文件夹，shader.vert存放顶点着色器GLSL代码，shader.frag存放片段着色器GLSL代码，后缀名无影响，习惯命名

创建脚本文件，利用glslangValidator.exe将GLSL转化为SPIR-V

/home/user/VulkanSDK/x.x.x.x/x86_64/bin/glslangValidator -V shader.vert
/home/user/VulkanSDK/x.x.x.x/x86_64/bin/glslangValidator -V shader.frag
pause
得到vert.spv，frag.spv存放SPIR-V代码

load
从vert.spv，frag.spv读取数据，存放在vector< char >中

#include <fstream>

static std::vector<char> readFile(const std::string& filename) {
    std::ifstream file(filename, std::ios::ate | std::ios::binary);

    if (!file.is_open()) {
        throw std::runtime_error("failed to open file!");
    }
}
ps：ios:: ate从尾部读取，可以预先申请好空间，不用动态扩容；ios:: binary 按二进制读取

create
根据加载的code 封装 VkShaderModule
image.png
ps：code记得转为u32

对封装的VkShaderModule指定着色器阶段

VkPipelineShaderStageCreateInfo fragShaderStageInfo = {};
fragShaderStageInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
fragShaderStageInfo.stage = VK_SHADER_STAGE_FRAGMENT_BIT;
fragShaderStageInfo.module = fragShaderModule;
fragShaderStageInfo.pName = "main";
info下一节再用

Fixed function
Vulkan中需要对所有状态都显示的给出，包括默认状态
下面的状态信息，都被用来createPipeline

Vertex input
顶点输入
描述传递给顶点着色器的顶点数据格式

Input assembly
图元装配
描述几何图元信息，以及是否复用图元

Viewports and scissors
视口：图像如何映射到帧缓冲
裁剪：裁剪矩形外的像素都会被过滤

Rasterizer
光栅化：多边形的填充、面的朝向剔除

rasterizerDiscardEnable字段如果设置为VK_TRUE，那么将不会有任何几何体能够通过光栅化阶段。
depthClampEnable字段被设置为VK_TRUE，那么超出近平面和远平面的片段就会被夹在其中，而不是丢弃。在生成ShadowMap的过程会被用到。
polygonMode字段决定了如何为几何体填充像素区域。如果需要线框渲染可以通过将polygonMode设为VK_POLYGON_MODE_LINE来实现。
cullMode字段决定了面剔除的配置，可以禁用或者背面剔除或者前面剔除或两者兼用。frontFace变量规定了被视为正面的顶点顺序，可以是顺时针或逆时针。这将影响面剔除的结果。
depthBiasEnable字段决定是否开启深度偏置，
depthBiasConstantFactor是一个常量，控制添加到每个片段的的固定深度值。
depthBiasClamp是一个片段的最大(或最小)深度偏差。depthBiasSlopeFactor是一个常量，在深度偏差计算中应用于片段的斜率。这个将会在软阴影的实现中用到
Multisampling
用于减少走样，比起提高分辨率再采样，开销更少
按下不表

Depth and stencil testing
depthTestEnable字段指定是否应将新片段的深度与Depth Buffer进行比较，看它们是否应被丢弃。
depthWriteEnable字段指定是否应将通过深度测试的新片段的深度实际写入Depth Buffer。
depthCompareOp字段指定了为保留或丢弃片段所进行的如何去比较。一般来说较低的深度=较近的惯例，所以新片段的深度应该较小。
depthBoundsTestEnable字段用于可选的深度边界测试是否开启。基本上，这允许你只保留落在指定深度范围内的片段。
minDepthBounds字段用于深度边界测试的最小深度。
maxDepthBounds字段用于深度边界测试的最大深度。
stencilTestEnable字段指定是否开启Stencil Test。
fron和back字段指定关于StencilTest的配置。
Color blending
current和last 混合
current和常量混合

Dynamic state
pipeline的创建，过于复杂，如果只想更改pipeline的部分属性，直接重建时不合适的
但是，只有非常有限的管线状态可以在不重建管线的情况下进行动态修改。这包括视口大小，线宽和混合常量

Pipeline Layout
我们可以在着色器中使用uniform变量，它可以在管线建立后动态地被应用程序修改，实现对着色器进行一定程度的动态配置。
uniform变量经常被用来传递变换矩阵给顶点着色器，以及传递纹理采样器句柄给片段着色器。

参考资料
理解Vulkan管线(Pipeline) - 知乎 (zhihu.com)

Render Pass
Attachment
描述存放color、depth等
ps：和buffer有什么区别？？？
buffer和image是真正存放数据的地方，attachment仅仅是定义数据的格式

SubPass
每个subPass绑定一条流水线，renderPass可以有多个subPass，subPass的输出可以作为下个subPass的输入

Pipeline Create
pipeline的创建，需要指定着色器，指定pipeline state，指定pipeline layout, 绑定renderPass和subPass的索引

Drawing
Framebuffer
真正存放VkImage的地方，用于屏幕显示
需要绑定renderPass、attachement

Command buffer
command而不是函数调用的好处在于，可以收集命令，一次性全部提交到队列中，方便GPU进行并行处理优化

Command Pool
命令池，管理buffer，从pool中分配内存
commadnPool需要指定队列族

Commad Create
帧缓冲和comand buffer一一对应，并从commandPool中分配内存
vkResetCommandPool：重置commandBuffer，commandPool回收这部分内存

Command state
起始状态，命令缓冲区刚被分配或重置时就是这个状态
记录状态，vkBeginCommandBuffer开始录制，命令将被记录在命令缓冲区中
可执行状态，vkEndCommandBuffer结束录制，命令缓冲区可以被提交
待定状态，命令缓冲区被提交到队列中会使它处于这个状态。提交后无法再更改命令，状态结束后，恢复到可执行状态
不可用状态，在这个状态下，命令缓冲区只能被重置或释放。
Rendering and persentation
drawFrame
从交换链中获取一个图像
记录command buffer，绘制到该图像上
提交command
present返回swap chain的图形

Synchronization
不同于OGL对外伪装的同步，所有GPU的命令都是异步执行的，OGL对伪装的同步，底下驱动在负重前行

Vulkan在cpu端调用的函数或提交的命令都是立刻执行，返回参数
但是，函数调用或命令提交在GPU没有被立刻执行，并且其是异步的
需要进行同步操作，人为控制命令的顺序

Fence
fence用来在CPU和GPU间进行同步，主要是针对不同帧的同步问题
例如：截图功能的执行，需要CPU等待GPU渲染完上一帧后，再执行截图

VkCommandBuffer A = ... // record command buffer with the transfer
VkFence F = ... // create the fence

// enqueue A, start work immediately, signal F when done
vkQueueSubmit(work: A, fence: F)

vkWaitForFence(F) // blocks execution until A has finished executing

save_screenshot_to_disk() // can't run until the transfer has finished
ps: fence会阻塞CPU线程，谨慎使用

Semaphores
semaphores用来控制GPU的顺序

伪代码：命令B等待命令A执行完，再执行

VkCommandBuffer A, B = ... // record command buffers
VkSemaphore S = ... // create a semaphore

// enqueue A, signal S when done - starts executing immediately
vkQueueSubmit(work: A, signal: S, wait: None)

// enqueue B, wait on S to start
vkQueueSubmit(work: B, signal: None, wait: S)
Waiting for the previous frame
通过fence，在drawFrame中，等待上一帧执行完，再渲染下一帧

Acquiring an image from the swap chain
vkAcquireNextImageKHR：从交换链中获取image, 并通过索引imageIndex指定帧缓冲

Recording the command buffer
录制命令

Submitting the command buffer
提交命令，并指定同步信号量

Subpass dependencies
Presentation
渲染结果提交到交换链，呈现到屏幕

Frame in fight
drawFrame中，阻塞了线程，等待上一帧的绘制完成

此处，开启了异步渲染

Swap chain recreation
当window窗口size发成变化时，会和交换链不兼容，需要监听窗口变化事件，重建交换链

按下不表

Vertex buffers
Uniform buffers
Texture mapping
Model
Rendering quality
Dynamic scenes
Push constants - Vulkan Tutorial (Rust) (kylemayes.github.io)

## surface

vulkan本身是与平台无关的，需要利用原生平台的窗口系统 ==WSI== 来显示渲染的内容，vulkan通过surfece抽象各平台的窗口系统，例如windows下，需要hwnd和hinstance句柄来创建surface

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

…

color space：颜色空间

VK_COLOR_SPACE_SRGB_NONLINEAR_KHR：标志是否支持SRGB

chooseSwapSurfaceFormat：用来查找是否支持期望的format属性，不支持的话 则设置formats[0]

#### Presentation mode

呈现模式用来指定什么情况下，图像被呈现到屏幕

VK_PRESENT_MODE_IMMEDIATE_KHR：应用程序提交的图像会被立即传输到屏幕上，可能会导致撕裂现象

VK_PRESENT_MODE_FIFO_KHR：先进先出，队列头用于呈现，当队列满时，应用层阻塞

VK_PRESENT_MODE_FIFO_RELAXED_KHR：先进先出，但是如果应用程序延迟，导致交换链的队列在上一次垂直回扫时为空，那么，如果应用程序在下一次垂直回扫前提交图像，图像会立即被显示。这一模式可能会导致撕裂现象

VK_PRESENT_MODE_MAILBOX_KHR：先进先出，但是队列满时，不会阻塞应用层，直接替换队列中的图像

chooseSwapPresentMode：按优先级选取呈现模式，优先4 三重缓冲

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

VkImage存放在GPU中，应用层无法直接访问，需要通过ImageView来绑定VkImage，其作为VkImage对外的句柄，描述了访问图像的方式

createInfo:  
![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306291032208.png)

# Graphic Pipeline Basic

```
PSO：pipeline state object
PCO: pipeline cache object
Pipeline Layout
```

## Introduction

TODO: link to pipeline

==注：此处的pipeline，相较于管线，流水线更加贴切，并且只是流水线模块的状态设置==

## Shader modules

[GL Shader Language（GLSL）基础语法](https://zhuanlan.zhihu.com/p/349296191)

### SPIR-V & GLSL

GLSL是类C代码，写起来比较流畅  
SPIR-V是Vulkan支持的二进制代码  
SPIR-V更加高效，且不同GPU厂商和平台解析起来也一致，但是不会有人直接去写二进制代码  
先写GLSL，再通过vulkan支持的glslangValidator.exe转成SPIR-V

### Vertex shader

==顶点着色器对每个传入的顶点进行处理==，输入坐标和顶点属性，输出裁剪坐标和顶点属性  
输出的裁剪坐标，进行透视除法后，得到NDC坐标  
顶点属性包括：颜色、纹理坐标。ps：==插值==也发生在这个阶段

```glsl
#version 450 
#extension GL_ARB_separate_shader_objects : enable 

out gl_PerVertex { vec4 gl_Position; }; 
vec2 positions[3] = vec2[]( vec2(0.0, -0.5), vec2(0.5, 0.5), vec2(-0.5, 0.5) ); 

void main() { 
	gl_Position = vec4(positions[gl_VertexIndex], 0.0, 1.0); 
}
```

其中，需要添加extension支持，有的变量为内建变量  
in：输入  
out：输出  
out gl_PerVertex { vec4 gl_Position; }; 顶点着色器的固定输出格式  
gl_Position：内建变量，顶点坐标  
gl_VertexIndex：内建变量，顶点索引，一般用于在顶点缓冲中读取数据

### Fragment shader

```glsl
#version 450 
#extension GL_ARB_separate_shader_objects : enable 

layout(location = 0) out vec4 outColor; 
void main() { 
	outColor = vec4(1.0, 0.0, 0.0, 1.0);
}
```

### Color interpolation

对顶点颜色进行插值

vertex shader:

```glsl
#version 450 
#extension GL_ARB_separate_shader_objects : enable 

layout(location = 0) out vec3 fragColor;

out gl_PerVertex { vec4 gl_Position; }; 
vec2 positions[3] = vec2[]( vec2(0.0, -0.5), vec2(0.5, 0.5), vec2(-0.5, 0.5) ); 
vec3 color[3] = vec3[](vec3(1.0, 0.0, 0.0),vec3(0.0, 1.0, 0.0), vec3(0.0, 0.0, 1.0));
void main() { 
	gl_Position = vec4(positions[gl_VertexIndex], 0.0, 1.0); 
	fragColor = colors[gl_VertexIndex];
}
```

fragment shader:

```glsl
#version 450 
#extension GL_ARB_separate_shader_objects : enable 

layout(location = 0) in vec3 fragColor;
layout(location = 0) in vec4 outColor;

void main() { 
	outColor = vec4(fragColor, 1.0);
}
```

对于输入变量和输出变量，通过location来关联，与变量名无关

### Create shader

#### compile

创建shader文件夹，shader.vert存放顶点着色器GLSL代码，shader.frag存放片段着色器GLSL代码，后缀名无影响，习惯命名

创建脚本文件，利用glslangValidator.exe将GLSL转化为SPIR-V

```shell
/home/user/VulkanSDK/x.x.x.x/x86_64/bin/glslangValidator -V shader.vert
/home/user/VulkanSDK/x.x.x.x/x86_64/bin/glslangValidator -V shader.frag
pause
```

得到vert.spv，frag.spv存放SPIR-V代码

#### load

从vert.spv，frag.spv读取数据，存放在vector< char >中

```cpp
#include <fstream>

static std::vector<char> readFile(const std::string& filename) {
    std::ifstream file(filename, std::ios::ate | std::ios::binary);

    if (!file.is_open()) {
        throw std::runtime_error("failed to open file!");
    }
}
```

ps：ios:: ate从尾部读取，可以预先申请好空间，不用动态扩容；ios:: binary 按二进制读取

#### create

根据加载的code 封装 VkShaderModule  
![image.png](https://images-1318884142.cos.ap-guangzhou.myqcloud.com/images/202306291526880.png)  
ps：code记得转为u32

对封装的VkShaderModule指定着色器阶段

```cpp
VkPipelineShaderStageCreateInfo fragShaderStageInfo = {};
fragShaderStageInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
fragShaderStageInfo.stage = VK_SHADER_STAGE_FRAGMENT_BIT;
fragShaderStageInfo.module = fragShaderModule;
fragShaderStageInfo.pName = "main";
```

info下一节再用

## Fixed function

Vulkan中需要对所有状态都显示的给出，包括默认状态  
下面的状态信息，都被用来createPipeline

### Vertex input

顶点输入  
描述传递给顶点着色器的顶点数据格式

### Input assembly

图元装配  
描述几何图元信息，以及是否复用图元

### Viewports and scissors

视口：图像如何==映射==到帧缓冲  
裁剪：裁剪矩形外的像素都会被==过滤==

### Rasterizer

光栅化：多边形的填充、面的朝向剔除

- rasterizerDiscardEnable字段如果设置为VK_TRUE，那么将不会有任何几何体能够通过光栅化阶段。
- depthClampEnable字段被设置为VK_TRUE，那么超出近平面和远平面的片段就会被夹在其中，而不是丢弃。在生成ShadowMap的过程会被用到。
- polygonMode字段决定了如何为几何体填充像素区域。如果需要线框渲染可以通过将polygonMode设为VK_POLYGON_MODE_LINE来实现。
- cullMode字段决定了面剔除的配置，可以禁用或者背面剔除或者前面剔除或两者兼用。frontFace变量规定了被视为正面的顶点顺序，可以是顺时针或逆时针。这将影响面剔除的结果。
- depthBiasEnable字段决定是否开启深度偏置，
- depthBiasConstantFactor是一个常量，控制添加到每个片段的的固定深度值。
- depthBiasClamp是一个片段的最大(或最小)深度偏差。depthBiasSlopeFactor是一个常量，在深度偏差计算中应用于片段的斜率。这个将会在软阴影的实现中用到

### Multisampling

用于减少走样，比起提高分辨率再采样，开销更少  
按下不表

### Depth and stencil testing

- depthTestEnable字段指定是否应将新片段的深度与Depth Buffer进行比较，看它们是否应被丢弃。
- depthWriteEnable字段指定是否应将通过深度测试的新片段的深度实际写入Depth Buffer。
- depthCompareOp字段指定了为保留或丢弃片段所进行的如何去比较。一般来说较低的深度=较近的惯例，所以新片段的深度应该较小。
- depthBoundsTestEnable字段用于可选的深度边界测试是否开启。基本上，这允许你只保留落在指定深度范围内的片段。
- minDepthBounds字段用于深度边界测试的最小深度。
- maxDepthBounds字段用于深度边界测试的最大深度。
- stencilTestEnable字段指定是否开启Stencil Test。
- fron和back字段指定关于StencilTest的配置。

### Color blending

current和last 混合  
current和常量混合

### Dynamic state

pipeline的创建，过于复杂，如果只想更改pipeline的部分属性，直接重建时不合适的  
但是，只有非常有限的管线状态可以在不重建管线的情况下进行动态修改。这包括视口大小，线宽和混合常量

### Pipeline Layout

我们可以在着色器中使用uniform变量，它可以在管线建立后动态地被应用程序修改，实现对着色器进行一定程度的动态配置。  
uniform变量经常被用来传递变换矩阵给顶点着色器，以及传递纹理采样器句柄给片段着色器。

### 参考资料

[理解Vulkan管线(Pipeline) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/617307194)

## Render Pass

### Attachment

描述存放color、depth等  
ps：和buffer有什么区别？？？  
buffer和image是真正存放数据的地方，attachment仅仅是定义数据的格式

### SubPass

每个subPass绑定一条流水线，renderPass可以有多个subPass，subPass的输出可以作为下个subPass的输入

## Pipeline Create

pipeline的创建，需要指定着色器，指定pipeline state，指定pipeline layout, 绑定renderPass和subPass的索引

# Drawing

## Framebuffer

真正存放VkImage的地方，用于屏幕显示  
需要绑定renderPass、attachement

## Command buffer

command而不是函数调用的好处在于，可以收集命令，一次性全部提交到队列中，方便GPU进行并行处理优化

### Command Pool

命令池，管理buffer，从pool中分配内存  
commadnPool需要指定队列族

### Commad Create

帧缓冲和comand buffer一一对应，并从commandPool中分配内存  
vkResetCommandPool：重置commandBuffer，commandPool回收这部分内存

### Command state

- 起始状态，命令缓冲区刚被分配或重置时就是这个状态
- 记录状态，vkBeginCommandBuffer开始录制，命令将被记录在命令缓冲区中
- 可执行状态，vkEndCommandBuffer结束录制，命令缓冲区可以被提交
- 待定状态，命令缓冲区被提交到队列中会使它处于这个状态。提交后无法再更改命令，状态结束后，恢复到可执行状态
- 不可用状态，在这个状态下，命令缓冲区只能被重置或释放。

## Rendering and persentation

drawFrame  
从交换链中获取一个图像  
记录command buffer，绘制到该图像上  
提交command  
present返回swap chain的图形

## Synchronization

不同于OGL对外==伪装==的同步，所有GPU的命令都是异步执行的，OGL对伪装的同步，底下驱动在负重前行

Vulkan在cpu端调用的函数或提交的命令都是立刻执行，返回参数  
但是，函数调用或命令提交在GPU没有被立刻执行，并且其是异步的  
需要进行同步操作，人为控制命令的顺序

### Fence

fence用来在CPU和GPU间进行同步，主要是针对不同帧的同步问题  
例如：截图功能的执行，需要CPU等待GPU渲染完上一帧后，再执行截图

```cpp
VkCommandBuffer A = ... // record command buffer with the transfer
VkFence F = ... // create the fence

// enqueue A, start work immediately, signal F when done
vkQueueSubmit(work: A, fence: F)

vkWaitForFence(F) // blocks execution until A has finished executing

save_screenshot_to_disk() // can't run until the transfer has finished
```

ps: fence会阻塞CPU线程，谨慎使用

### Semaphores

semaphores用来控制GPU的顺序

伪代码：命令B等待命令A执行完，再执行

```cpp
VkCommandBuffer A, B = ... // record command buffers
VkSemaphore S = ... // create a semaphore

// enqueue A, signal S when done - starts executing immediately
vkQueueSubmit(work: A, signal: S, wait: None)

// enqueue B, wait on S to start
vkQueueSubmit(work: B, signal: None, wait: S)
```

### Waiting for the previous frame

通过fence，在drawFrame中，等待上一帧执行完，再渲染下一帧

### Acquiring an image from the swap chain

vkAcquireNextImageKHR：从交换链中获取image, 并通过索引imageIndex指定帧缓冲

### Recording the command buffer

录制命令

### Submitting the command buffer

提交命令，并指定同步信号量

### **Subpass dependencies**

### **Presentation**

渲染结果提交到交换链，呈现到屏幕

## Frame in fight

drawFrame中，阻塞了线程，等待上一帧的绘制完成

此处，开启了异步渲染

# Swap chain recreation

当window窗口size发成变化时，会和交换链不兼容，需要监听窗口变化事件，重建交换链

按下不表

# Vertex buffers

# Uniform buffers

# Texture mapping

# Model

# Rendering quality

# Dynamic scenes

[Push constants - Vulkan Tutorial (Rust) (kylemayes.github.io)](https://kylemayes.github.io/vulkanalia/dynamic/push_constants.html)