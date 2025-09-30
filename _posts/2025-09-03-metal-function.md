---
title: 探索 Metal 音视频技术（3）：函数与库
description: 本文将介绍 Metal 函数与库。
author: Keyframe
date: 2025-09-03 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 渲染, Metal]
pin: false
math: true
mermaid: true
---

>想要学习和提升音视频技术的朋友，快来加入我们的<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">【音视频技术社群】</a>，加入后你就能：
>
>- 1）下载 30+ 个开箱即用的「音视频及渲染 Demo 源代码」
>- 2）下载包含 500+ 知识条目的完整版「音视频知识图谱」
>- 3）下载包含 200+ 题目的完整版「音视频面试题集锦」
>- 4）技术和职业发展咨询 100% 得到回答
>- 5）获得简历优化建议和大厂内推
>  
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/keyframe-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }

---


这个系列文章我们来探索 Metal 音视频技术，对于想要开始学习音视频技术的朋友，这些文章是份不错的入门资料，本章将介绍如何创建 `MTLFunction` 对象来引用 Metal 着色器或计算函数，以及如何使用 `MTLLibrary` 对象来组织并访问这些函数。

## 1、MTLFunction 表示一个着色或计算函数


`MTLFunction` 对象代表一个用 Metal 着色语言编写、在 GPU 上作为图形或计算管线一部分执行的函数。有关 Metal 着色语言的详细信息，请参见《Metal Shading Language Guide》。

为了在 Metal 运行时与用 Metal 着色语言编写的图形或计算函数之间传递数据或状态，你需要为纹理、缓冲区和采样器指定一个参数索引（argument index）。该索引用于标识 Metal 运行时与着色代码所引用的纹理、缓冲区或采样器。

在渲染通道中，你需要在 `MTLRenderPipelineDescriptor`对象中指定一个 `MTLFunction`对象作为顶点或片元着色器，详见“创建渲染管线状态”。在计算通道中，你需要在为目标设备创建 `MTLComputePipelineState` 对象时指定一个 `MTLFunction` 对象，详见“为计算命令编码器指定计算状态与资源”。

## 2、库（Library）是函数的仓库


`MTLLibrary` 对象是一个或多个 `MTLFunction` 对象的仓库。一个 `MTLFunction` 对象代表一个用着色语言编写的 Metal 函数。在 Metal 着色语言源码中，任何使用了 Metal 函数限定符（`vertex`、`fragment`或 `kernel`）的函数，都可以由库中的一个 `MTLFunction` 对象表示。未使用这些限定符的函数不能被直接表示为 `MTLFunction` 对象，但可以被着色器中的其他函数调用。

库中的 `MTLFunction` 对象可以来自以下两种来源：

- 在构建应用过程中，将 Metal 着色语言源码编译为二进制库格式。
- 应用运行时，对包含 Metal 着色语言源码的字符串进行编译。
    

### 2.1、从已编译的代码创建库

为获得最佳性能，应在 Xcode 的应用构建过程中将 Metal 着色语言源码编译为库文件，避免在运行时编译函数源码带来的开销。要从库二进制文件创建 `MTLLibrary`对象，请调用 `MTLDevice`的以下方法之一：

- `newDefaultLibrary`：获取为主程序包构建的库，其中包含应用 Xcode 项目中所有着色和计算函数。
- `newLibraryWithFile:error:`：接收库文件的路径，返回包含该库文件中所有函数的 `MTLLibrary`对象。
- `newLibraryWithData:error:`：接收包含库函数代码的二进制数据，返回 `MTLLibrary`对象。
    

有关在构建过程中编译 Metal 着色语言源码的更多信息，请参见“在应用构建过程中创建库”。

### 2.2、从源码创建库

要从可能包含多个函数的 Metal 着色语言源码字符串创建 `MTLLibrary`，请调用 `MTLDevice`的以下方法之一。这些方法会在创建库时编译源码。若要指定编译器选项，请设置 `MTLCompileOptions`对象的属性。

- `newLibraryWithSource:options:error:`：同步编译输入字符串中的源码，创建 `MTLFunction`对象，并返回包含这些对象的 `MTLLibrary`。
- `newLibraryWithSource:options:completionHandler:`：异步编译输入字符串中的源码，创建 `MTLFunction`对象，并返回包含这些对象的 `MTLLibrary`。`completionHandler`是一个在对象创建完成时被调用的代码块。
    

### 2.3、从库中获取函数

`MTLLibrary`的 `newFunctionWithName:`方法返回指定名称的 `MTLFunction`对象。如果未在库中找到使用 Metal 着色语言函数限定符的函数名称，则 `newFunctionWithName:`返回 `nil`。

**清单 4-1**展示了如何使用`MTLDevice`的`newLibraryWithFile:error:`方法通过完整路径名定位库文件，并利用其内容创建包含一个或多个`MTLFunction`对象的`MTLLibrary`。加载文件时的任何错误将通过`error`返回。接着，使用`MTLLibrary`的`newFunctionWithName:`方法创建表示源码中名为`my_func`的函数的`MTLFunction`对象。返回的函数对象`myFunc`现在可在应用中使用。

**清单 4-1**：从库中访问函数

```
NSError *errors;
id <MTLLibrary> library = [device newLibraryWithFile:@"myarchive.metallib"
                                              error:&errors];
id <MTLFunction> myFunc = [library newFunctionWithName:@"my_func"];
```

## 3、在运行时确定函数详情


由于 `MTLFunction`对象的实际内容由图形着色器或计算函数定义，这些函数可能在 `MTLFunction`对象创建之前就已编译，因此其源码可能无法直接被应用访问。你可以在运行时查询以下 `MTLFunction`属性：

- `name`：函数名称的字符串。
- `functionType`：指示函数声明为顶点、片元还是计算函数。
- `vertexAttributes`：一个 `MTLVertexAttribute`对象数组，描述顶点属性数据在内存中的组织方式及其如何映射到顶点函数参数。详见“用于数据组织的顶点描述符”。
    

`MTLFunction`并不提供对函数参数的访问。可在创建管线状态时获取反射对象（`MTLRenderPipelineReflection`或 `MTLComputePipelineReflection`，取决于命令编码器的类型），该对象揭示了着色或计算函数参数的详细信息。有关创建管线状态与反射对象的详细信息，请参见“创建渲染管线状态”或“创建计算管线状态”。如果不会使用反射数据，请避免获取它们。

反射对象包含命令编码器支持的每类函数的 `MTLArgument`对象数组。对于 `MTLComputeCommandEncoder`，`MTLComputePipelineReflection`有一个 `arguments`属性，其中包含与其计算函数参数对应的 `MTLArgument`对象数组。对于 `MTLRenderCommandEncoder`，`MTLRenderPipelineReflection`有两个属性：`vertexArguments`和 `fragmentArguments`，它们是与顶点函数参数和片元函数参数对应的数组。

并非函数的所有参数都会出现在反射对象中。反射对象仅包含有关联资源的参数，而不包含使用 `[[ stage_in ]]`限定符或内置的 `[[ vertex_id ]]`或 `[[ attribute_id ]]`限定符声明的参数。

**清单 4-2**展示了如何获取反射对象（本例中为 `MTLComputePipelineReflection`），并遍历其 `arguments`属性中的 `MTLArgument`对象。

**清单 4-2**：遍历函数参数

```
MTLComputePipelineReflection* reflection;
id <MTLComputePipelineState> computePS = [device
              newComputePipelineStateWithFunction:func
              options:MTLPipelineOptionArgumentInfo
              reflection:&reflection error:&error];
for (MTLArgument *arg in reflection.arguments) {
    // 处理每个 MTLArgument
}
```

`MTLArgument`的属性揭示了着色语言函数参数的详细信息：

- `name`：参数名称。
- `active`：布尔值，指示该参数是否可以被忽略。
- `index`：在其对应参数表中的从 0 开始的位置。例如，对于 `[[ buffer(2) ]]`，`index`为 2。
- `access`：描述访问限制，例如读或写访问限定符。
- `type`：由着色语言限定符指示，例如 `[[ buffer(n) ]]`, `[[ texture(n) ]]`, `[[ sampler(n) ]]`或 `[[ threadgroup(n) ]]`。
    

`type`决定了其他 `MTLArgument`属性的相关性：

- 如果 `type`是 `MTLArgumentTypeTexture`，则 `textureType`属性指示整体纹理类型（例如着色语言中的 `texture1d_array`、`texture2d_ms`和 `texturecube`类型），`textureDataType`属性指示分量数据类型（如 `half`、`float`、`int`或 `uint`）。
- 如果 `type`是 `MTLArgumentTypeThreadgroupMemory`，则 `threadgroupMemoryAlignment`和 `threadgroupMemoryDataSize`属性相关。
- 如果 `type`是 `MTLArgumentTypeBuffer`，则 `bufferAlignment`、`bufferDataSize`、`bufferDataType`和 `bufferStructType`属性相关。
    

如果缓冲区参数是一个结构体（即 `bufferDataType`为 `MTLDataTypeStruct`），则 `bufferStructType`属性包含一个表示该结构体的 `MTLStructType`，`bufferDataSize`包含该结构体的大小（以字节为单位）。如果缓冲区参数是一个数组（或指向数组的指针），则 `bufferDataType`指示元素的数据类型，`bufferDataSize`包含一个数组元素的大小（以字节为单位）。

**清单 4-3**深入`MTLStructType`对象，检查结构体成员的详细信息，每个成员由`MTLStructMember`对象表示。结构体成员可以是简单类型、数组或嵌套结构体。如果成员是嵌套结构体，则调用`MTLStructMember`的`structType`方法获取表示该结构体的`MTLStructType`对象，然后递归深入分析。如果成员是数组，则使用`MTLStructMember`的`arrayType`方法获取表示它的`MTLArrayType`。然后检查`MTLArrayType`的`elementType`属性。如果`elementType`是`MTLDataTypeStruct`，则调用`elementStructType`方法获取结构体并继续深入分析其成员。如果`elementType`是`MTLDataTypeArray`，则调用`elementArrayType`方法获取子数组并进一步分析。

**清单 4-3**：处理结构体参数

```
MTLStructType *structObj = [arg.bufferStructType];
for (MTLStructMember *member in structObj.members) {
    // 处理每个 MTLStructMember
    if (member.dataType == MTLDataTypeStruct) {
       MTLStructType *nestedStruct = member.structType;
       // 递归深入嵌套结构体
    }
    elseif (member.dataType == MTLDataTypeArray) {
       MTLStructType *memberArray = member.arrayType;
       // 检查 elementType 并在必要时深入
    }
    else {
       // 成员既不是结构体也不是数组
       // 进行分析；无需进一步深入
    }
}
```


---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群和更多同行朋友来交流和讨论：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }



