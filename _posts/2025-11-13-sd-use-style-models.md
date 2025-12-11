---
title: AI 教程：使用微调模型控制生成画风
description: 这一篇内容我们就来介绍一些使用微调模型控制生成画风的案例。
author: Keyframe
date: 2025-11-13 08:08:08 +0800
categories: [AI 教程]
tags: [AI 教程, AI, AIGC, Stable Diffusion]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-005.jpg
  alt: Stable Diffusion 微调模型
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





我们已经介绍过如何在 Stable Diffusion WebUI 中配置各种模型的方法，这些模型最大的用处就是用来支持我们进行不同风格、不同场景的高质量 AI 绘图，这一篇内容我们就来介绍一些使用微调模型控制生成画风的案例。

下面介绍的各种画风只是 Stable Diffusion WebUI 搭配各种微调模型所能支持的一小部分而已，大家可以到模型发现和下载网站去找到更多模型来创作其他风格的作品。


## 1、真人照片风格



**1）示例 1**


![真人照片风格示例 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-001.png)


主模型：

- ChilloutMix: [https://civitai.com/models/6424/chilloutmix](https://civitai.com/models/6424/chilloutmix)


LoRA 模型：

- koreanDollLikeness_v10: [https://huggingface.co/Kanbara/doll-likeness-series/tree/main](https://huggingface.co/Kanbara/doll-likeness-series/tree/main)


提示词：

```
<lora:koreanDollLikeness_v10:0.35>,best quality ,masterpiece, illustration, an extremely delicate and beautiful, extremely detailed ,CG ,unity ,8k wallpaper, Amazing, finely detail, masterpiece,best quality,official art,extremely detailed CG unity 8k wallpaper,absurdres, incredibly absurdres, huge filesize , ultra-detailed, highres, extremely detailed,beautiful detailed girl, extremely detailed eyes and face, beautiful detailed eyes,light on face,(Hanfu:1.1),1girl.
```

负向提示词： 

```
sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad anatomy,(long hair:1.4),DeepNegative,(fat:1.2),facing away, looking away,tilted head, {Multiple people}, lowres,bad anatomy,bad hands, text, error, missing fingers,extra digit, fewer digits, cropped, worstquality, low quality, normal quality,jpegartifacts,signature, watermark, username,blurry,bad feet,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digit, extra arms, extra leg, extra foot,
```


其他参数：

```
Steps: 35, Sampler: DPM++ SDE Karras, CFG scale: 7.5, Seed: 1798156725, Size: 448x704, Model hash: 3a17d0deff, Denoising strength: 0.57, Clip skip: 2, ENSD: 31337, Hires upscale: 1.5, Hires upscaler: Latent
```



**2）示例 2**


![真人照片风格示例 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-002.png)


主模型：

- ChilloutMix: [https://civitai.com/models/6424?modelVersionId=11745](https://civitai.com/models/6424?modelVersionId=11745)


LoRA 模型：

- japaneseDollLikeness_v15: [https://huggingface.co/Kanbara/doll-likeness-series/tree/main](https://huggingface.co/Kanbara/doll-likeness-series/tree/main)
- koreanDollLikeness_v20: [https://huggingface.co/Kanbara/doll-likeness-series/tree/main](https://huggingface.co/Kanbara/doll-likeness-series/tree/main)
- Cute_girl_mix4: [https://civitai.com/models/14171?modelVersionId=16677](https://civitai.com/models/14171?modelVersionId=16677)
- ChilloutMixss3.0: [https://civitai.com/models/16274?modelVersionId=19219](https://civitai.com/models/16274?modelVersionId=19219)



提示词：

```
masterpiece,bestquality,(realistic,photo-realistic:1.4),(RAWphoto:1.2),extremelydetailedCGunity8kwallpaper,anextremelydelicateandbeautiful,amazing,finelydetail,officialart,absurdres,incrediblyabsurdres,hugefilesize,ultra-detailed,extremelydetailed,beautifuldetailedgirl,(perfectfemalefigure),extremelydetailedeyesandface,beautifuldetailedeyes,(wearingschooluniformandgreyplaidskirt),collaredshirt,(softlight),lightonface,lookingatviewer,slimwaist,(fullbody),(smallbreasts),standing,midriff,bangs,<lora:japaneseDollLikeness_v15:0.2>,<lora:koreanDollLikeness_v20:0.2>,<lora:mix4:0.4>,<lora:xss3-0:0.2>,pureerosface_v1:0.8
```

负向提示词： 

```
(worst quality:2), (low quality:2), (normal quality:2), lowres, ((monochrome)), ((grayscale)), paintings, sketches, nipples, skin spots, acnes, skin blemishes, bad anatomy, facing away, looking away, tilted head, lowres, bad anatomy, bad hands, text, error, missing fingers, extra digit, fewer digits, blurry, bad feet, cropped, poorly drawn hands, poorly drawn face, mutation, deformed, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, extra fingers, fewer digits, extra limbs, extra arms, extra legs, malformed limbs, fused fingers, too many fingers, long neck, cross-eyed, mutated hands, polar lowres, bad body, bad proportions, gross, nipples, ng_deepnegative_v1_75t, badhandv4
```

其他参数：

```
Steps: 20, Size: 512x768, Seed: 365013168, Model: chilloutmix_NiPrunedFp32Fix, Sampler: DPM++ 2M Karras, CFG scale: 7
```



**3）示例 3**


![真人照片风格示例 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-003.jpg)


主模型：

- ChilloutMix: [https://civitai.com/models/6424?modelVersionId=11745](https://civitai.com/models/6424?modelVersionId=11745)


LoRA 模型：

- koreanDollLikeness_v15: [https://huggingface.co/Kanbara/doll-likeness-series/tree/main](https://huggingface.co/Kanbara/doll-likeness-series/tree/main)）



**4）示例 4**


![真人照片风格示例 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-004.jpg)


主模型：

- ChilloutMix: [https://civitai.com/models/6424?modelVersionId=11745](https://civitai.com/models/6424?modelVersionId=11745)


LoRA 模型：

- koreanDollLikeness_v15: [https://huggingface.co/Kanbara/doll-likeness-series/tree/main](https://huggingface.co/Kanbara/doll-likeness-series/tree/main)）




提示词：

```
<lora:Beautiful_Dress:0.65>,(printed colorful dress),(bare shoulders),1 girl,(full body),(stiletto heels),(short hair:1.1),(realistic:1.7),((best quality)),absurdres,(ultra high res),(photorealistic:1.6),photorealistic,octane render,(hyperrealistic:1.2), (photorealistic face:1.2), (8k), (4k), (Masterpiece),(realistic skin texture), (illustration, cinematic lighting,wallpaper),( beautiful eyes:1.2),((((perfect face)))),(cute),(standing),(smile:0.9),(black hair),(long hair),black eyes,red lips, (outdoors),  <lora:koreanDollLikeness_v15:0.4>,
```

负向提示词： 

```
Negative prompt: (worst quality:2), (low quality:2), (normal quality:2),(uneven eyes),lowres, normal quality,bad anatomy,bad face,(uneven eyes),paintings,ugly, bad hands,open mouth,multiple girls,extra faces, extro breasts, multiple breasts,obese, fat rolls,extra arms, extra eyes,inverted nipples,extra ears,nipple rings,severed arm, bad arm, nipple bar,asymmetrical eyebrows,big mouth,
```

其他参数：

```
Steps: 25, ENSD: 31337, Size: 1856x3712, Seed: 317121033, Model: chilloutmix_NiPrunedFp32Fix, Sampler: DPM++ 2M Karras New, CFG scale: 6.5, Clip skip: 2, Mask blur: 4, Model hash: fc2511737a, Denoising strength: 0.2, Ultimate SD upscale padding: 64, Ultimate SD upscale upscaler: R-ESRGAN 4x+, Ultimate SD upscale mask_blur: 16, Ultimate SD upscale tile_width: 921, Ultimate SD upscale tile_height: 1382
```





## 2、风景画作风格




**1）示例 1**


![风景画作风格示例 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-005.jpg)


主模型：

- Counterfeit-V3.0: [https://civitai.com/models/4468?modelVersionId=57618](https://civitai.com/models/4468?modelVersionId=57618)





提示词：


```
(1girl:1.5), In a green meadow stands a girl leading a group of knights. BREAK With a brave expression, she guides them towards their destination. BREAK Behind her, a green forest stretches out and beyond that, mountains rise in the distance. BREAK The most suitable effect for this scene would be a watercolor painting technique to capture the softness of the meadow and the fluidity of the movement.
```

负向提示词：

```
EasyNegativeV2, wind, lowres, bad anatomy, bad hands, missing fingers, extra digit, fewer digits, head out of frame, cropped
```


其他参数：

```
Steps: 25, ENSD: 31337, Size: 1280x1920, Seed: 3788104181, Model: Counterfeit-V3.0_fp16, model: control_v11f1e_sd15_tile [a371b31b], weight: 1, Version: v1.3.1, Sampler: Euler a, CFG scale: 7, Clip skip: 2, Model hash: cbfba64e66, resize mode: Just Resize, control mode: ControlNet is more important, "preprocessor: tile_resample, pixel perfect: True, starting/ending: (0, 1), Denoising strength: 0.35, preprocessor params: (512, 1, 64)", Ultimate SD upscale padding: 32, Ultimate SD upscale upscaler: R-ESRGAN 4x+ Anime6B, Ultimate SD upscale mask_blur: 8, Ultimate SD upscale tile_width: 640, Ultimate SD upscale tile_height: 640
```



**2）示例 2**


![风景画作风格示例 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-006.jpg)


主模型：

- MIX-Pro-V4: [https://civitai.com/models/7241/mix-pro-v4?modelVersionId=34559](https://civitai.com/models/7241/mix-pro-v4?modelVersionId=34559)

LoRA 模型：

- Airconditioner: [https://civitai.com/models/22607?modelVersionId=27334](https://civitai.com/models/22607?modelVersionId=27334)


提示词：


```
outdoors, power lines, scenery, building, utility pole, ((horizon)), street, sky, road, city, cityscape, cinematic lighting, light pole, sunset, sunlight, shadow, extremely detailed, <lora:LoconLoraAirconditioner_loraVersion:1>
```

负向提示词：

```
Negative prompt: (watermark), sketch, duplicate, monochrome, blurry, depth of field, (worst quality:2.0), (low quality:2.0), By bad artist -neg, bad_prompt_version2,
```

其他参数：

```
Steps: 30, ENSD: 31337, Size: 720x512, Seed: 2680886306, Model: mixProV4_v4, Sampler: DDIM, CFG scale: 7, Clip skip: 2, Model hash: 61e23e57ea, Hires steps: 15, Hires resize: 1024x720, Hires upscaler: Latent (nearest-exact), Denoising strength: 0.58
```


**3）示例 3**


![风景画作风格示例 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-007.jpg)


主模型：

- Counterfeit-V3.0: [https://civitai.com/models/4468?modelVersionId=57618](https://civitai.com/models/4468?modelVersionId=57618)

LoRA 模型：

- Pyramid_lora_Ghibli_Background: [https://civitai.com/models/64610?modelVersionId=69243](https://civitai.com/models/64610?modelVersionId=69243)


提示词：

```
(((best quality))), old school bus, old gravel road, trees, field, flowers <lora:Pyramid lora_Ghibli_v2:1>
```

其他参数：

```
Steps: 20, Size: 608x304, Seed: 838764230, Model: CounterfeitV30_v30, Version: v1.2.1, Sampler: DPM++ 2M, CFG scale: 5, Model hash: cbfba64e66, Hires upscale: 2, Hires upscaler: Latent, Denoising strength: 0.5
```




**4）示例 4**


![风景画作风格示例 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-008.jpg)


主模型：

- Counterfeit-V3.0: [https://civitai.com/models/4468?modelVersionId=57618](https://civitai.com/models/4468?modelVersionId=57618)

LoRA 模型：

- Pyramid_lora_Ghibli_Background: [https://civitai.com/models/64610?modelVersionId=69243](https://civitai.com/models/64610?modelVersionId=69243)

提示词：

```
(((best quality))),High saturation,clear,rational construction,cartoon style,architecture, bridge, building,castle, chimney, cloud, cloudy_sky, day,  fantasy, gate, grass, house, mountain, no_humans, outdoors, river, scenery, sky, sunset,, tree, window ,<lora:Pyramid lora_Ghibli:0.8>
```

负向提示词： 

```
Negative prompt: easynegative,Unreasonable structure, Redundant structure,duplicate Windows, Redundant chimney,Strange structure,Twisted structure,Fuzzy structure,Strange, confusing, irrational
```

其他参数：

```
Steps: 28, Size: 616x416, Seed: 2053107434, Model: concept_scene_B, Sampler: DPM++ 2M Karras, CFG scale: 7, Model hash: a074b8864e, Hires upscale: 2, Hires upscaler: Latent, Denoising strength: 0.7
-->





## 3、二次元漫画人物风格


**1）示例 1**


![二次元漫画人物风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-009.jpg)


主模型：

- Meina V11: [https://civitai.com/models/7240?modelVersionId=119057](https://civitai.com/models/7240?modelVersionId=119057)





提示词：

```
1girl, japanese armor, white hair, purple eyes, sword, ((holding sword)), blue flames, glowing weapon, light particles, wallpaper, chromatic aberration
```

负向提示词： 

```
Negative prompt: (worst quality, low quality:1.4), (zombie, sketch, interlocked fingers, comic),
```

其他参数：

```
Steps: 28, Size: 512x1024, Seed: 935594180, Model: Meina V11, Version: v1.4.0, Sampler: DPM++ SDE Karras, CFG scale: 7, Clip skip: 2, Model hash: 54ef3e3610, Hires steps: 10, Hires upscale: 2, Hires upscaler: R-ESRGAN 4x+ Anime6B, Denoising strength: 0.35
-->



**2）示例 2**


![二次元漫画人物风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-010.jpg)


主模型：

- Meina V11: [https://civitai.com/models/7240?modelVersionId=119057](https://civitai.com/models/7240?modelVersionId=119057)

提示词：

```
1girl, oil painting, painting \(medium\), ((wlop))
```

负向提示词： 

```
Negative prompt: (worst quality, low quality:1.4), (zombie, sketch, interlocked fingers, comic),
```

其他参数：

```
Steps: 25, Size: 512x1024, Seed: 4193302177, Model: Meina V11, Version: v1.4.0, Sampler: DPM++ SDE Karras, CFG scale: 7, Clip skip: 2, Model hash: 54ef3e3610, Hires steps: 10, Hires upscale: 2, Hires upscaler: R-ESRGAN 4x+ Anime6B, Denoising strength: 0.5
-->


**3）示例 3**


![二次元漫画人物风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-011.jpg)


主模型：

- Counterfeit-V3.0: [https://civitai.com/models/4468?modelVersionId=57618](https://civitai.com/models/4468?modelVersionId=57618)

LyCORIS 模型：

- pseudo-daylight: [https://civitai.com/models/66578?modelVersionId=71235](https://civitai.com/models/66578?modelVersionId=71235)

提示词：

```
kahuka1, 1girl, solo, flower, long hair, red flower, looking at viewer, hair flower, hatsune miku, hair ornament, branch, blurry, upper body, hair between eyes, leaf, aqua hair, rose, aqua eyes, bangs, head wreath
<lora:kahuka-pynoise:1>
```

负向提示词： 

```
Negative prompt: (worst quality, low quality, extra digits:1.4),  badhandv4, EasyNegative
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 512x768, Seed: 536513680, Model: Counterfeit-V3.0_fp16, Sampler: UniPC, CFG scale: 6.5, Clip skip: 2, Model hash: cbfba64e66, Hires steps: 20, Hires upscale: 2, Hires upscaler: Latent (nearest-exact), Denoising strength: 0.58
-->



**4）示例 4**


![二次元漫画人物风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-012.jpg)


主模型：

- ReV Animated v1.2.2: [https://civitai.com/models/7371?modelVersionId=46846](https://civitai.com/models/7371?modelVersionId=46846)

LoRA 模型：

- Studio Ghibli Style LoRA: [https://civitai.com/models/6526?modelVersionId=7657](https://civitai.com/models/6526?modelVersionId=7657)



提示词：

```
ghibli style, EVA, 1girl, arms behind back, bare shoulders, blush, brown eyes, brown hair, long hair,wearing earrings, looking at viewer, sleeveless, smile, solo, upper body , ((masterpiece)) , <lora:ghibli_style_offset:1>
```

负向提示词： 

```
Negative prompt: (painting by bad-artist-anime:0.9), (painting by bad-artist:0.9), watermark, text, error, blurry, jpeg artifacts, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, artist name, (worst quality, low quality:1.4), bad anatomy
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 768x900, Seed: 1838318321, Model: revAnimated_v122, Sampler: DPM++ SDE Karras, CFG scale: 7, Clip skip: 2, Model hash: 4199bcdd14
```







## 4、等距微缩风格


**1）示例 1**


![等距微缩风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-013.jpg)


主模型：

- Isometric Future: [https://civitai.com/models/10063/isometric-future](https://civitai.com/models/10063/isometric-future)


提示词：

```
IsometricFuture, Anime Style School, isometric cutaway, night sky,  city view, Depth of Field,  Full-HD, Rim Lighting, Vibrant Sharp Cinematic Lighting, Shadows, Chromatic Aberration, art by artgerm, greg rutowski, square enix, unreal engine 5, FXAA, IsometricFuture
```

负向提示词： 

```
Negative prompt: text, watermarks, logo, out of frame, blurry,
```

其他参数：

```
Steps: 40, Size: 512x512, Seed: 3776673056, Model: Stable_Diffusion_Models_Duskfall_Models_IsometricFuture, Sampler: DPM++ SDE, CFG scale: 10, Clip skip: 2, Model hash: e884eb8de4
```


**2）示例 2**


![等距微缩风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-014.jpg)


主模型：

- Realistic Vision V2.0: [https://civitai.com/models/4201?modelVersionId=29460](https://civitai.com/models/4201?modelVersionId=29460)


LyCORIS 模型：

- Miniature world style: [https://civitai.com/models/28531/miniature-world-style?modelVersionId=34223](https://civitai.com/models/28531/miniature-world-style?modelVersionId=34223)


提示词：

```
mini\(ttp\), (8k, RAW photo, best quality, masterpiece:1.2), factory, miniature, landscape,  isometric, <lora:miniature_V1:0.6>, in a box,
```

负向提示词： 

```
Negative prompt: (((blur))), (EasyNegative:1.2), ng_deepnegative_v1_75t, paintings, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), bad anatomy,(long hair:1.4),DeepNegative,(fat:1.2),facing away, looking away,tilted head,lowres,bad anatomy,bad hands, text, error, missing fingers,extra digit, fewer digits, cropped, worstquality, low quality, normal quality,jpegartifacts,signature, watermark, username,blurry,bad feet,cropped,poorly drawn hands,poorly drawn face,mutation,deformed,worst quality,low quality,normal quality,jpeg artifacts,signature,watermark,extra fingers,fewer digits,extra limbs,extra arms,extra legs,malformed limbs,fused fingers,too many fingers,long neck,cross-eyed,mutated hands,polar lowres,bad body,bad proportions,gross proportions,text,error,missing fingers,missing arms,missing legs,extra digi
```

其他参数：

```
Steps: 40, ENSD: 31337, Size: 768x768, Seed: 3957513106, Model: realisticVisionV20_v20, Sampler: DPM++ 2M Karras, CFG scale: 7, Model hash: e6415c4892, Hires upscale: 2, Hires upscaler: ESRGAN_4x, Denoising strength: 0.5
```


**3）示例 3**


![等距微缩风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-015.jpg)


主模型：

- Lyriel: [https://civitai.com/models/22922/lyriel?modelVersionId=46658](https://civitai.com/models/22922/lyriel?modelVersionId=46658)

LoRA 模型：

- Isometric Chinese style Architecture LoRa: [https://civitai.com/models/63376/isometric-chinese-style-architecture-lora](https://civitai.com/models/63376/isometric-chinese-style-architecture-lora)

提示词：

```
isometric chinese style architecture
```

负向提示词： 

```
Negative prompt: 3d, cartoon, anime, sketches, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, bad anatomy, girl, loli, young, large breasts, red eyes, muscular
```

其他参数：

```
Steps: 35, ENSD: 31337, Size: 512x512, Seed: 302680275, Model: lyriel_v14, Sampler: DPM++ 2M Karras, CFG scale: 7, Model hash: f1b08b30f8, Hires steps: 40, Mimic scale: 7, Hires upscale: 2, AddNet Enabled: True, AddNet Model 1: Isometric_Chinese_style_architecture_v01(b6686cd9b728), Hires upscaler: Latent, AddNet Module 1: LoRA, AddNet Weight A 1: 0.65, AddNet Weight B 1: 0.65, Denoising strength: 0.53, Threshold percentile: 100, Dynamic thresholding enabled: True
```



**4）示例 4**


![等距微缩风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-016.jpg)


主模型：

- Isometric Future: [https://civitai.com/models/10063/isometric-future](https://civitai.com/models/10063/isometric-future)

提示词：

```
isometric cutaway,It has texture,bed,sofa, table, stool, bonsai, mural,Windows, desks,white_background,simple _background,Depth of Field, Gamma, High Contrast, Kinemacolor, 3D, Ray Tracing Global Illumination, Ray Traced, Anti-Aliasing, FXAAFull-HD, Rim Lighting,Shadows,unreal engine 5,octane render,dribbble,pinterest,knollingcase, 8k
,duskametric15
```

负向提示词： 

```
Negative prompt: text, watermarks, logo, out of frame, blurry,
```

其他参数：

```
Steps: 30, Size: 512x512, Seed: 570156573, Model: isometricFuture_isometricFutures10, Sampler: DPM++ SDE Karras, CFG scale: 6.5, Model hash: aa9c45d00a, Hires steps: 10, Hires upscale: 2, Hires upscaler: R-ESRGAN 4x+, Denoising strength: 0.4
```






## 5、B 端元素风格



**1）示例 1**


![B 端元素风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-017.jpg)


主模型：

- DDicon: [https://civitai.com/models/38511/ddicon?modelVersionId=44447](https://civitai.com/models/38511/ddicon?modelVersionId=44447)


提示词：

```
best quality, many details,4k,blender,octane render,C4D,transparent glass texture,DDicon,blue|purple, frosted glass, transparent technology sense, industrial design,dark gradient background,studio lighting, sunshine,flat, minimal,quasi-object，axisymmetric，Data, cylinder,cube,camera,tiktok,
```

负向提示词： 

```
Negative prompt: lowers, bad anatomy, ((bad hands)), (worst quality:2), (low quality:2), (normal quality:2), paintings, sketches, lowres, bad anatomy, bad hands, text, error, missing fingers,
```

其他参数：

```
Steps: 20, Sampler: Euler a, CFG scale: 7
```




**2）示例 2**


![B 端元素风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-018.jpg)


主模型：

- DDicon: [https://civitai.com/models/38511/ddicon?modelVersionId=44447](https://civitai.com/models/38511/ddicon?modelVersionId=44447)

LoRA 模型：

- DDicon_lora: [https://civitai.com/models/38558/ddiconlora](https://civitai.com/models/38558/ddiconlora)


提示词：

```
best quality, many details,4k,blender,octane render,C4D,transparent glass texture,DDicon,blue|green, frosted glass, transparent technology sense, industrial design,white background,studio lighting, sunshine,flat, minimal,quasi-object，axisymmetric，Data, file，shiled，
```

负向提示词： 

```
Negative prompt: lowres, bad anatomy, ((bad hands)), (worst quality:2), (low quality:2), (normal quality:2), paintings, sketches, lowres, bad anatomy, bad hands, text, error, missing fingers,
```

其他参数：

```
Steps: 20, Sampler: Euler a, CFG scale: 7
```



**3）示例 3**


![B 端元素风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-019.jpg)


主模型：

- ReV Animated v1.1: [https://civitai.com/models/7371/rev-animated?modelVersionId=19575](https://civitai.com/models/7371/rev-animated?modelVersionId=19575)

LoRA 模型：

- DDicon_lora: [https://civitai.com/models/38558/ddiconlora](https://civitai.com/models/38558/ddiconlora)


提示词：

```
(masterpiece, top quality, best quality), ( open book:1.4) , DDicon, clean background, <lora:DDicon_v1_lora:1>
```

负向提示词： 

```
Negative prompt: (worst quality, low quality:2),
```

其他参数：

```
Steps: 35, ENSD: 31337, Size: 600x600, Seed: 124748696, Model: revAnimated_v11, Sampler: DPM++ 2S a Karras, CFG scale: 8, Clip skip: 2, Model hash: d725be5d18, Denoising strength: 0.3, SD upscale overlap: 64, SD upscale upscaler: R-ESRGAN 4x+
```


**4）示例 4**


![B 端元素风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-020.jpg)


主模型：

- Product Design eddiemauro 2.0: [https://civitai.com/models/23893/product-design-minimalism-eddiemauro?modelVersionId=85831](https://civitai.com/models/23893/product-design-minimalism-eddiemauro?modelVersionId=85831)

LoRA 模型：

- DDicon_lora: [https://civitai.com/models/38558/ddiconlora](https://civitai.com/models/38558/ddiconlora)


提示词：

```
best quality,many details,4k,blender,octane render,C4D,transparent glass texture,DDicon,blue|green,frosted glass,transparent technology sense,industrial design,white background,studio lighting,sunshine,flat,minimal,quasi-object,axisymmetric,Data,file,shiled,<lora:DDicon_v2_lora:1>,
```

负向提示词： 

```
Negative prompt: lowres, bad anatomy, ((bad hands)), (worst quality:2), (low quality:2), (normal quality:2), paintings, sketches, lowres, bad anatomy, bad hands, text, error, missing fingers,
```

其他参数：

```
Steps: 28, ENSD: 31337, Size: 512x512, Seed: 122654496, Model: productDesign_eddiemauro20, model: control_v11f1e_sd15_tile [a371b31b], weight: 1, Sampler: Euler a, CFG scale: 8, Clip skip: 2, Model hash: db58f07060, resize mode: Crop and Resize, control mode: Balanced, "preprocessor: tile_resample, Hires upscale: 2, pixel perfect: True, Hires upscaler: R-ESRGAN 4x+, starting/ending: (0, 1), Denoising strength: 0.7, preprocessor params: (512, 1, 200)"
```







## 6、2.5D 和 3D 风格


**1）示例 1**


![2.5D 和 3D 风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-021.jpg)


主模型：

- ReV Animated v1.2.2: [https://civitai.com/models/7371/rev-animated?modelVersionId=46846](https://civitai.com/models/7371/rev-animated?modelVersionId=46846)

LoRA 模型：

- 3D rendering style 3DMM_V12: [https://civitai.com/models/73756/3d-rendering-style?modelVersionId=107366](https://civitai.com/models/73756/3d-rendering-style?modelVersionId=107366)




提示词：

```
1girl, tohsaka rin, solo, long hair, sweater, red sweater, looking at viewer, blue background, black hair, simple background, two side up, turtleneck, blue eyes, lips, closed mouth, ribbon, hair ribbon, bangs, turtleneck sweater, upper body, parted bangs, black ribbon, ribbed sweater, twintails, nose,
```

负向提示词： 

```
Negative prompt: badhandv4, paintings, sketches, (worst qualit:2), (low quality:2), (normal quality:2), lowers, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, (outdoor:1.6), manboobs, (backlight:1.2), double navel, muted arms, hused arms, neck lace, analog, analog effects, (sunglass:1.4), nipples, nsfw, bad architecture, watermark, (mole:1.5),
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 512x768, Seed: 2987955084, Model: revAnimated_v122, Version: v1.3.2, Sampler: Euler a, CFG scale: 7, Clip skip: 2, Model hash: 4199bcdd14, Hires steps: 20, Hires upscale: 1.5, AddNet Enabled: True, AddNet Model 1: 3DMM_V12(3e0ce2773235), Hires upscaler: 4x-UltraSharp, AddNet Module 1: LoRA, AddNet Weight A 1: 1, AddNet Weight B 1: 1, Denoising strength: 0.3
```




**2）示例 2**


![2.5D 和 3D 风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-022.jpg)


主模型：

- ReV Animated v1.1: [https://civitai.com/models/7371/rev-animated?modelVersionId=19575](https://civitai.com/models/7371/rev-animated?modelVersionId=19575)

LoRA 模型：

- blindbox v1 mix: [https://civitai.com/models/25995/blindbox?modelVersionId=32988](https://civitai.com/models/25995/blindbox?modelVersionId=32988)


提示词：

```
(masterpiece),(best quality),(ultra-detailed), (full body:1.2),
1girl,chibi,cute, smile, open mouth,
flower, outdoors, playing guitar, music, beret, holding guitar, jacket, blush, tree, :3, shirt, short hair, cherry blossoms, green headwear, blurry, brown hair, blush stickers, long sleeves, bangs, headphones, black hair, pink flower,
(beautiful detailed face), (beautiful detailed eyes),
 <lora:blindbox_v1_mix:1>,
```

负向提示词： 

```
Negative prompt: (low quality:1.3), (worst quality:1.3)
```

其他参数：

```
Steps: 28, ENSD: 31337, Size: 576x768, Seed: 941189224, Model: revAnimated_v11, Sampler: Euler a, CFG scale: 7, Clip skip: 2, Model hash: d725be5d18, Hires upscale: 1.5, Hires upscaler: Latent (bicubic antialiased), Denoising strength: 0.5
```




**3）示例 3**


![2.5D 和 3D 风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-023.jpg)


主模型：

- ReV Animated v1.1: [https://civitai.com/models/7371/rev-animated?modelVersionId=19575](https://civitai.com/models/7371/rev-animated?modelVersionId=19575)

LoRA 模型：

- blindbox v1 mix: [https://civitai.com/models/25995/blindbox?modelVersionId=32988](https://civitai.com/models/25995/blindbox?modelVersionId=32988)


提示词：

```
(masterpiece),(best quality),(ultra-detailed), (full body:1.2), 1boy,chibi, smile,(beautiful detailed face), (beautiful detailed eyes), (Simple and clean style_1.5), tactical vest, long overcoat, ballistic gear, black gear, (brown sweater), <lora:blindbox_V1Mix:1>
```

负向提示词： 

```
Negative prompt: (low quality:1.3), (worst quality:1.3),
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 512x744, Seed: 4263564851, Model: revAnimated_v11, Sampler: DPM++ 2M Karras, CFG scale: 7, Clip skip: 2, Model hash: d725be5d18
```



**4）示例 4**


![2.5D 和 3D 风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-024.jpg)


主模型：

- ReV Animated v1.1: [https://civitai.com/models/7371/rev-animated?modelVersionId=19575](https://civitai.com/models/7371/rev-animated?modelVersionId=19575)

LoRA 模型：

- blindbox v1 mix: [https://civitai.com/models/25995/blindbox?modelVersionId=32988](https://civitai.com/models/25995/blindbox?modelVersionId=32988)


提示词：

```
(masterpiece),(best quality),(ultra-detailed), (full body:1.2),
panda,bamboo forest, cute, mascot,
flowers, outdoors,  blush, tree, :3,  cherry blossoms, green headwear, blurry,  blush stickers, 
(beautiful detailed face), (beautiful detailed eyes),
<lora:blindbox_V1Mix:1>
```

负向提示词： 

```
Negative prompt: (low quality:1.3), (worst quality:1.3)
```

其他参数：

```
Steps: 28, ENSD: 31337, Size: 576x768, Seed: 1504665324, Model: revAnimated_v11, Sampler: Euler a, CFG scale: 7, Clip skip: 2, Model hash: d725be5d18, Hires upscale: 1.5, Hires upscaler: Latent (bicubic antialiased), Denoising strength: 0.5
```



## 7、建筑设计风格



**1）示例 1**


![建筑设计风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-025.jpg)


主模型：

- Stable Diffusion v1.5: [https://huggingface.co/runwayml/stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5)

LoRA 模型：

- architectural design sketches with markers: [https://civitai.com/models/34384/architectural-design-sketches-with-markers?modelVersionId=40665](https://civitai.com/models/34384/architectural-design-sketches-with-markers?modelVersionId=40665)


提示词：

```
Modern architecture,handsketch,Marker architectural illustration <lora:2700marker:1>,outside,facade
```

负向提示词： 

```
Negative prompt: lowres, text, error, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry, artist name
```

其他参数：

```
Steps: 50, Size: 700x512, Seed: 1743923072, Model: realistic-archi-sd15_v3, Sampler: Euler a, CFG scale: 7, Clip skip: 2, Model hash: 974f353810, ControlNet-0 Model: control_canny-fp16 [e3fe7712], ControlNet-0 Module: none, ControlNet-0 Weight: 0.95, ControlNet-0 Enabled: True, ControlNet-0 Guidance End: 1, ControlNet-0 Guidance Start: 0
```




**2）示例 2**


![建筑设计风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-026.jpg)


主模型：

- architecture_Exterior_SDlife_Chiasedamme: [https://civitai.com/models/114612/architectureexteriorsdlifechiasedamme?modelVersionId=123908](https://civitai.com/models/114612/architectureexteriorsdlifechiasedamme?modelVersionId=123908)

LoRA 模型：

- house_architecture_Exterior_SDlife_Chiasedamme: [https://civitai.com/models/115932/housearchitectureexteriorsdlifechiasedamme?modelVersionId=125485](https://civitai.com/models/115932/housearchitectureexteriorsdlifechiasedamme?modelVersionId=125485)

提示词：

```
dvArchModern, 85mm, f1.8, portrait, photo realistic, hyperrealistic, orante, super detailed, intricate, dramatic, sunlight lighting, shadows, high dynamic range,  house, masterpiece,best quality,(8k, RAW photo:1.2),(( ultra realistic)), modernvilla, blackandwhite, trees, vine, architecture, building, cloud, vivid colour, masterpiece,best quality,super detailed,realistic,photorealistic, 8k, sharp focus, a photo of a building  <lora:house_architecture_Exterior_SDlife_Chiasedamme:0.7> wood,
```

负向提示词： 

```
Negative prompt: signature, soft, blurry, drawing, sketch, poor quality, ugly, text, type, word, logo, pixelated, low resolution, saturated, high contrast, oversharpened
```

其他参数：

```
Steps: 25, ENSD: 31337, Size: 512x768, Seed: 2001349004, Model: architecture_Exterior_SDlife_Chiasedamme_V4.0, model: control_v11p_sd15_scribble [d4ba51ff], weight: 1, Version: v1.4.1, Sampler: Euler a, CFG scale: 12, Clip skip: 2, Model hash: 03b2d23370, resize mode: Crop and Resize, control mode: Balanced, "preprocessor: scribble_xdog, Hires upscale: 2, pixel perfect: True, Hires upscaler: 4x-UltraSharp, starting/ending: (0, 1), Denoising strength: 0.55, preprocessor params: (512, 32, 200)", "house_architecture_Exterior_SDlife_Chiasedamme: 1ee09b264ac5"
```




**3）示例 3**


![建筑设计风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-027.jpg)


主模型：

- architecture_Exterior_SDlife_Chiasedamme: [https://civitai.com/models/114612/architectureexteriorsdlifechiasedamme?modelVersionId=123908](https://civitai.com/models/114612/architectureexteriorsdlifechiasedamme?modelVersionId=123908)

LoRA 模型：

- house_architecture_Exterior_SDlife_Chiasedamme: [https://civitai.com/models/115932/housearchitectureexteriorsdlifechiasedamme?modelVersionId=125485](https://civitai.com/models/115932/housearchitectureexteriorsdlifechiasedamme?modelVersionId=125485)

提示词：

```
dvArchModern, 85mm, f1.8, portrait, photo realistic, hyperrealistic, orante, super detailed, intricate, dramatic, sunlight lighting, shadows, high dynamic range,  house, masterpiece,best quality,(8k, RAW photo:1.2),(( ultra realistic)), modernvilla, blackandwhite, trees, vine, architecture, building, cloud, vivid colour, masterpiece,best quality,super detailed,realistic,photorealistic, 8k, sharp focus, a photo of a building  <lora:house_architecture_Exterior_SDlife_Chiasedamme:0.7>
```

负向提示词： 

```
Negative prompt: signature, soft, blurry, drawing, sketch, poor quality, ugly, text, type, word, logo, pixelated, low resolution, saturated, high contrast, oversharpened
```

其他参数：

```
Steps: 25, ENSD: 31337, Size: 512x768, Seed: 1189999251, Model: architecture_Exterior_SDlife_Chiasedamme_V4.0, model: control_v11p_sd15_scribble [d4ba51ff], weight: 1, Version: v1.4.1, Sampler: Euler a, CFG scale: 12, Clip skip: 2, Model hash: 03b2d23370, resize mode: Crop and Resize, control mode: Balanced, "preprocessor: scribble_xdog, Hires upscale: 2, pixel perfect: True, Hires upscaler: 4x-UltraSharp, starting/ending: (0, 1), Denoising strength: 0.55, preprocessor params: (512, 32, 200)", "house_architecture_Exterior_SDlife_Chiasedamme: 1ee09b264ac5"
```


**4）示例 4**


![建筑设计风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-028.jpg)


主模型：

- XSarchitecturalV3: [https://civitai.com/models/49516/xsarchitecturalv3commercialbuildingrendering?modelVersionId=54092](https://civitai.com/models/49516/xsarchitecturalv3commercialbuildingrendering?modelVersionId=54092)

LoRA 模型：

- XSArchi_127: [https://civitai.com/models/100287/xsarchi127neo-sci-fi?modelVersionId=107343](https://civitai.com/models/100287/xsarchi127neo-sci-fi?modelVersionId=107343)


提示词：

```
a library with a sofa and book shelves,, (masterpiece),(high quality), best quality, real,(realistic), super detailed, (full detail),(4k),8k,scenery, sky, outdoors, day, building, water, blue sky, watercraft, boat, reflection, flower, signature<lora:XSArchi_127:1>
```

负向提示词： 

```
Negative prompt: (normal quality), (low quality), (worst quality), paintings, sketches,fog,signature,soft, blurry,drawing,sketch, poor quality, uply text,type, word, logo, pixelated, low resolution.,saturated,high contrast, oversharpened,dirt,
```

其他参数：

```
Steps: 30, Size: 512x768, Seed: 2117536779, Model: XSarchitecturalV3Commercialbuildingrendering, Sampler: Euler a, CFG scale: 7.5, Model hash: 7bf72c55d3, Hires upscale: 2, Hires upscaler: ESRGAN_4x, Denoising strength: 0.4
```






## 8、室内设计风格


**1）示例 1**


![室内设计风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-029.jpg)


主模型：

- XSarchitectural-InteriorDesign-ForXSLora: [https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722](https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722)


提示词：

```
(masterpiece),(high quality), best quality, real,(realistic), super detailed, (full detail),(4k),8k,interior,interior desgin,living room,white,red and green
```

负向提示词： 

```
Negative prompt: (normal quality), (low quality), (worst quality), paintings, sketches,
```

其他参数：

```
Steps: 30, ENSD: 31337, Size: 704x512, Seed: 786679420, Model: XSarchitecturalV11InteriorDesgin, Sampler: Euler a, CFG scale: 7, Model hash: 631eea1a0e, Hires upscale: 2, Hires upscaler: R-ESRGAN 4x+, Denoising strength: 0.3
```





**2）示例 2**


![室内设计风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-030.jpg)


主模型：

- XSarchitectural-InteriorDesign-ForXSLora: [https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722](https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722)




提示词：

```
(masterpiece),(high quality), best quality, real,(realistic), super detailed, (full detail),(4k),8k,interior,Living room,white
```

负向提示词： 

```
Negative prompt: (normal quality), (low quality), (worst quality), paintings, sketches,
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 704x512, Seed: 2114630674, Model: XSarchitecturalV11InteriorDesgin, Sampler: Euler a, CFG scale: 7, Model hash: 631eea1a0e, Hires upscale: 2, Hires upscaler: R-ESRGAN 4x+, Denoising strength: 0.3
```




**3）示例 3**


![室内设计风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-031.jpg)


主模型：

- XSarchitectural-InteriorDesign-ForXSLora: [https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722](https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722)


LoRA 模型：

- XSarchitectural-38InteriorForBedroom: [https://civitai.com/models/45262/xsarchitectural-38interiorforbedroom?modelVersionId=49877](https://civitai.com/models/45262/xsarchitectural-38interiorforbedroom?modelVersionId=49877)


提示词：

```
(masterpiece),(high quality), best quality, real,(realistic), super detailed, (full detail),(4k),8k,interior,living room,green <lora:XSarchitectural-36ChineseInteriorDecoration:1>
```

负向提示词： 

```
Negative prompt: (normal quality), (low quality), (worst quality), paintings, sketches,
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 512x512, Seed: 3093110488, Model: XSarchitecturalV11InteriorDesgin, Sampler: Euler a, CFG scale: 7, Model hash: 631eea1a0e, Hires upscale: 2.5, Hires upscaler: R-ESRGAN 4x+, Denoising strength: 0.3
```





**4）示例 4**


![室内设计风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-032.jpg)


主模型：

- XSarchitectural-InteriorDesign-ForXSLora: [https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722](https://civitai.com/models/28112/xsarchitectural-interiordesign-forxslora?modelVersionId=50722)


LoRA 模型：

- XSarchitectural-38InteriorForBedroom: [https://civitai.com/models/45262/xsarchitectural-38interiorforbedroom?modelVersionId=49877](https://civitai.com/models/45262/xsarchitectural-38interiorforbedroom?modelVersionId=49877)


提示词：

```
(masterpiece),(high quality), best quality, real,(realistic), super detailed, (full detail),(4k),8k,interior,(office),black<lora:XSarchitectural-36ChineseInteriorDecoration:1>
```

负向提示词： 

```
Negative prompt: (normal quality), (low quality), (worst quality), paintings, sketches,
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 512x512, Seed: 837796741, Model: XSarchitecturalV11InteriorDesgin, Sampler: Euler a, CFG scale: 7, Model hash: 631eea1a0e, Hires upscale: 2.5, Hires upscaler: R-ESRGAN 4x+, Denoising strength: 0.3
```


## 9、剪纸风格



**1）示例 1**


![剪纸风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-033.jpg)


主模型：

- Stable Diffusion v1.5: [https://huggingface.co/runwayml/stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5)

LoRA 模型：

- 剪纸: [https://civitai.com/models/14892](https://civitai.com/models/14892)


提示词：

```
black,<lora:剪纸:1.2>
```

负向提示词： 

```
Negative prompt: cxd,EasyNegative
```

其他参数：

```
Steps: 20, Seed: 52084194, Sampler: DPM++ SDE Karras, CFG scale: 7
```


**2）示例 2**


![剪纸风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-034.jpg)


主模型：

- Stable Diffusion v1.5: [https://huggingface.co/runwayml/stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5)

LoRA 模型：

- 剪纸: [https://civitai.com/models/14892](https://civitai.com/models/14892)


提示词：

```
sea,ocean,fish,boat, <lora:剪纸:1>, papercut,vivid,sky,cloud,
```

负向提示词： 

```
Negative prompt: easynegative,
```

其他参数：

```
Steps: 20, Seed: 3571779252, Sampler: DPM++ 2M Karras, CFG scale: 7
```


**3）示例 3**


![剪纸风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-035.jpg)


主模型：

- Stable Diffusion v1.5: [https://huggingface.co/runwayml/stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5)

LoRA 模型：

- 剪纸: [https://civitai.com/models/14892](https://civitai.com/models/14892)

提示词：

```
transparent,water,green,<lora:剪纸:1.2> ,1 girl
```

负向提示词： 

```
Negative prompt: cxd,EasyNegative
```

其他参数：

```
Steps: 20, Seed: 4223434982, Sampler: DPM++ SDE Karras, CFG scale: 7
```



**4）示例 4**


![剪纸风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-036.jpg)


主模型：

- Stable Diffusion v1.5: [https://huggingface.co/runwayml/stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5)

LoRA 模型：

- 剪纸: [https://civitai.com/models/14892](https://civitai.com/models/14892)


提示词：

```
blue,(1 girl:1.2),<lora:剪纸:1>
```

负向提示词： 

```
Negative prompt: cxd,EasyNegative
```

其他参数：

```
Steps: 20, Seed: 2369163141, Sampler: DPM++ SDE Karras, CFG scale: 7
```


## 10、线稿风格


**1）示例 1**


![线稿风格 1](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-037.jpg)


主模型：

- AbyssOrangeMix3: [https://civitai.com/models/9942/abyssorangemix3-aom3?modelVersionId=17233](https://civitai.com/models/9942/abyssorangemix3-aom3?modelVersionId=17233)

LoRA 模型：

- Anime Lineart/Manga-like: [https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907](https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907)


提示词：

```
medieval knight, lineart, monochrome, <lora:animeLineartMangaLike_v30MangaLike:1>
```

负向提示词： 

```
Negative prompt: bad_prompt_version2, bad-artist-anime, easynegative
```

其他参数：

```
Steps: 30, Size: 512x768, Seed: 3325367275, Model: abyssorangemix3AOM3_aom3a1b, Sampler: DPM++ 2M Karras, CFG scale: 7, Model hash: 5493a0ec49, Hires upscale: 2, Hires upscaler: R-ESRGAN 4x+ Anime6B, Denoising strength: 0.4
```



**2）示例 2**


![线稿风格 2](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-038.jpg)


主模型：

- Stable Diffusion v1.5: [https://huggingface.co/runwayml/stable-diffusion-v1-5](https://huggingface.co/runwayml/stable-diffusion-v1-5)

LoRA 模型：

- Anime Lineart/Manga-like: [https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907](https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907)


提示词：

```
masterpiece, best quality, 1girl, solo, looking away, expressionless, from side, white dress, colorful, floral background, rain, lake, fog, barefoot, sitting on water, from top, lineart, monochrome, <lora:animeoutlineV4_16:1>
```

负向提示词： 

```
Negative prompt: EasyNegative, badhandv4
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 512x768, Seed: 2845465797, "mode: Horizontal, Sampler: DPM++ 2M Karras, Use base: False, CFG scale: 7, Clip skip: 2, Base ratio: 0.2, Model hash: 6e430eb514, Use common: False, Hires steps: 10, Use N-common: False", divide ratio: 1,1, Hires upscale: 2.5, Hires upscaler: SwinIR_4x, Denoising strength: 0.5
```



**3）示例 3**


![线稿风格 3](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-039.jpg)


主模型：

- Deliberate v1.1: [https://civitai.com/models/4823/deliberate?modelVersionId=5616](https://civitai.com/models/4823/deliberate?modelVersionId=5616)

LoRA 模型：

- Anime Lineart/Manga-like: [https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907](https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907)


提示词：

```
lineart, monochrome, hyperdetailed, futuristic city, neon lights, a man standing on wasteland, snow in the background, sunny day, cold color palette, (Realistic:1.5), hyper-detalied, 8k, cinematic light, volumetric light <lora:animeLineartStyle_v20Offset:1.5>
```

负向提示词： 

```
Negative prompt: deformed, bad anatomy, disfigured, poorly drawn face, mutation, mutated, extra limb, ugly, disgusting, poorly drawn hands, missing limb, floating limbs, disconnected limbs, malformed hands, blurry, ((((mutated hands and fingers)))), watermark, watermarked, oversaturated, censored, distorted hands, amputation, missing hands, obese, doubled face, double hands, asian, b&w, black and white, sepia, deformed, bad anatomy, disfigured, poorly drawn face, mutation, mutated, extra limb, ugly, disgusting, poorly drawn hands, missing limb, floating limbs, disconnected limbs, malformed hands, blurry, ((((mutated hands and fingers)))), watermark, watermarked, oversaturated, censored, distorted hands, amputation, missing hands, obese, doubled face, double hands, sepia
```

其他参数：

```
Steps: 30, ENSD: 31337, Size: 600x1000, Seed: 3270187169, Model: deliberate_v11, Sampler: DPM++ 2M Karras, CFG scale: 7, Model hash: d8691b4d16, ControlNet Model: control_canny-fp16 [e3fe7712], ControlNet Module: canny, ControlNet Weight: 1, ControlNet Enabled: True, ControlNet Guidance End: 1, ControlNet Guidance Start: 0
```


**4）示例 4**


![线稿风格 4](assets/resource/aigc-tutorial/sd-use-style-models/sd-use-style-models-040.jpg)


主模型：

- 3D Thick coated UP V4: [https://civitai.com/models/13747/3d-thick-coated?modelVersionId=25080](https://civitai.com/models/13747/3d-thick-coated?modelVersionId=25080)

LoRA 模型：

- Anime Lineart/Manga-like: [https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907](https://civitai.com/models/16014/anime-lineart-manga-like-style?modelVersionId=28907)


提示词：

```
(upper body: 1.5),(white background:1.4), ((tachi-e)), original,(illustration:1.1),(best quality),(masterpiece:1.1),(extremely detailed CG unity 8k wallpaper:1.1), (colorful:0.9),(panorama shot:1.4),(full body:1.05),(solo:1.2), (ink splashing),(color splashing),((watercolor)), clear sharp focus,{ 1girl standing },((chinese style )),(flowers,woods),outdoors,rocks, looking at viewer, make happy expressions,soft smile,pure,beautyfull detailed face and eyes,beautyfull intricacy clothing decorative pattern details, black hair,black eyes,  <lora:animeLineartMangaLike_v30MangaLike:1>
```

负向提示词： 

```
Negative prompt: paintings, sketches, fingers, (worst quality:2), (low quality:2), (normal quality:2), lowres, normal quality, ((monochrome)), ((grayscale)), skin spots, acnes, skin blemishes, age spot, (outdoor:1.6), backlight,(ugly:1.331), (duplicate:1.331), (morbid:1.21), (mutilated:1.21), (tranny:1.331), mutated hands, (poorly drawn hands:1.5), blurry, (bad anatomy:1.21), (bad proportions:1.331), extra limbs, (disfigured:1.331), (more than 2 nipples:1.331), (missing arms:1.331), (extra legs:1.331), (fused fingers:1.61051), (too many fingers:1.61051), (unclear eyes:1.331), lowers, bad hands, missing fingers, extra digit, (futa:1.1),bad hands, missing fingers
```

其他参数：

```
Steps: 20, ENSD: 31337, Size: 512x768, Seed: 2769086267, Model: 3dThickCoated_3dThickCoatedV4, Sampler: DPM++ 2M, CFG scale: 2, Clip skip: 2, Model hash: b3854b01ed
```


