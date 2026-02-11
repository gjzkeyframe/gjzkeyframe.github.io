---
title: AI 魔法提示词集锦第 7 期：创意融合与场景构建
description: 这期我们来展示使用提示词让 AI 进行创意融合与场景构建。
author: Keyframe
date: 2026-02-02 08:08:08 +0800
categories: [AI 提示词]
tags: [AI 提示词, AI, AIGC, 视频, 照片, GIF, Gemini, Nano Banana, Wan, QWen, 人物写真]
pin: false
math: false
mermaid: false
image:
  path: assets/resource/aigc-prompt/aigc-prompt-7-1.jpeg
  alt: AIGC Image
---



## 1、K-Pop 视觉美学四宫格

Gemini Nano Banana Pro 出图示例：

![K-Pop 四宫格](assets/resource/aigc-prompt/aigc-prompt-7-1.jpeg)



**Prompt：**

```json
{
  "system_architecture": {
    "version": "KR_Idol_PureDesire_v2",
    "project_id": "Seamless_Life_Collage",
    "framework": {
      "format": "2x2 Seamless Grid (Zero Gap)",
      "aspect_ratio": "3:4",
      "composite_mode": "Perfectly fused edges / No white borders / No gutters",
      "engine_mode": "Analog Film Emulation / K-Pop Visual Aesthetic"
    }
  },
  "biometric_anchor": {
    "subject_desc": "Korean K-pop idol, visual member, pure and sexy aura",
    "identity_logic": {
      "consistency_lock": "100% facial feature match across quadrants",
      "vibe_blend": "Innocent face + Alluring vibe (Pure Desire style)",
      "surface_parameters": [
        "Pale glass skin",
        "Soft peach blush",
        "Slight sweat/glow for realism",
        "Film grain texture"
      ]
    }
  },
  "quadrant_temporal_logic": {
    "quadrant_01": {
      "id": "MORNING_BED",
      "perspective": "POV Boyfriend / High angle looking down",
      "state_logic": {
        "facial": "Messy hair, rubbing one eye, lips slightly parted, defenseless look",
        "pose": "Lying on stomach, propped up on elbows, looking up lazily",
        "apparel": "Oversized white boyfriend shirt (one shoulder slipping off)"
      },
      "environmental_logic": {
        "location": "White bed sheets",
        "lighting": "Hazy overexposed morning sunlight",
        "style": "Dreamy, soft focus, intimate"
      }
    },
    "quadrant_02": {
      "id": "PRACTICE_SWEAT",
      "perspective": "Mirror Selfie (Floor level)",
      "state_logic": {
        "facial": "Biting lower lip playfully, wink, flushed cheeks from exercise",
        "pose": "Sitting with legs stretched out towards mirror, phone covering half face",
        "apparel": "Tight crop top + Grey sweatpants (waistband folded)",
        "accessory": "Cute stickers on phone case"
      },
      "environmental_logic": {
        "location": "Dance Studio",
        "lighting": "Mixed fluorescent and window light",
        "style": "Authentic grit, slight motion blur"
      }
    },
    "quadrant_03": {
      "id": "STREET_SUMMER_VIBE",
      "perspective": "High-Angle Candid (The 'Puppy' Angle)",
      "state_logic": {
        "facial": "Looking up at camera with wide innocent eyes, cooling cheek with a cold glass bottle",
        "pose": "Squatting/Crouching next to a retro vending machine, knees together (cute but alluring silhouette)",
        "apparel": "Short pleated skirt + fitted camisole"
      },
      "environmental_logic": {
        "location": "Street corner / Vending Machine",
        "lighting": "Golden hour sunlight hitting the face, lens flare",
        "style": "Vibrant colors, nostalgic summer film vibe, visually distinct"
      }
    },
    "quadrant_04": {
      "id": "NIGHT_FLASH_FUN",
      "perspective": "Direct Flash Close-up",
      "state_logic": {
        "facial": "Scrunching nose, messy eating (cream/sauce on lip), playful eye contact",
        "pose": "Leaning in very close to the lens, holding a snack/pizza slice",
        "apparel": "Silk pajamas or comfy homewear"
      },
      "environmental_logic": {
        "location": "Dimly lit living room",
        "lighting": "Harsh direct camera flash",
        "style": "Retro disposable camera aesthetic, high contrast"
      }
    }
  },
  "technical_pipeline": {
    "optical_simulation": {
      "film_stock": "Kodak Portra 400 (Warm & Grainy)",
      "lens": "35mm Prime",
      "effects": "Halation, Film Grain, Light Leaks"
    },
    "prohibition_matrix": {
      "elements": "NO TEXT, NO WATERMARKS, NO BORDERS, NO FRAMES, NO SPLIT LINES",
      "quality": "No bad anatomy, no blurry face",
      "vibe": "No western makeup style, no overly sexualized poses (keep it subtle/suggestive)"
    }
  }
}
```

**中文版：**

```json
{
  "system_architecture": {
    "版本": "KR_Idol_PureDesire_v2",
    "project_id": "Seamless_Life_Collage",
    "框架": {
      "格式": "2x2 无缝网格（零间隙）",
      "aspect_ratio": "3:4",
      "composite_mode": "完美融合的边缘 / 无白边 / 无间距",
      "engine_mode": "模拟胶片模拟/K-Pop视觉美学"
    }
  },
  "biometric_anchor": {
    "subject_desc": "韩国流行偶像，门面担当，拥有清纯性感的气质",
    "identity_logic": {
      "consistency_lock": "各象限面部特征匹配度达到100%",
      "vibe_blend": "纯真面目 + 魅惑气质（纯粹欲望风格）",
      "surface_parameters": [
        "苍白的玻璃皮肤",
        "柔和的蜜桃腮红",
        "略微出汗/泛红，以增加真实感",
        "胶片颗粒纹理"
      ]
    }
  },
  "quadrant_temporal_logic": {
    "quadrant_01": {
      "id": "MORNING_BED",
      "视角": "男友视角/俯视高角度",
      "state_logic": {
        "面部表情": "头发凌乱，揉着一只眼睛，嘴唇微张，一副无助的样子",
        "姿势": "俯卧，用手肘支撑身体，懒洋洋地抬头看",
        "服装": "超大号白色男友衬衫（一侧肩膀滑落）"
      },
      "environmental_logic": {
        "地点": "白色床单",
        "光线": "朦胧过曝的晨光",
        "风格": "梦幻、柔焦、亲密"
      }
    },
    "quadrant_02": {
      "id": "PRACTICE_SWEAT",
      "视角": "镜子自拍（地面视角）",
      "state_logic": {
        "面部表情": "俏皮地咬着下唇，眨眨眼，运动后脸颊泛红",
        "姿势": "双腿伸直对着镜子坐着，手机遮住半张脸",
        "服装": "紧身露脐上衣 + 灰色运动裤（腰带折叠）",
        "配件": "手机壳上的可爱贴纸"
      },
      "environmental_logic": {
        "地点": "舞蹈工作室",
        "照明": "荧光灯和窗户光线混合",
        "风格": "真实粗粝，略带动态模糊"
      }
    },
    "quadrant_03": {
      "id": "STREET_SUMMER_VIBE",
      "视角": "高角度抓拍（'小狗'视角）",
      "state_logic": {
        "面部表情": "睁大无辜的眼睛望着镜头，用冰冷的玻璃瓶给脸颊降温",
        "姿势": "蹲/屈膝站在一台复古自动售货机旁，双膝并拢（可爱而迷人的轮廓）",
        "服装": "短款百褶裙+修身吊带背心"
      },
      "environmental_logic": {
        "位置": "街角/自动售货机",
        "光线": "日落时分的阳光照射在脸上，镜头光晕",
        "风格": "色彩鲜艳，充满怀旧夏日电影氛围，视觉效果独特"
      }
    },
    "quadrant_04": {
      "id": "NIGHT_FLASH_FUN",
      "视角": "直接闪光特写",
      "state_logic": {
        "面部表情": "皱鼻子，吃东西狼狈（嘴唇上沾有奶油/酱汁），顽皮的眼神交流",
        "姿势": "身体前倾，靠近镜头，手里拿着一块零食/披萨",
        "服装": "丝绸睡衣或舒适的家居服"
      },
      "environmental_logic": {
        "地点": "光线昏暗的客厅",
        "照明": "强烈的相机直射闪光灯",
        "风格": "复古一次性相机美学，高对比度"
      }
    }
  },
  "technical_pipeline": {
    "optical_simulation": {
      "film_stock": "柯达 Portra 400（暖色调颗粒感）",
      "镜头": "35mm 定焦镜头",
      "效果": "光晕、胶片颗粒、漏光"
    },
    "禁止矩阵": {
      "元素": "无文字、无水印、无边框、无框架、无分割线",
      "质量": "没有糟糕的解剖结构，没有模糊的脸部",
      "氛围": "不要西式妆容，不要过于性感的姿势（保持含蓄/暗示性）"
    }
  }
}
```

**出处：** [https://x.com/BubbleBrain/status/2009112785980985688](https://x.com/BubbleBrain/status/2009112785980985688)


---

**本文所介绍的 AI 人物生成，也可以使用 FaceXSwap 端侧离线换脸生成。**

![FaceXSwap 换脸生成作品](assets/resource/aigc-product/facexswap_preview_short.gif)
_FaceXSwap 换脸生成作品_

---



## 2、手臂搭在巨大的 3D 汤姆猫身上

Gemini Nano Banana Pro 出图示例：

![汤姆猫](assets/resource/aigc-prompt/aigc-prompt-7-2.jpeg)



**Prompt：**

```
"image_generation": {
  "quality": "hyper-realistic",
  "face": {
    "preserve_original": true,
    "reference_match": true
  },
  "subject": {
    "description": "A stylish person with the same face.",
    "clothing": {
      "Top": {
        "type": "knitted sweater",
        "color": "light grey (Tom Theme)"
      },
      "pants": {
        "type": "high-waisted jeans",
        "color": "denim blue"
      },
      "shoes": {
        "type": "high-top sneakers",
        "color": "white"
      }
    },
    "pose": "standing with arm around a giant 3D Tom while Jerry sits on Tom's shoulder",
    "expression": "fun, mischievous"
  },
  "character_element": {
    "name": "Tom & Jerry",
    "type": "3D photorealistic duo",
    "interaction": "Tom poses confidently, Jerry looks playful"
  },
  "environment": {
    "background": "clean grey-blue backdrop"
  }
}
```

**中文版：**

```
"image_generation": {
  "质量": "超逼真",
  "face": {
    "preserve_original": true,
    "reference_match": true
  },
  "主题": {
    "描述": "一个穿着时尚但长相雷同的人。",
    "服装": {
      "上衣": {
        "类型": "针织毛衣",
        "颜色": "浅灰色（汤姆主题）"
      },
      "裤子": {
        "类型": "高腰牛仔裤",
        "颜色": "牛仔蓝"
      },
      "鞋子": {
        "类型": "高帮运动鞋",
        "颜色": "白色"
      }
    },
    "姿势": "手臂搭在巨大的3D汤姆猫身上，而杰瑞坐在汤姆的肩膀上",
    "表达方式": "有趣，调皮"
  },
  "character_element": {
    "name": "汤姆和杰瑞",
    "type": "3D 逼真双人组",
    "interaction": "汤姆自信地摆姿势，杰瑞看起来很顽皮"
  },
  "环境": {
    "background": "干净的灰蓝色背景"
  }
}
```

**出处：** [https://x.com/ZaraIrahh/status/2005816508602278010](https://x.com/ZaraIrahh/status/2005816508602278010)



## 3、一张超写实的竖屏照片

Gemini Nano Banana Pro 出图示例：

![竖屏照片](assets/resource/aigc-prompt/aigc-prompt-7-3.jpeg)



**Prompt：**

```
# 图片复刻元提示词 (Image Reproduction Meta-Prompt)

## 1. 角色指定 (Role)
你是一位**资深人像摄影大师 (Senior Portrait Photographer)** 和 **光影构图专家**。你擅长捕捉日常生活中的自然瞬间（Candid Moments），精通室内布光与景深控制，能够完美复刻"男友视角"的社交媒体风格照片。

## 2. 图片结构与框架 (Structure & Frame)
* **画幅比例:** 9:16 (竖屏全画幅)
* **构图模式:** 近景人像 (Medium Close-up)，人物占据画面前景左侧 60% 区域。
* **核心锚点:** 
  * 前景：浅色木质圆桌边缘（切过画面左下角）。
  * 中景：人物上半身，特别是面部和托腮的手臂。
  * 背景：虚化的咖啡店柜台与人群。
* **文字处理:** 本图无UI文字框。需在画面中生成的自然文字为人物左臂衣袖上的 "alo" 品牌标签。

## 3. 图片主题内容生成 Workflow
**Step 1: 场景构建 (Scene Setup)**
* 设定环境为现代繁忙的咖啡店内部。
* 天花板：裸露的工业风管道，安装有轨道射灯。
* 背景：远处有模糊的服务柜台（红色菜单板为特征）和排队的深色衣着路人。

**Step 2: 主体刻画 (Subject Definition)**
* 生成一名[可爱短发亚洲女性]。
* 发型：棕色短发，空气刘海。
* 着装：穿着黑色半拉链立领Fleece材质卫衣，质感柔软厚实。
* **关键细节:** 左大臂处必须有一个清晰的黑色正方形补丁，上有白色 "alo" 字母Logo。

**Step 3: 姿态与神情 (Pose & Expression)**
* 动作：身体向桌子前倾，重心下沉。左手手肘撑在桌面上，手掌托住脸颊/下巴。
* 视线：直视镜头，眼神清澈，带有一丝温柔或探究的笑意。

**Step 4: 摄影参数模拟 (Camera Parameters)**
* 焦段：50mm 或 85mm 定焦镜头。
* 光圈：f/1.8 或 f/2.0 (制造背景虚化)。
* 光线：模拟室内顶光，面部受光均匀，带有轻微暖调。

## 4. 图片整体描述 (Overall Description)
* **风格:** 真实感摄影 (Photorealistic)，生活方式 (Lifestyle)，高清 (8k resolution)，Instagram 风格。
* **色彩:** 黑色(衣服)与暖木色(桌子)为前景主调，背景杂糅暖黄光与红色点缀。
* **纹理:** 重点表现卫衣的抓绒质感、头发的光泽感、木桌的纹理。

## 5. 目标物体和语言输入框 (User Inputs)
* **[目标角色特征]:** （可爱短发亚洲女性）- *默认为：可爱短发亚洲女性*
* **[服装品牌细节]:** () - *默认为：alo 品牌 Logo*
* **[环境氛围]:** (户外咖啡馆) - *默认为：星巴克风格咖啡店*

---

**生成指令 (中文提示词参考):**
一张超写实的竖屏照片，视角略微俯视。画面主体是一位[可爱短发亚洲女性]，她正坐在咖啡店的浅色圆木桌前。她穿着黑色的半拉链高领抓绒卫衣，左侧袖子上有一个清晰的 "[alo 品牌 Logo]" 标签。她身体前倾，单手托腮，手肘撑在桌上，眼神温柔地看向镜头。背景是虚化的繁忙咖啡店，可以看到天花板的轨道灯、远处红色的菜单板和模糊的顾客。光线为温暖的室内顶光，肤色自然，发丝清晰，具有极高的摄影质感。
```

**出处：** [https://x.com/langzihan/status/2000808841089527981](https://x.com/langzihan/status/2000808841089527981)



## 4、职业西装风手账

Gemini Nano Banana Pro 出图示例：

![职业手账](assets/resource/aigc-prompt/aigc-prompt-7-4.jpeg)



**Prompt：**

```
9:16极简主义时尚插画。背景是干净的高级灰哑光纸张，仅有极细的工程制图线条。贴纸元素布局严谨，白边锐利。中央是用户穿着职业西装或极简风穿搭的贴纸。Labubu公仔贴纸戴着黑框眼镜，系着领带，一副精英模样。衣物解构贴纸包括折叠完美的西裤和名表。隐藏层贴纸是一件极简的高级黑色丝绸衬裙，展现低调奢华。所有的标注文字都是极细的黑色针管笔手写体。画面冷静、克制，无杂乱装饰，纯粹通过排版和材质对比展示高级感。
```

**出处：** https://x.com/songguoxiansen/status/1994309283488444523



## 5、大幅油画肖像

Gemini Nano Banana Pro 出图示例：

![油画肖像](assets/resource/aigc-prompt/aigc-prompt-7-5.jpeg)



**Prompt：**

```
Use the uploaded photo as the face reference for both the large painted portrait in the background and the full-body woman in the foreground. Create a stylish cinematic scene with a woman sitting confidently on the table in her personal luxury office room. She wears a loose pastel-toned dress or an oversized soft-colored suit, blending elegance and subtle boldness.The background features a huge artistic portrait of the same woman, painted with expressive pastel brushstrokes — pink, peach, beige — dynamic, sweeping strokes that create movement. Soft daylight, fashion-editorial mood, clean composition.Signature: Shreya Yadav
```

**中文版：** 

```
使用上传的照片作为面部参考，绘制背景中的大幅油画肖像和前景中的全身女性形象。创作一个时尚的电影场景：一位女性自信地坐在她豪华私人办公室的桌子上。她身着宽松的粉彩色连衣裙或宽松的浅色套装，优雅中透着一丝大胆。背景是一幅同一位女性的巨幅艺术肖像，以富有表现力的粉彩色笔触——粉色、桃色、米色——绘制而成，动感流畅的笔触营造出灵动的氛围。柔和的日光，时尚大片的风格，简洁的构图。签名：Shreya Yadav
```

**出处：** [https://x.com/__Shreyayadav/status/1993331098005520856](https://x.com/__Shreyayadav/status/1993331098005520856)





---

推荐一款端侧离线换脸工具，如果你想快速二创人物作品，可以使用 [FaceXSwap](https://www.facexswap.com) 在手机端换脸实现即可：

![FaceXSwap 换脸生成作品](assets/resource/aigc-product/facexswap_preview_short.gif)
_FaceXSwap 换脸生成作品_

![在 AppStore 搜索 'facexswap'](assets/resource/aigc-product/facexswap-2.png)
_在 AppStore 搜索 'facexswap'_

![FaceXSwap](assets/resource/aigc-product/facexswap.png)
_FaceXSwap_

- FaceXSwap 官网：<a href="https://www.facexswap.com" target="_blank">FaceXSwap: On-Device Offline AI Face Swap for Free</a>
- FaceXSwap iOS App 下载：<a href="https://apps.apple.com/app/id6752116909  " target="_blank">FaceXSwap iOS App Download</a>


