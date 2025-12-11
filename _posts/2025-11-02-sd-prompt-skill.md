---
title: AI 教程：Stable Diffusion WebUI 提示词技巧
description: 这篇内容就来讨论一下 Stable Diffusion WebUI 提示词技巧。
author: Keyframe
date: 2025-11-02 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-004.jpg
  alt: Stable Diffusion 提示词技巧
---


>向你推荐 **<a href="https://apps.apple.com/app/id6752116909" target="_blank">FaceXSwap</a>**：On-Device Offline AI Face Swap for Free
>
>在 iPhone 上直接对照片、GIF 动图、视频中的人脸实现快速换脸。无需上传任何数据，确保完全隐私与安全。即时、安全、无限次数、功能强大，现在即可免费试用。
>
>![在 AppStore 搜索 'facexswap'](assets/resource/aigc-product/facexswap-2.png)
>_在 AppStore 搜索 'facexswap'_
>
>- FaceXSwap 官网：<a href="https://facexswap.com" target="_blank">https://facexswap.com</a>
>- FaceXSwap iOS App 下载：<a href="https://apps.apple.com/app/id6752116909" target="_blank">https://apps.apple.com/app/id6752116909</a>
{: .prompt-tip }

---







我们前面介绍了文生图的基础功能，可以看到 Stable Diffusion 的功能是基于提示词作为核心输入条件来构建的，这样一来编写提示词的技巧就显得尤为重要，我们这篇内容就来讨论一下 Stable Diffusion WebUI 提示词技巧。



## 1、示例提示词

在学习 Stable Diffusion WebUI 提示词技巧之前，我们可以看看一些示例作品的提示词，分析一下其中有哪些共性的地方可供参考。

1）人物

![](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-002.jpg)

提示词：`An oil painting of the autumnal equinox, a woman surrounded by autumn leaves, an airbrush painting by Josephine Wall, deviantart, psychedelic art, airbrush art, detailed painting, pre-raphaelite, 3d render, rococo art`

翻译：`一幅描绘秋分时节的油画（内容特征），一个被秋叶包裹的女人（内容主体），一幅约瑟芬·沃尔创作的喷绘作品（画风、艺术家），离经叛道的艺术品（画风），迷幻艺术（画风、艺术流派），喷绘艺术（画风、艺术流派），细节绘画（画风），前拉斐尔派（画风、艺术流派），3D 渲染（渲染方式），洛可可艺术（画风、艺术流派）`


2）动物

![](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-003.png)

提示词：`Tiny cute giraffe using a typewriter toy, (standing character), soft smooth lighting, soft pastel colors, skottie young, 3d blender render, polycount, modular constructivism, pop surrealism, physically based rendering, ((square image))`

翻译：`使用打字机玩具可爱长颈鹿（内容主体），站立角色（内容特征），柔和光滑的照明（光线），柔和的粉彩色（内容特征、颜色），斯科蒂·杨（画风、艺术家），3D Blender 渲染（渲染方式），polycount（画风、艺术社区），模块化建构主义（画风、艺术流派），流行超现实主义（画风、艺术流派），物理基础渲染（渲染方式），方形图像（宽高比）`


3）风景

![](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-004.jpg)

提示词：`Multiple layers of silhouette mountains, with silhouette of forest, sharp edges, (at sunset), with heavy fog in air, (vector style), horizon silhouette Landscape wallpaper by Alena Aenami, firewatch game style, vector style background`

翻译：`多层轮廓的山峰（内容主体、内容特征），森林轮廓（内容主体），锐利的边缘（内容特征），日落时分（内容主体），空气中弥漫着浓雾（内容主体），矢量风格（画风），由 Alena Aenami 创作的地平线轮廓风景墙纸（内容主体、画风、艺术家），火表游戏风格（画风），矢量风格的背景（画风）`





从上面的示例作品可以看到，提示词一般都是由一系列的短语组成，这里面可能让大家比较奇怪的是：为什么有的提示词加了括号 `()`？这其实是 Stable Diffusion WebUI 对提示词权重进行设置的一种语法。既然谈到语法，那我们就从提示词权重设置开始介绍一下 Stable Diffusion WebUI 的提示词语法。



## 2、提示词语法



### 2.1、提示词权重

调整提示词权重的方式有下面几种：


**1）越靠前的提示词，权重越高。**


我们用下面的提示词来画一只『天空中的制服猫』：`A painting of a cute goldendoodle wearing a suit, natural light, in the sky, with bright colors, by Studio Ghibli`。结果如下：

![dog](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-005.png)

『制服猫』是有了，但是它却不在空中。这表示 AI 模型没有很好的接收 `in the sky` 这句提示词，其中的一种原因就是权重不够。我们试试把 `in the sky` 的顺序往前放一放，把提示词改为：`A painting of a cute cat in the sky, wearing a suit, natural light, with bright colors, by Studio Ghibli`。这次我们得到的结果如下：

![dog](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-006.png)

『制服猫』终于『在空中』了。


从这个例子中可以看到提示词的顺序对其在生成结果中权重的影响。那我们还有其他改变提示词权重的方法吗？我们接着来介绍。



**2）使用 `(keyword: factor)` 语法设置权重。**


在 `(keyword: factor)` 语法中，`factor` 表示 `keyword` 的权重值，`factor` 小于 1 表示 `keyword` 不如其他提示词重要，大于 1 则表示 `keyword` 比其他提示词更重要。


下面是我们使用提示词 `dog, man, walking, spring in city, by Studio Ghibli` 分别设置 `dog` 的权重为 1 和 1.5 生成的结果图：

![dog](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-007.png)


![(dog: 1.5)](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-008.png)


可以看到，当 `dog` 的权重提升后，画面中出现了更多的狗。

提示词权重对生成图像结果的影响并不一定在每一张图片都有类似的效果，但是在统计意义上是有效的。



**3）使用 `()` 提升权重，使用 `[]` 降低权重。**


`(keyword)` 表示将 `keyword` 的权重提升 1.1 倍，等同于 `(keyword: 1.1)`。

同时，`()` 支持叠加，也就是说 `((keyword))` 表示将 `keyword` 的权重提升 1.1x1.1=1.21 倍，等同于 `(keyword: 1.21)`。

同理，`(((keyword)))` 等同于 `(keyword: 1.331)`。


与之相反，`[keyword]` 表示的是将 `keyword` 的权重降低为原来的 0.9，等同于 `(keyword: 0.9)`。

`[[keyword]]` 等同于 `(keyword: 0.81)`。

`[[[keyword]]]` 等同于 `(keyword: 0.729)`。



### 2.2、提示词混合


我们可以使用 `[keyword1 : keyword2: factor]` 语法做提示词混合。


`[keyword1 : keyword2: factor]` 语法表示的是在绘图过程中，在采样进行到一定步数时把提示词从 `keyword1` 切到 `keyword2` 继续采样，而在哪一步切换，是由 `factor` 决定。`factor` 是一个取值范围在 0-1 的值，它乘上`采样步数（Sampling steps）`得到的数值即 `keyword1` 开始切到 `keyword2` 的步数。

举个例子，当我们用上面的语法将提示词设置为 `Oil painting portrait of [cat: dog: 0.5]`，将`采样步数（Sampling steps）`设置为 30 时，表示在第 30x0.5=15 步时，关键词会发生切换，也就是在 1-15 步采样中，用到的提示词是：`Oil painting portrait of cat`，在 16-30 步采样中，用到的提示词是：`Oil painting portrait of dog`。我们生成的结果如下图所示：


![[cat: dog: 0.5]](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-009.png)


我们把 `factor` 数值分别调整为 0.2 和 0.8，再生成图像得到的效果如下：


![[cat: dog: 0.2]](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-010.png)

![[cat: dog: 0.8]](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-011.png)


可以看到，`factor` 数值越小，结果越接近 `dog`；`factor` 数值越大，结果越接近 `cat`。但是，整体的轮廓都是更像 `cat`，看耳朵部分可以印证这点。这是因为在 Stable Diffusion 扩散绘图的过程中，图像的整体结构是在初始的采样步骤里决定的，所以前面的采样步骤基于 `keyword1` 决定了初始的整体结构，后面的采样步骤则基于 `keyword2` 丰富了细节。


基于这个语法我们可以想到一些有趣的使用场景：

**1）对知名人物（比如明星）进行人脸融合从而创造出有近似长相的新面孔。**



**2）通过设置 `factor` 数值来生成画面结构高度相似的图像。**

举个例子，下面是我们分别用提示词 `Oil painting portrait of a boy holding an [apple: pear: 0.9]` 和 `Oil painting portrait of a boy holding an [apple: pear: 0.1]` 生成的图像效果：

![[apple: pear: 0.9]](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-014.png)

![[apple: pear: 0.1]](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-015.png)

两张图像的整体结构还是很接近的，这背后的原因是图像的整体构图是由早期的扩散过程设定的，一旦扩散被置于一个很小的空间，交换任何关键字都不会对整个图像产生很大的影响，只会改变图像的一小部分。

需要注意的是，在生成的过程中要保持种子（Seed）值和采样步数（Sampling steps）等参数不变，只改动 `factor` 数值。


### 2.3、提示词长度

当我们使用 Stable Diffusion 基础模型进行绘画时，根据你使用的服务，你可以输入的提示词长度可能是有限制的。不过需要注意的是，这里的长度更严谨的含义是指令牌（Tokens）长度，而不是提示词（Prompts）单词长度。Stable Diffusion v1.x 模型可接收的令牌（Tokens）长度是 75。

令牌（Tokens）是由 Stable Diffusion 模型知道的单词转换而成的数字表示形式。Stable Diffusion 是否知道一个单词，是由其训练数据决定的。当我们输入提示词（Prompts）来驱动 Stable Diffusion 进行绘画时，Stable Diffusion 的 CLIP 模块会将提示词（Prompts）转换为模型理解的令牌（Tokens）。如果我们输入的提示词（Prompts）中包含模型不知道的单词，那么这样的单词会被尝试拆分为模型知道的多个词后再转换为令牌（Tokens）。

打个比方，`sun` 和 `house` 是 2 个 Stable Diffusion 模型知道的单词，但是 `sunhouse` 是模型不认识的，这时候当我们把 `sunhouse` 作为 1 个单词放到提示词（Prompts）中，它就会被拆分为 `sun` 和 `house` 这 2 个单词，并转换为 2 个令牌（Tokens）输入给模型。


但是这个令牌（Tokens）长度也可以通过一些方式来突破，在 Stable Diffusion WebUI 的实现中，就在一定的条件下突破了 Stable Diffusion 模型的 75 令牌（Tokens）长度的限制：当提示词（Prompts）包含的令牌（Tokens）长度超过 75 时，就开启一个新的数据块来处理更多的令牌（Tokens），当然这个新块的令牌（Tokens）长度限制也是 75，但是这样的数据块可以一直新开直到计算机的内存耗尽。不过，每个数据块都是独立处理的，在输入给 Stable Diffusion 模型的 U-Net 模块之前才被连接起来。


### 2.4、提示词有效性


上面我们在解释令牌（Tokens）时讲到了 Stable Diffusion 模型受其训练所用数据的限制对于有些单词是不认识的，此时不认识的单词会被尝试拆分成模型认识的较小的单词，但是如果单词不可拆分，这时会怎样呢？这时候，这个单词对于模型来说其实就是噪声，是无效的。

比如，下面是我们用提示词 `cat, by Vincent van Gogh` 生成的几张图像：

![Vincent van Gogh 的有效性测试](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-016.png)

可以看到 `Vincent van Gogh` 应该是对 Stable Diffusion 有效的。

下面是我们用提示词 `cat, by wlop` 生成的几张图像：

![wlop 的有效性测试](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-017.png)

可以看到 `wlop` 似乎并没有起到什么作用，这表示它对 Stable Diffusion 是无效的。


所以，当我们编写提示词时，条件允许的情况下，可以测试一下提示词对模型的有效性，否则可能写了一大堆词只是徒增噪声而已。










## 3、提示词结构

现在我们已经了解了 Stable Diffusion WebUI 提示词的语法规则，现在我们可以继续研究提示词的具体用词，去看看它们有没有通用的结构，这个过程其实是对具体的提示词样本进行抽象分类的一个过程。

回到上面几幅示例作品的提示词，当我们把其中的单词和短语归类后发现它们大部分都包含这些类别：`内容主体、内容特征、颜色、光线、视角、渲染方式、画风、艺术家、艺术流派`。


这些类别还比较离散，我们可以进一步对这些类别再进行归类，如下：

- `内容主体、内容特征` → 内容描述
- `画风、艺术家、艺术流派、光线、颜色、视角、渲染方式` → 风格描述

这样一来，我们就总结出了一种简洁的 Stable Diffusion 绘画提示词结构：**内容描述 + 风格描述**。

由于上面我们参考的示例作品比较少，所以对单词和短语进行第一次归类得到的类别还是比较少的，但是当我们有了**内容描述 + 风格描述**这样更抽象简洁的结构后，我们可以回过头来再补充完善`内容描述`和`风格描述`的细节结构，下面我们就来继续介绍。









### 3.1、内容描述


提示词的**内容描述**部分是指定画面中有什么。编写这部分时，除了考虑上面我们在示例作品中归类的`内容主体、内容特征`外，更完备的做法是思考下面几个问题：

- 1）主体是什么？
- 2）主体有什么特征和细节？
- 3）主体之外有没有其他元素，与主体之间的关系是什么？
- 4）其他元素有什么特征和细节？
- 5）画面的背景、环境是什么？


举个例子：


1）主体是什么？

我们想画一个男孩，所以我们写下提示词：`a boy`。

2）主体有什么特征和细节？

我们希望男孩穿着套装、一头短发，所以提示词可以完善为：`a boy wearing a suit with short hair`。

3）主体之外有没有其他元素，与主体之间的关系是什么？

我们想让画面中有一棵苹果树，所以我们继续添加提示词：`a boy wearing a suit with short hair standing under an apple tree`。

这里我们交代了一下主体和元素的关系，即：男孩是站在苹果树下。这是需要注意的一点，当画面除了主体外还有其他元素时，需要尽量让他们之间的关系符合逻辑。

4）其他元素有什么特征和细节？

我们希望苹果树上有红色的苹果，所以我们完善提示词：`a boy wearing a suit with short hair standing under an apple tree, red apples on the tree`。


5）画面的背景、环境是什么？

画面应该是在一个果园，所以我们继续添加提示词：`a boy wearing a suit with short hair standing under an apple tree, red apples on the tree, in an orchard`


最后我们画图得到的结果如下：

![内容描述](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-018.png)





### 3.2、风格描述


**风格描述**部分则是指定图像的风格。编写这部分，我们可以考虑下面几个方面：




**1）画风**


要影响图像的`画风`有很多方法，我们可以通过指定`艺术媒介、艺术家、艺术流派、艺术工作室、艺术作品、年代`等词来设定画风。


**1.1）艺术媒介**


比如，是照片（Photo）还是画作（Painting）、是素描（Sketch）还是油画（Oil painting）等等，这些表示艺术媒介的词可以影响画风。

下图是用分别用下面两条提示词画出的两只猫：

- ① `cat, Sketch`
- ② `cat, Oil painting`


![cat, Sketch](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-019.png)

![cat, Oil painting](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-020.png)


下面是一些`艺术媒介`提示词示例：

- `Sketch`：素描
- `Pencil Painting`：铅笔画
- `Oil Painting`：油画
- `Chalk Painting`：粉笔画
- `Water Color Painting`：水彩画
- `Graffiti`：街头涂鸦
- `Fabric`：针织物
- `Made of Wood`：木制品
- `Clay`：陶土制品

大家可以探索和尝试更多其他的提示词，也许会收获惊喜的效果。


**1.2）艺术家**

我们也可以通过指定`艺术家`来让画风匹配艺术家的风格。


下图是分别用下面几条提示词绘图得到的结果：

- ① `masterpiece by Pablo Picasso`
- ② `masterpiece by Vincent van Gogh`
- ③ `masterpiece by Hayao Miyazaki`
- ④ `masterpiece by Makoto Shinkai`


![masterpiece by Pablo Picasso](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-021.png)

![masterpiece by Vincent van Gogh](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-022.png)

![masterpiece by Hayao Miyazaki](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-023.png)

![masterpiece by Makoto Shinkai](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-024.png)


下面是一些`艺术家`提示词示例：

- `Pablo Picasso`：巴勃罗·毕加索，抽象派画家
- `Vincent van Gogh`：梵高，后印象派画家
- `Paul Cézanne`：保罗·塞尚，后印象派画家
- `Georges Pierre Seurat`：秀拉，印象派点描派画家
- `Hayao Miyazaki`：宫崎骏，漫画家
- `Makoto Shinkai`：新海诚，漫画家
- `Eiichiro Oda`：尾田荣一郎，漫画家
- `Katsuhiro Otomo`：大友克洋，漫画家




**1.3）艺术流派**

我们也可以通过指定`艺术流派`来设定画风。下面是几个示例的提示词和结果：

- ① `masterpiece by Impressionism`
- ② `masterpiece by Rococo`
- ③ `masterpiece by Cubism`
- ④ `masterpiece by Constructivism`


![masterpiece by Impressionism](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-025.png)

![masterpiece by Rococo](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-026.png)

![masterpiece by Cubism](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-027.png)

![masterpiece by Constructivism](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-028.png)



下面是一些`艺术流派`提示词示例：

- `Impressionism`：印象派
- `Rococo`：洛可可式
- `Fauvism`：野兽派
- `Cubism`：立体派
- `Abstract Art`：抽象艺术
- `Abstract xpressionism`：抽象表现主义
- `Baroque`：巴洛克
- `Constructivism`：建构主义
- `Surrealism`：超现实主义



**1.4）艺术工作室**

我们还可以通过指定`艺术工作室`来设定画风。下面是几个示例的提示词和结果：

- ① `masterpiece by DreamWorks Pictures`
- ② `masterpiece by Pixar`
- ③ `masterpiece by Ghibli Studio`


![masterpiece by DreamWorks Pictures](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-029.png)

![masterpiece by Pixar](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-030.png)

![masterpiece by Ghibli Studio](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-031.png)



下面是一些`艺术工作室`提示词示例：

- `DreamWorks Pictures`：梦工厂动画
- `Pixar`：皮克斯动画
- `Ghibli Studio`：吉卜力



**1.5）艺术作品**


我们也可以通过指定`艺术作品`来设定画风。艺术作品可以是电影、动漫、游戏等等。

下面是几个示例的提示词和结果：

- ① `masterpiece by Jojo's Bizarre Adventure`
- ② `masterpiece by Pokémon`
- ③ `masterpiece by League of legends`
- ④ `masterpiece by Breath of The Wild`


![masterpiece by Jojo's Bizarre Adventure](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-032.png)

![masterpiece by Pokémon](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-033.png)

![masterpiece by League of legends](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-034.png)

![masterpiece by Breath of The Wild](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-035.png)



下面是一些`艺术作品`提示词示例：

- `Jojo's Bizarre Adventure`：JoJo 的奇妙冒险，动漫作品
- `Pokémon`：宝可梦，动漫作品
- `AFK Arena`：剑与远征，游戏
- `League of legends`：英雄联盟，游戏
- `Breath of The Wild`：旷野之息，游戏
- `Warframe`：星际战甲，游戏



**1.6）年代**

我们还可以通过指定`年代`来影响画风。下面是几个示例的提示词和结果：


- ① `beautiful young girl, in low cut, 1960s`
- ② `beautiful young girl, in low cut, 1990s`


![beautiful young girl, in low cut, 1960s](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-036.png)

![beautiful young girl, in low cut, 1990s](assets/resource/aigc-tutorial/sd-prompt-skill/sd-prompt-skill-037.png)





**2）光线**


下面是一些`光线`提示词示例：

- `Volumetric Lighting`：体积光
- `Mood Lighting`：气氛光
- `Bright`：明亮
- `Soft Lights`：柔光
- `Rays of Shimmering Light`：闪烁的光线
- `Crepuscular Ray`：云隙光
- `Bioluminescence`：生物发光
- `Bisexual Lighting`：双性打光
- `Rembrandt Lighting`：人物的 45 度角侧向光
- `Split Lighting`：高对比侧面光
- `Front Lighting`：正面光
- `Back Lighting`：逆光
- `Oblique Back Lighting`：斜逆光
- `Rim Lights`：边缘光
- `Global Illumination`：全局光
- `Warming Lighting`：暖光灯
- `Dramatic Lighting`：戏剧灯光
- `Natural Lighting`：自然光




**3）视角**



下面是一些`视角`提示词示例：


- `Aerial View`：鸟瞰
- `Top View`：顶视
- `Tilt-shift`：移轴效果
- `Satellite View`：卫星视图
- `Bottom View`：仰视
- `Front/Side/Rear View`：前/侧/后视图
- `Product View`：产品视角
- `Closeup View`：特写视角
- `Outer Space View`：太空视角
- `Isometric View`：等距视角
- `High Angle View`：高角度视角
- `Microscopic View`：微视角
- `First-person View`：第一人称视角
- `Third-person Perspective`：第三人称视角
- `Two-point Perspective`：两点透视
- `Three-point Perspective`：三点透视
- `Elevation Perspective`：立面视角
- `Cinematic Shot`：电影镜头
- `In Focus`：对焦
- `Depth of Field`：景深
- `Wide-angle View`：广角



**4）摄影参数**


图像的`摄影参数`可以是镜头、设备等。



下面是一些`摄影参数`提示词示例：


- `Wide-angle Lens`：广角镜头
- `Telephoto Lens`：长焦镜头
- `24mm Lens`：24mm 镜头
- `EF 70mm Lens`：EF 70mm 镜头
- `800mm Lens`：800mm 长焦镜头
- `Fish-eye Lens`：鱼眼镜头
- `Macro Lens`：微距镜头
- `iPhone X`：iPhone X 手机摄影
- `Nikon Z FX`：Nikon Z FX 相机摄影
- `Canon`：佳能相机摄影
- `Gopro`：Gopro 相机摄影
- `Drone`：无人机摄影
- `Thermal Camera`：热成像相机摄影




**5）渲染方式**


下面是一些`渲染方式`提示词示例：


- `Unreal Engine 5`：虚幻引擎 5 渲染
- `3D Render`：3D 渲染



**6）魔法词**

除了上面这些类别的提示词外，还有一些魔法词也可以在绘图的时候尝试一下。


- `HDR, UHD, 4K/8K/64K`：这些指定图像质量的提示词有时候能起到不错的效果
- `Highly Detailed`：高度细节，有时可以让图像呈现更多细节
- `Studio Lighting`：摄影棚内光线，有时可以给图像增添不错的纹理效果
- `Professional`：专业级，有时可以提升图像的对比度和细节呈现
- `Trending on Artstation`：艺术站趋势，可以生成流行的风格
- `Vivid Colors`：生动的色彩，可以提升图像的颜色丰富度
- `High Resolution Scan`：高分辨率扫描，可以给照片增加年代感
- `Bokeh`：可以实现背景虚化的效果




### 3.3、Stable Diffusion 参数

除了上面提到的提示词外，对生产图像影响很大的还包括 Stable Diffusion 参数信息。这里面就包括：`分辨率（Width/Height）、提示词相关性（CFG Scale）、采样方法（Sampling method）、采样步数（Sampling steps）`。

这些参数在前面《文生图》的内容中已经介绍过，这里就不再赘述了。

























