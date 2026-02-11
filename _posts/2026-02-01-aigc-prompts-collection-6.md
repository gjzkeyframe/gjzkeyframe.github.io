---
title: AI 魔法提示词集锦第 6 期：超写实人物生成
description: 这期我们来展示使用提示词让 AI 创作超写实人物。
author: Keyframe
date: 2026-02-01 08:08:08 +0800
categories: [AI 提示词]
tags: [AI 提示词, AI, AIGC, 视频, 照片, GIF, Gemini, Nano Banana, Wan, QWen, 人物写真]
pin: false
math: false
mermaid: false
image:
  path: assets/resource/aigc-prompt/aigc-prompt-6-1.jpeg
  alt: AIGC Image
---



## 1、粉调棚拍的傲娇定格照

Gemini Nano Banana Pro 出图示例：

![粉调棚拍](assets/resource/aigc-prompt/aigc-prompt-6-1.jpeg)



**Prompt：**

```
Photorealistic edit using the input person photo as strict identity reference: keep the same face, facial features, skin tone, hairstyle (color/bangs/length/volume), outfit and accessories unchanged (no face swap, no new person, no hair/outfit change). Only modify expression, subtle head angle/gaze, light body language, and background.
Expression must be light tsundere: her boyfriend just complimented her / wants to film her; she acts annoyed like "stop it / don't film me," but it's actually soft and sweet underneath. Facial actions: slightly averted gaze (side glance), a tiny eyebrow raise, lips gently pressed or a small pout, a barely-there restrained smirk (almost smiling but resisting), with a hint of shy pride; not real anger, not hostile, no big toothy grin.
Body language: torso slightly turned away as if avoiding the camera, shoulders subtly rotated out; eyes glance back with a side-eye; one hand makes a gentle "stop / don't film" block near the chest (light, half-hearted, not grabbing the phone).
Clean soft studio realism, face in sharp focus. Replace background with a solid pink seamless studio backdrop (subtle gradient ok).
Avoid identity changes, hair/outfit changes, exaggerated kawaii deformation, rage/aggressive vibe, text/watermarks/frames, extra fingers/deformed limbs.
```

**中文版：**

```
使用输入的人物照片作为严格的身份参考进行逼真编辑：保持相同的面部特征、肤色、发型（颜色/刘海/长度/蓬松度）、服装和配饰不变(no换脸，不添加新人物，不改变发型/服装）。仅修改表情、头部角度/眼神、轻微的肢体语言和背景。表情要带点轻傲娇：男友夸了她/想拍她；她表面上装作很恼火的样子，好像在说"别这样/别拍我"，但其实内心很温柔甜蜜。面部表情：眼神略微移开（侧目），眉毛轻轻挑起，嘴唇微微抿起或微微嘟嘴，嘴角带着一丝不易察觉的克制微笑（几乎要笑了却又忍住了），带着一丝羞涩的骄傲；不是真的生气，也不是敌意，更不会咧嘴大笑。肢体语言：躯干略微转向一边，仿佛在躲避镜头，肩膀微微向外旋转；眼睛斜睨着回头；一只手轻轻地在胸前做出"停止/不要拍摄"的手势（轻柔、漫不经心，没有抓住手机）。干净柔和的影棚写实风格，面部清晰对焦。将背景替换为纯粉色无缝影棚背景（可以有轻微的渐变）。避免身份改变、发型/服装改变、夸张的可爱变形、愤怒/攻击性氛围、文字/水印/边框、额外的手指/变形的四肢。
```

**出处：** [https://x.com/MANISH1027512/status/2008857436069781592](https://x.com/MANISH1027512/status/2008857436069781592)



---

**本文所介绍的 AI 人物生成，也可以使用 FaceXSwap 端侧离线换脸生成。**

![FaceXSwap 换脸生成作品](assets/resource/aigc-product/facexswap_preview_short.gif)
_FaceXSwap 换脸生成作品_

---



## 2、超写实工作室家庭肖像

Gemini Nano Banana Pro 出图示例：

![家庭肖像](assets/resource/aigc-prompt/aigc-prompt-6-2.jpeg)



**Prompt：**

```json
{
  "image_prompt": {
    "style": "Hyper-realistic studio family portrait",
    "resolution": "8K",
    "quality": "Ultra high definition, sharp focus on eyes",
    "lighting": {
      "type": "Soft studio lighting",
      "characteristics": "Diffused, warm, gentle shadows, intimate mood"
    },
    "background": {
      "type": "Solid color",
      "color": "Soft beige",
      "texture": "Smooth, seamless studio backdrop"
    },
    "subjects": [
      {
        "role": "Father",
        "position": "Top of the composition",
        "appearance": "Handsome young man",
        "expression": "Soft, calm, affectionate",
        "gaze": "Looking directly at the camera",
        "clothing": "Cream-colored chunky knit sweater"
      },
      {
        "role": "Mother",
        "position": "Below the father",
        "appearance": "Cute young woman",
        "pose": "Leaning back gently against the father",
        "expression": "Warm, relaxed, loving",
        "clothing": "Cream-colored chunky knit sweater"
      },
      {
        "role": "Baby",
        "position": "Front, centered",
        "appearance": "Cute infant",
        "expression": "Calm, innocent, natural",
        "pose": "Comfortably supported by parents",
        "clothing": "Cream-colored chunky knit sweater"
      }
    ],
    "composition": {
      "framing": "Close, intimate family grouping",
      "depth_of_field": "Shallow depth of field, creamy background blur",
      "focus_priority": "Eyes of all three subjects"
    },
    "atmosphere": "Warm, intimate, aesthetic, emotionally connected",
    "photography_style": "Professional studio portrait photography",
    "color_palette": "Neutral creams and warm beige tones"
  }
}
```

**中文版：**

```json
{
  "image_prompt": {
    "风格": "超写实工作室家庭肖像",
    "resolution": "8K",
    "质量": "超高清，眼睛清晰聚焦",
    "lighting": {
      "类型": "柔和的影棚灯光",
      "特点": "柔和温暖的阴影，营造出亲密的氛围"
    },
    "背景": {
      "类型": "纯色",
      "颜色": "柔和米色",
      "纹理": "光滑无缝的摄影棚背景"
    },
    "主题": [
      {
        "角色": "父亲",
        "位置": "构图顶部",
        "外貌": "英俊的年轻人",
        "表情": "柔和、平静、深情",
        "凝视": "直视镜头",
        "服装": "米色粗针织毛衣"
      },
      {
        "角色": "母亲",
        "位置": "在父亲之下",
        "外貌": "可爱的年轻女子",
        "姿势": "轻轻地靠在父亲身上",
        "表达方式": "温暖、放松、充满爱意",
        "服装": "米色粗针织毛衣"
      },
      {
        "角色": "婴儿",
        "位置": "正面，居中",
        "外貌": "可爱的婴儿",
        "表情": "平静、天真、自然",
        "姿势": "在父母的舒适支撑下",
        "服装": "米色粗针织毛衣"
      }
    ],
    "作品": {
      "框架": "亲密的家庭群体",
      "depth_of_field": "浅景深，柔和的背景虚化",
      "focus_priority": "三个拍摄对象的眼睛"
    },
    "氛围": "温暖、亲密、美观、充满情感联系",
    "摄影风格": "专业影棚人像摄影",
    "color_palette": "中性奶油色和暖米色调"
  }
}
```

**出处：** [https://x.com/Shreyayadav_2/status/2009128343535538546](https://x.com/Shreyayadav_2/status/2009128343535538546)



## 3、写实与皮克斯风的温柔对视

Gemini Nano Banana Pro 出图示例：

![皮克斯风](assets/resource/aigc-prompt/aigc-prompt-6-3.jpeg)



**Prompt：**

```json
{
  "Objective": "Create a hyper-realistic studio portrait blending photorealism with a whimsical 3D cartoon miniature character",
  "PersonaDetails": {
    "PrimarySubject": {
      "Type": "Young man",
      "Appearance": {
        "Hair": "Light brown hair, neatly styled",
        "FacialHair": "Neatly trimmed beard",
        "Skin": "Photorealistic, natural texture"
      },
      "Expression": "Soft, warm smile",
      "Gaze": "Looking at the miniature character on his fingertips",
      "Wardrobe": "Blue t-shirt"
    },
    "SecondarySubject": {
      "Type": "Tiny 3D cartoon version of the man",
      "Scale": "Miniature, resting on fingertips",
      "Style": "Pixar-style 3D character design",
      "Proportions": "Exaggerated cute proportions",
      "Features": {
        "Eyes": "Large, expressive",
        "Expression": "Friendly, playful pose"
      },
      "Consistency": "Matching hairstyle and outfit of the main subject"
    }
  },
  "Composition": {
    "Framing": "Close-up studio portrait",
    "Focus": "Ultra-sharp focus on both faces",
    "DepthOfField": "Shallow depth of field with soft background blur",
    "Balance": "Clean, centered composition emphasizing interaction"
  },
  "LightingAndMood": {
    "LightingStyle": "Cinematic soft studio lighting",
    "Highlights": "Gentle highlights on skin and miniature character",
    "Background": "Dark neutral backdrop",
    "Mood": "Modern, whimsical, warm"
  },
  "ArtDirection": {
    "StyleFusion": [
      "Photorealistic human portrait",
      "Pixar-style 3D character rendering"
    ],
    "DetailLevel": "High detail with clean textures",
    "RealismBalance": "Realistic human combined with stylized miniature"
  },
  "PhotographyStyle": {
    "Genre": "Professional studio portrait photography",
    "LensLook": "85mm lens perspective",
    "ImageQuality": "High resolution, ultra-sharp clarity"
  },
  "ColorAndTone": {
    "ColorPalette": "Natural skin tones with soft, modern colors",
    "Contrast": "Balanced contrast, smooth tonal transitions"
  },
  "NegativePrompt": [
    "cartoon human",
    "uncanny valley",
    "harsh lighting",
    "oversaturated colors",
    "blurry faces",
    "low detail",
    "messy composition"
  ],
  "ResponseFormat": {
    "Type": "Single image",
    "Orientation": "Portrait",
    "AspectRatio": "2:3"
  }
}
```

**中文版：**

```json
{
  "目标": "创作一幅超写实的影棚肖像，将照片写实主义与异想天开的3D卡通微缩人物相结合。",
  "PersonaDetails": {
    "主要科目": {
      "类型": "年轻男子",
      "外貌": {
        "头发": "浅棕色头发，梳理整齐",
        "面部毛发": "修剪整齐的胡须",
        "皮肤": "逼真自然的纹理"
      },
      "表情": "温柔温暖的微笑",
      "凝视": "看着他指尖上的微型人物",
      "衣橱": "蓝色T恤"
    },
    "第二学科": {
      "类型": "该男子的微型3D卡通版本",
      "比例尺": "微型，置于指尖之上",
      "风格": "皮克斯风格的3D角色设计",
      "比例": "夸张的可爱比例",
      "特征": {
        "眼睛": "大而有神",
        "表情": "友好、俏皮的姿势"
      },
      "一致性": "与主体发型和服装相匹配"
    }
  },
  "作品": {
    "构图": "特写影棚肖像",
    "对焦": "两张脸都实现了超清晰对焦",
    "景深": "浅景深，背景柔和虚化",
    "平衡": "简洁、居中的构图，强调互动"
  },
  "灯光和氛围": {
    "照明风格": "电影柔和的影棚照明",
    "亮点": "在皮肤和微缩人物上添加柔和的高光",
    "背景": "深色中性背景",
    "氛围": "现代、奇趣、温暖"
  },
  "艺术指导": {
    "StyleFusion": [
      "照片级写实人物肖像",
      "皮克斯风格的3D角色渲染"
    ],
    "细节级别": "高细节，纹理清晰",
    "写实平衡": "写实的人物与风格化的微缩模型相结合"
  },
  "摄影风格": {
    "类型": "专业影棚人像摄影",
    "镜头视角": "85mm镜头视角",
    "图像质量": "高分辨率，超清晰"
  },
  "ColorAndTone": {
    "调色板": "柔和、现代的自然肤色",
    "对比度": "均衡的对比度，平滑的色调过渡"
  },
  "否定提示": [
    "卡通人物",
    "恐怖谷效应",
    "刺眼的灯光",
    "色彩过饱和",
    "模糊的脸",
    "低细节",
    "杂乱的构图"
  ],
  "ResponseFormat": {
    "类型": "单张图片",
    "方向": "竖屏",
    "宽高比": "2:3"
  }
}
```

**出处：** [https://x.com/Taaruk_/status/2008818683234361811](https://x.com/Taaruk_/status/2008818683234361811)



## 4、超写实的户外冬季时尚肖像

Gemini Nano Banana Pro 出图示例：

![冬季肖像](assets/resource/aigc-prompt/aigc-prompt-6-4.jpeg)



**Prompt：**

```json
{
  "Ultra-realistic outdoor winter fashion portrait of a young Asian woman crouching in fresh snow. She is wearing a fitted short black dress paired with fur-lined snow boots. Her makeup is soft and natural. She has long, wavy black hair flowing naturally. One finger is gently touching her lips, giving a playful yet thoughtful expression. The background features snow-covered pine trees with a foggy winter atmosphere. Overcast natural lighting creates soft shadows. Shallow depth of field isolates the subject with a cinematic composition. High detail, sharp focus, Instagram aesthetic, 85mm lens look, soft contrast, and cold color tones dominate the scene.": "",
  "style": "ultra-realistic, cinematic, winter fashion",
  "environment": "outdoor, snowy forest, foggy winter atmosphere",
  "lighting": "overcast natural light, soft shadows",
  "camera": {
    "lens": "85mm",
    "depth_of_field": "shallow",
    "focus": "sharp"
  },
  "color_palette": "cold tones, soft contrast",
  "quality": "high detail, ultra realistic, instagram aesthetic",
  "aspect_ratio": "4:5",
  "instruction": "Use the face without any change."
}
```

**中文版：**

```json
{
  "描述": "这是一张超写实的户外冬季时尚肖像，描绘了一位年轻的亚洲女性蹲在皑皑白雪中。她身着黑色修身短裙，搭配毛绒雪地靴，妆容清新自然。一头乌黑的长卷发自然垂落。她用一根手指轻轻触碰嘴唇，神情既俏皮又沉思。背景是白雪皑皑的松树，笼罩着一层薄雾，营造出冬日的朦胧氛围。阴天的自然光线投射出柔和的阴影。浅景深使主体突出，构图极具电影感。画面细节丰富，焦点清晰，呈现出类似Instagram的审美风格，85mm镜头特有的质感，柔和的对比度和冷色调是这幅作品的主旋律。",
  "风格": "超写实、电影感十足的冬季时尚",
  "环境": "户外，白雪皑皑的森林，雾气弥漫的冬季氛围",
  "光线": "阴天自然光，柔和的阴影",
  "相机": {
    "镜头": "85mm",
    "景深": "浅",
    "焦点": "清晰"
  },
  "color_palette": "冷色调，柔和对比",
  "品质": "高细节、超逼真、Instagram 美学",
  "aspect_ratio": "4:5",
  "instruction": "使用面部，无需任何改变。"
}
```

**出处：** [https://x.com/Adam38363368936/status/2008804071235612934](https://x.com/Adam38363368936/status/2008804071235612934)



## 5、女子仿佛从刚冲洗出来的照片中浮现出来

Gemini Nano Banana Pro 出图示例：

![照片浮现](assets/resource/aigc-prompt/aigc-prompt-6-5.jpeg)



**Prompt：**

```json
{
  "subject": {
    "description": "A hyper-realistic optical-illusion photograph. The woman from the uploaded reference portrait appears to be emerging from a freshly developed instant photo (Polaroid-style) lying on a small cafe table. In the instant photo frame, her full outfit is visible; in reality, her upper body and head rise out of the glossy print, casting a real shadow onto the table.",
    "reference_image_rules": {
      "use_uploaded_reference_portrait": true,
      "preserve_identity": true,
      "preserve_hairline_and_facial_structure": true,
      "no_face_morphing": true
    },
    "age": "20s",
    "expression": {
      "eyes": {
        "look": "Playful and confident",
        "direction": "Looking at the viewer"
      },
      "mouth": {
        "position": "Pouting or blowing a kiss",
        "energy": "Chic and charming"
      },
      "overall": "Lifelike, engaging interaction"
    },
    "hair": {
      "style": "Long, loose waves",
      "effect": "Realistic shine, slight wind movement"
    },
    "pose": {
      "position": "Upper torso emerging out of the instant photo, one hand slightly forward as if stepping into reality",
      "overall": "Energetic, spontaneous, full of life"
    },
    "clothing": {
      "top": "High-neck knit turtleneck, premium textile detail",
      "bottom": "Mini skirt and leather boots (boots visible clearly inside the instant photo)"
    }
  },
  "mirror_rules": "All handwritten annotations must be perfectly legible and NOT mirrored. Keep printed text on the instant photo frame readable.",
  "props": {
    "instant_photo": {
      "look": "Glossy Polaroid print with subtle fingerprint smudges and micro-scratches",
      "frame_text": "Small printed caption line at the bottom of the frame (readable, not mirrored)"
    },
    "annotations_on_print": [
      {
        "text": "leather boots",
        "style": "white handwritten marker",
        "arrow_to": "boots inside the print"
      },
      {
        "text": "clean turtleneck",
        "style": "white handwritten marker",
        "arrow_to": "top inside the print"
      },
      {
        "text": "mini skirt",
        "style": "white handwritten marker",
        "arrow_to": "skirt inside the print"
      }
    ]
  },
  "photography": {
    "camera_style": "DSLR photorealism, macro lens for print texture",
    "shot_type": "Forced-perspective composite realism",
    "angle": "Top-down 3/4 angle, close and intimate POV",
    "aspect_ratio": "3:4",
    "lighting": "Soft overcast daylight, natural shadows",
    "depth_of_field": "Shallow DOF, the instant photo and her face sharp, background cafe bokeh"
  },
  "background": {
    "setting": "Paris sidewalk cafe in autumn",
    "elements": [
      "small espresso cup",
      "fallen leaves",
      "stone pavement",
      "soft distant pedestrians bokeh"
    ]
  },
  "the_vibe": {
    "mood": "Fashion-forward, viral illusion",
    "story": "OOTD breakdown escaping the photo",
    "authenticity": "Photoreal texture, not CGI"
  },
  "constraints": {
    "must_keep": [
      "Use uploaded reference portrait identity",
      "Photorealistic skin texture",
      "Instant photo looks physically real",
      "Handwritten annotations readable",
      "Strong pop-out illusion with real shadows"
    ],
    "avoid": [
      "3D render style",
      "cartoon",
      "plastic skin",
      "blurred or mirrored text",
      "fake glossy CGI print"
    ]
  },
  "negative_prompt": [
    "3d",
    "render",
    "cgi",
    "cartoon",
    "anime",
    "plastic skin",
    "illegible text",
    "mirrored text",
    "oversharpened halos",
    "uncanny face"
  ]
}
```

**中文版：**

```json
{
  "主题": {
    "描述": "一张超逼真的光学错觉照片。上传的参考肖像中的女子仿佛从一张刚冲洗出来的拍立得照片（宝丽来风格）中浮现出来，照片放在一张小咖啡桌上。在拍立得照片的相框中，她的全身衣着清晰可见；而实际上，她的上半身和头部从光亮的照片中浮现出来，在桌面上投下真实的阴影。",
    "reference_image_rules": {
      "use_uploaded_reference_portrait": true,
      "preserve_identity": true,
      "preserve_hairline_and_facial_structure": true,
      "no_face_morphing": true
    },
    "年龄": "20多岁",
    "表达": {
      "眼睛": {
        "look": "活泼自信",
        "direction": "看着观众"
      },
      "嘴": {
        "position": "撅嘴或飞吻",
        "energy": "时尚迷人"
      },
      "overall": "栩栩如生、引人入胜的互动"
    },
    "头发": {
      "style": "长长的、蓬松的波浪卷发",
      "effect": "逼真的光泽，轻微的风动"
    },
    "姿势": {
      "position": "上半身从即时照片中浮现出来，一只手微微向前伸出，仿佛正步入现实。",
      "overall": "精力充沛、率真、充满活力"
    },
    "衣服": {
      "上衣": "高领针织衫，优质面料细节",
      "下装": "迷你裙和皮靴（照片中可以清晰地看到靴子）"
    }
  },
  "mirror_rules": "所有手写注释必须清晰可辨，且不得镜像。请保持即时照片相框上的打印文字清晰可读。",
  "道具": {
    "instant_photo": {
      "look": "光亮的宝丽来照片，带有细微的指纹污渍和微划痕",
      "frame_text": "位于画框底部的小型印刷标题行（可读，非镜像）"
    },
    "annotations_on_print": [
      {
        "text": "皮靴",
        "style": "白色手写马克笔",
        "arrow_to": "打印内部的靴子"
      },
      {
        "text": "干净的高领毛衣",
        "style": "白色手写马克笔",
        "arrow_to": "打印内容的顶部"
      },
      {
        "text": "迷你裙",
        "style": "白色手写马克笔",
        "arrow_to": "裙边在印刷品内"
      }
    ]
  },
  "摄影": {
    "camera_style": "DSLR 真实感，用于打印纹理的微距镜头",
    "shot_type": "强制透视合成真实感",
    "angle": "俯视 3/4 角度，近距离亲密视角",
    "aspect_ratio": "3:4",
    "lighting": "柔和的阴天日光，自然的阴影",
    "depth_of_field": "浅景深，即时照片和她的脸部清晰，背景咖啡馆散景"
  },
  "背景": {
    "setting": "秋天的巴黎街边咖啡馆",
    "elements": [
      "小杯浓缩咖啡",
      "落叶",
      "石板路",
      "柔和的远景行人散景"
    ]
  },
  "the_vibe": {
    "mood": "时尚前卫，病毒式传播的错觉",
    "story": "OOTD 解析，摆脱照片的束缚",
    "authenticity": "照片级纹理，而非 CGI"
  },
  "约束": {
    "must_keep": [
      "使用上传的参考肖像身份",
      "逼真的皮肤纹理",
      "即时照片看起来非常逼真",
      "手写批注清晰可辨",
      "强烈的立体感，带有真实的阴影"
    ],
    "avoid": [
      "3D渲染风格",
      "卡通片",
      "塑料皮肤",
      "模糊或镜像文字",
      "仿光泽 CGI 印刷"
    ]
  },
  "negative_prompt": [
    "3d",
    "render",
    "cgi",
    "cartoon",
    "anime",
    "plastic skin",
    "无法辨认的文字",
    "镜像文本",
    "过度锐化的光晕",
    "怪异的脸"
  ]
}
```

**出处：** [https://x.com/hellokaton/status/2003381235331268757](https://x.com/hellokaton/status/2003381235331268757)





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


