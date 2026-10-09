---
name: character-sheet-forge
description: "Create and refine character reference sheets from user-provided images. Use when generating, editing, or validating 2x2 character sheets, front/back/side turnarounds, consistent character views, featureless-face panels, or image-generation prompts for character design."
compatibility: "For GitHub Copilot and other Agent Skills-compatible agents. Image generation, local image editing, and pixel-difference checks depend on tools available in the host environment."
metadata:
  version: "1.6.0"
  language: "zh-CN"
  category: "image-workflow"
---
# Character Sheet Forge

> 中文名称：角色参考图锻造工坊

> 版本：1.6.0。强化右上格“正面脸部特写”的景别定义、面部占画比例、正脸角度与构图验收；避免退化为肩部以上常规人像或三分之二侧脸。其余固定四宫格、左上无五官、参考图一致性和失败回退规则保持不变。

## 0. 触发范围与执行契约

### 何时启用

当用户要求生成、编辑、重绘、优化或验收角色参考图，并涉及以下任一任务时启用本 Skill：

- 角色四宫格、三视图、正面/背面/侧面 turnaround sheet。
- 以参考图锁定角色外观、服装、发型、配饰和跨视角一致性。
- 左上面部无五官、其他面板五官清晰的特殊设定图。
- 优化角色图像生成提示词、定位失败项或执行定向修复。

若用户只询问一般绘画知识，或请求与角色参考图无关的普通图片，不要强行套用本 Skill。

### 输入与输出

- **必要输入**：当前对话中可访问的角色参考图，或用户明确提供的角色描述。
- **可选输入**：目标风格、输出比例、指定模型/工具、补充视图、不可改变的服装细节。
- **默认输出**：一张四宫格角色参考图；若当前环境没有图像生成能力，则输出可直接使用的提示词和执行步骤，并明确说明尚未生成图片。
- **交付原则**：必须区分“提示词已编写”“图像已生成”“图像已验收”。三者不可互相替代。

### 不可违反的执行契约

1. 不得声称调用了当前环境不存在的图像生成、局部修复、蒙版、拼版或像素对比工具。
2. 不得把模型的自我描述当作验收证据；验收必须基于实际可查看的图像或可验证的处理结果。
3. 不具备局部编辑能力时，不得承诺只修改一个面板或蒙版外像素完全不变。
4. 不具备文件写入/导出能力时，不得声称已创建可下载文件。
5. 用户最新明确指令优先于本 Skill 的默认美术选择；但若用户要求与任务硬性条件冲突，应指出冲突并请求确认。


## 1. 目标与执行原则

将用户提供的人物参考图转化为一张 **9:16 竖版、2×2 四宫格角色参考图**。四格必须是同一个角色在不同视角下的对应呈现，角色外观、服装、发型、配饰、体型、配色、材质与整体视觉风格均以参考图为准。

本 Skill 的核心不是“生成一张看起来相似的拼图”，而是生成一张**顺序固定、视角准确、细节一致、面部处理规则明确且通过逐项验收**的角色参考图。

### 优先级

发生提示词冲突时，按以下优先级处理：

1. 四格顺序、视角与构图硬约束。
2. 左上格无五官规则、右上格五官清晰规则。
3. 同一角色及服装、发型、配饰的一致性。
4. 参考图的风格、光线、背景与画质。
5. 画质与视觉美化。
6. 非必要装饰与风格化增强。

不得为了追求美观而改变视角、交换格子、裁切全身、错误移除其他面板的五官或添加参考图中不存在的元素。

## 2. 固定布局规范（硬约束）

画布比例为 **9:16 竖版**，采用无边框的 2×2 四宫格。四格阅读顺序固定为：左上 → 右上 → 左下 → 右下。

| 位置 | 视角与构图 | 必须满足 |
|---|---|---|
| 左上 | 正面全身全景 | 人物正对镜头，标准放松站姿，双手自然下垂；从头到脚完整入镜，双脚和鞋子完整可见；**面部无可见五官（无眼睛、眉毛、鼻子、嘴巴）** |
| 右上 | 正面脸部特写（Close-up） | **以脸部为主体的紧凑特写**，头部正对镜头、双眼平视；完整保留头顶发型轮廓至下巴，脸部占据画面主要区域；仅允许少量颈部/肩部边缘，不得退化为肩部以上常规人像、半身像或 3/4 侧脸；五官清晰锐利，**禁止模糊** |
| 左下 | 背面全身全景 | 人物背对镜头，自然放松站立；从头到脚完整入镜，展示发型背面、服装后背及整体轮廓 |
| 右下 | 左侧面全身全景 | 人物身体严格旋转 90°，面朝画面左侧；纯侧面，不是 3/4 侧面；从头到脚完整入镜 |

### 全局构图规则

- 四格尺寸尽可能相等，位置和顺序不可更改。
- 四格均为平视机位（eye-level），不仰拍、不俯拍、不鸟瞰。
- 三格全身图必须完整展示头部、身体、腿部、鞋子及鞋底，不得裁切头顶、脚尖或鞋底。
- 右上格必须是**脸部景别的正面特写**，不是泛指“肩部以上肖像”：头顶至下巴完整入镜，头部/脸部成为画面绝对主体；肩膀只可在底边少量出现，胸口和上半身不得占据明显画面面积。
- 右上格必须正对镜头：头部无明显左右转角或倾斜，鼻梁大致位于面部中轴，双眼均完整可见且视觉大小合理；禁止 3/4 角度、侧脸、俯拍、仰拍或夸张透视。
- 右上格可保留参考图中的眼镜、刘海、耳饰等设计，但不得让配饰或头发遮挡关键五官；画面应优先呈现完整、清晰的脸部信息。
- 所有格子中的角色必须是同一个人；不得因视角变化而改变发色、发型、服装、配饰、体型或肤色。
- 背景统一为纯白或参考图要求的统一背景；默认使用干净的纯白影棚背景。
- 格子之间不添加边框、分隔线、标签、文字、水印或额外装饰。
- 不允许加入第二个人物、镜子中的额外人物或与参考图无关的道具。

### 四格拼版与几何一致性

- 先定义统一画布，再以准确的 2×2 网格分配四个等宽、等高面板；不得让右上特写跨越中线，也不得让上下两排高度不等。
- 若图像模型无法稳定输出精确分格，优先分别生成四个面板，再用图像处理按固定坐标拼版；不要依赖模型自由绘制分隔布局。
- 拼版时统一画布尺寸、面板尺寸与背景色；除非用户明确要求，不加可见边框或分隔线。
- 生成后检查中线位置、四格边界、面板顺序与画面比例。仅凭“看起来像四宫格”不算通过。
- 不允许把单张四格图再次整体拉伸来适配比例；应使用等比例缩放与留白，避免角色变胖、变瘦或头身比变化。
- 三个全身面板的角色高度应尽可能接近；特写面板只允许按特写构图放大面部，不得改变角色本身比例。

## 3. 左上格无五官规则（最高优先级）

“无五官”规则**只应用于左上格**，不得扩散到其他格子。它不是模糊、马赛克或遮挡，而是让面部呈现干净、自然、没有可见五官的平滑状态。

### 左上格：面部无五官

- 左上格人物仍保持正面全身、头部姿态与发型不变；面部皮肤区域不显示任何可辨认的五官。
- 必须移除/不绘制：双眼、眼球、瞳孔、眼睑线、眉毛、鼻梁、鼻尖、鼻翼、鼻孔、嘴唇、嘴线及可识别的嘴部细节。
- 保留自然的脸部外轮廓、肤色、光照渐变与基本体积感；脸部应像平滑、自然的无五官面部，而不是空洞、恐怖、破损或面具状。
- 不得以模糊、像素化、马赛克、强阴影、头发遮挡、手遮脸或转头代替“无五官”。
- 耳朵、发际线、刘海、脸颊轮廓、下巴、颈部、耳饰、项圈等非五官元素应保留并与原角色一致。
- 不要抹掉整张脸，不要改变头部形状、肤色、发型、头部大小或光照方向。编辑边界应限制在面部皮肤区域。
- 若模型无法可靠生成自然无五官面部，先生成正常清晰底图，再对左上格的面部皮肤区域做局部修复/重绘；只编辑脸部，不重绘头发、身体、服装或其他面板。

### 右上格：正面脸部特写与清晰度（硬约束）

- **景别定义：脸部特写（face close-up）**。画面重点是脸，而不是肩部、胸口或上半身。头顶发型轮廓、额头、双眼、鼻子、嘴巴、下巴应完整呈现；除非参考图造型本身需要，不得切掉头顶、刘海关键轮廓或下巴。
- **画面占比：** 头部从发顶到下巴约占面板高度的 75%–90%；脸部位于面板中央，左右留白适度。颈部和肩部只允许在画面下缘少量出现，不得使用远景或普通肩部以上人像构图。
- **正面角度：** 面部正对镜头，头部水平、眼睛平视；鼻梁接近垂直中轴，双眼同时可见且不因转头产生明显大小差。不得使用 3/4 视角、侧脸、歪头姿态、俯拍或仰拍。
- **焦点与表情：** 眼睛、眉毛、鼻子、嘴巴和面部轮廓全部清晰对焦；自然中性表情，不微笑、不皱眉、不张嘴做夸张表情。
- **造型保留：** 忠实保留参考图中的发型、眼镜、妆容和面部可见特征；允许刘海自然覆盖少量额头，但不得遮住关键五官。
- 不允许面部模糊、马赛克、遮挡、景深虚化或运动模糊。如果右上格虽清晰但景别过远、不是正脸，或头部被明显转成 3/4 角度，仍判定该硬性项目 `FAIL`。

### 面部局部修复规范（用于避免误改其他区域）

**不要让整图生成步骤同时承担精确面部编辑。** 初始生成时可让四格面部均保持自然清晰；随后只对左上格面部皮肤区域进行局部修复，移除五官。右上格保持完整清晰。

1. 先裁出左上格，再定位面部皮肤边界；不得直接用全画布固定百分比猜测编辑区域。
2. 建立一个贴合面部皮肤范围的蒙版 `MASK_FACE_FEATURES`，覆盖五官所在区域，但不得越过脸部轮廓、发际线、刘海、耳朵或下巴边界。
3. 在蒙版内使用局部修复/内容感知重绘，将眼睛、眉毛、鼻子和嘴巴自然移除，并重建连续的皮肤色调与光影。不要使用高斯模糊替代无五官处理。蒙版外像素必须保持不变。
4. 蒙版边缘轻微羽化以融合皮肤，但不能越过脸部轮廓或污染头发、耳朵和背景。若无法准确分割面部皮肤，停止自动处理并标记 `NOT_VERIFIED`。
5. 完成后将左上格放回原四宫格，不重绘或重采样其他三个面板。若拼接/导出改变了整张图，应检查右上格与其他面板是否仍保持原样。
6. 处理前后对比：确认左上格不再存在眼睛、眉毛、鼻子或嘴巴，同时脸部轮廓、肤色、光影、发际线和刘海自然；右上格完整五官完全清晰。
7. 如果没有可用的人脸分割或局部修复能力，不要凭猜测涂抹整张脸；保留原图并将无五官项标记为 `NOT_VERIFIED`，说明需要支持局部修复的编辑步骤。

**无五官效果原则：** 目标是完全看不到可识别五官，同时保留自然的脸部体积、肤色与光照；不得留下淡淡的眼眶、鼻影、嘴线或眉毛残影。

## 4. 参考图特征锚定

生成前先分析参考图，并把观察结果明确写入提示词。至少记录以下维度：

1. **整体风格**：写实人像、Cosplay 摄影、插画、3D 渲染等。
2. **画质质感**：摄影或渲染质感、清晰度、皮肤纹理、材质细节。
3. **肤色与妆容**：肤色、妆容浓淡、可见的面部特征。
4. **发型发色**：颜色、长度、刘海、卷直、马尾/双马尾、发饰与发束走向。
5. **服装款式**：上下装结构、颜色、材质、图案、层次、剪裁。
6. **配饰细节**：耳饰、项圈、手套、袜子、鞋子、腰带、扣具、蝴蝶结等。
7. **体型比例**：整体身形与四肢比例，避免夸张变形。
8. **背景**：背景颜色、场景类型、是否纯色。
9. **光影**：自然柔光、影棚光、阴影强弱、色温与对比度。

只描述参考图中确实可见的特征；看不清的细节标记为未知，不要自行编造。保留参考图中具有辨识度的设计元素，不要为了“统一风格”而擅自替换服装或配饰。

### 参考信息置信度与缺失细节处理

将参考信息分为三类，并在生成前内部整理：

- **A级：直接可见**——参考图中清楚可辨的发型、上衣、饰品、颜色、可见材质等，必须严格保留。
- **B级：可合理推断**——从现有结构能较有把握推断的背面接缝、衣物延续关系等，只能采用最简洁、最不显眼的方案。
- **C级：不可知**——参考图没有展示的鞋款、背部图案、隐藏配件、裤袜上端结构等，不得编造醒目的独特设计。若任务必须补全全身，可使用低装饰、低辨识度的中性延续方案，并在验收报告中标注“部分细节为推断”，不能声称完全忠实于不可见部分。

若用户提供的是半身或膝上参考图，而任务要求全身三视图：
1. 严格复制可见区域。
2. 对未显示的下半身、鞋子和背面结构，只做完成构图所必需的最小推断。
3. 不得擅自添加高跟鞋、靴子、复杂花纹、额外饰品或新的服装层，除非用户明确授权。
4. 如果不可见区域的设计对角色辨识度很重要，应先询问用户补充参考图；若用户希望直接继续，则使用保守方案并标记不确定项。

提示词中应包含明确的一致性语义，例如：

`exact same character, exact same outfit, exact same hairstyle, same accessories and proportions as the reference image`

若原图为非写实风格，不得强制转换成写实摄影；风格必须跟随参考图。若用户明确要求写实风格，则以用户要求为准。

## 5. 视觉美化与画质质量规范

本节用于提升成片质感，但不得改变参考角色设计，也不得覆盖第 2、3 节的硬性规则。美化目标是“更干净、更精致、更可读、更统一”，不是擅自增加装饰或把参考图改造成另一种风格。

### 5.1 美化原则与优先级

1. **忠于参考图**：服装剪裁、发型结构、发饰、鞋靴、配件、配色和角色气质以参考图为准。
2. **结构先于细节**：先保证四格布局、角度、人体结构、全身完整度，再优化材质与微细节。
3. **真实细节优先于过度锐化**：保留自然皮肤、布料、皮革、金属和头发的材质差异；禁止塑料皮肤、过强磨皮、过度锐化白边、假 HDR 和不自然的高频纹理。
4. **一致性优先于单格惊艳**：四格的色温、曝光、对比度、背景和材质表现应一致。不得只让某一格出现强烈电影光或不同滤镜。
5. **适度美化，不重设计**：允许整理杂乱边缘、改善照明与细节可读性；禁止凭空添加纹身、首饰、花纹、武器、翅膀、发光特效、额外腰带或服装层。
6. **无文字输出**：除非用户明确要求，禁止标题、标签、角度说明、尺寸标注、签名、水印和 UI 装饰。

### 5.2 摄影、光线与色彩

默认写实摄影路线（仅当参考图或用户要求写实时启用）：

- 影棚级柔和主光，辅以自然的环境补光；保留轻微而合理的接触阴影。
- 面部、头发、服装和鞋靴均有清晰但不过硬的光影层次。
- 高光不过曝，白色服装仍能看见褶皱与蕾丝细节；黑色皮革仍保留缝线、折痕和反光，不糊成纯黑块。
- 肤色自然，避免偏绿、偏灰、过度橙红或过度粉白；四格肤色与白平衡保持一致。
- 背景保持干净、均匀，人物边缘与背景有足够分离度；默认纯白或非常浅的中性灰背景。
- 阴影方向、主光方向和反射逻辑一致，不允许同一格左侧受光、另一格无理由右侧受光。
- 避免强烈景深虚化、镜头光斑、烟雾、粒子、彩色轮廓光等干扰设定图可读性的效果。

若参考图为插画、动漫或 3D 风格，则沿用对应媒介，不强制套用写实摄影术语。

### 5.3 材质细节与微观质量

- **头发**：保留发束分组、发丝方向、发根与发梢层次；发饰、蝴蝶结、马尾和长发轮廓跨格一致。避免头发融成一整块、发丝无规律飞散或穿过身体。
- **皮肤**：右上特写允许自然细腻的皮肤纹理与轻微毛孔；不要磨皮成瓷娃娃，不要过度生成雀斑、痣或妆容。
- **布料**：表现织物厚薄、褶皱走向、缝线、蕾丝和裙摆层次；褶皱应服从姿势与重力，不可随机堆叠。
- **皮革/漆皮**：高光有方向性，保留暗部层次、缝线、扣带、孔眼和边缘厚度；避免材质像液态塑料。
- **金属配件**：扣具、铆钉、拉链、环扣和鞋饰应有合理的小面积高光，形状清楚且数量不随视角随机变化。
- **透明/薄纱材质**：保留透明度、层次和边缘，不要与皮肤或背景融为一体。
- **鞋靴**：鞋底、鞋跟、鞋带、扣具和鞋头结构清楚；全身格不得因美化裁掉鞋子或鞋底。

### 5.4 构图、留白与人物比例

- 四格均衡、人物比例统一，角色在三个全身格中的视觉高度尽量接近。
- 全身格四周保留适量安全留白，头顶、发饰、飘带、长发末端、手指、鞋尖和鞋底不得贴边或被裁切。
- 参考图中有夸张飘动的长发或裙摆时，应在构图中为其留出空间；不能为填满画面而切断发束。
- 正面、背面、侧面采用相近的镜头距离与透视尺度，避免侧面人物显著变大或变小。
- 右上特写需清晰呈现面部和肩部以上轮廓，头顶发饰尽可能保留；不得为了放大五官而切掉下巴或关键发饰。
- 手指、关节、鞋带、扣具等细节应自然，禁止多指、黏连手指、扭曲关节、穿模和不合理肢体长度。
- 角色采用标准、便于比对的中性站姿；不把参考图里的动态跳跃姿势直接套用到三视图站姿中，除非用户明确要求保留动态姿势。

### 5.5 分辨率、清晰度与输出

- 优先选择工具支持的最高实用分辨率与高质量输出模式；不得编造具体像素、采样步数或模型参数。
- 输出应适合放大检查：发饰边缘、服装接缝、鞋靴扣具、面部细节和四格边界清楚，无明显压缩块、锯齿、涂抹或重复纹理。
- 清晰度应均衡；右上特写可以呈现更丰富的面部细节，但不得因过度锐化而与其他三格呈现不同的材质风格。
- 不使用强降噪导致细节融化；不使用过度锐化造成轮廓白边；不以高对比度掩盖人体结构问题。
- 若生成结果尺寸较低，可在不改变内容的前提下使用高质量放大；放大后仍须检查是否出现假细节、重复纹理或边缘光晕。
- 最终导出优先使用 PNG；若因平台要求使用 JPEG，应避免过高压缩。实际文件格式以工具能力为准。
- 不得为了提升“高清感”而增加参考图不存在的纹理、图案、妆容或配件。

### 5.6 统一负面约束

根据所用图像工具支持情况，将以下内容加入负面提示词或主提示词末尾。若工具没有独立负面提示词栏，则将其作为普通文本约束写入主提示词。

`low resolution, blurry details, compression artifacts, oversharpening, plastic skin, waxy skin, excessive smoothing, fake HDR, harsh blown highlights, crushed blacks, inconsistent lighting, inconsistent color grading, inconsistent character design, different outfit between panels, altered hairstyle, missing accessories, invented accessories, random ornaments, extra straps, extra jewelry, text, watermark, logo, borders, panel labels, bad anatomy, deformed hands, extra fingers, missing fingers, fused limbs, duplicated limbs, floating objects, broken perspective, cropped head, cropped hair accessories, cropped hands, cropped feet, missing shoes, inconsistent panel scale`

注意：负面约束不可误伤必要内容。例如参考图本来有多条腰带或大量饰品时，`extra straps` 与 `extra jewelry` 的意思是“不要额外编造”，不是要求删除原有设计。

### 5.7 提示词信息密度

- 先写硬性结构，再写角色特征，再写画质和美化，最后写禁止项。
- 使用具体可观察的描述，避免堆叠同义词，例如同时重复 “masterpiece, best quality, ultra-detailed, 8K, perfect” 并不能保证质量。
- 不要使用相互冲突的词组，例如“纯白无阴影背景”与“强烈戏剧性投影”；应明确选择干净设定图照明。
- 避免只写“精美、高清、高级感”。要说明具体目标：清晰的材质分层、自然的肤色、细致的缝线、均匀的光线、完整的轮廓和无伪影。
- 当提示词长度受限时，保留顺序：布局/角度 > 无五官规则 > 角色锚点 > 全身完整度 > 画质要求 > 装饰性形容词。

## 6. 工作流程

按阶段执行。每一阶段都要检查进入下一阶段的条件；不满足条件时先修正或说明限制，不要跳过失败继续美化。

### Step 1 — 检查输入与工具能力

1. 确认当前对话中存在可访问的参考图，或用户已提供足够具体的角色描述。
2. 多张参考图时，确定主参考图；其他图仅用于补充其实际展示的细节。
3. 判断当前宿主是否具备所需能力：图像生成、参考图条件输入、局部蒙版编辑、图像拼版、图像查看/比较、文件导出。
4. 只使用实际可用的能力。能力缺失时，选择可行的降级路径并在最终交付中说明限制。
5. 如果参考图无法访问或关键要求无法理解，先提出一个聚焦问题，不要凭空编造角色设计。

**阶段门槛：** 已确认输入来源、主要目标和可用工具；未知的参考细节已标记为未知。

### Step 2 — 提取角色锚点与不确定项

依据第 4 节记录角色锚点，至少包括风格、发型、发色、服装、配饰、体型、背景、光线和可见材质。

将信息标记为：
- `OBSERVED`：参考图直接可见，必须保留。
- `INFERRED`：为补齐视图所需的谨慎推断，需尽量低装饰、低辨识度。
- `UNKNOWN`：参考图无法支持，不得伪装成已知事实。

**阶段门槛：** 关键可见特征已整理；不会把不可见细节写成确定事实。

### Step 3 — 组装提示词

提示词按以下顺序组织：

1. 输出比例、2×2 布局和面板顺序。
2. 统一背景、视角、摄影/渲染媒介和画质要求。
3. 角色锚点与跨格一致性。
4. 左上、右上、左下、右下各格的独立要求。
5. 禁止项、裁切限制和角色设计保护。
6. 明确优先级：布局/视角/全身完整度/面部规则优先于美化。

不得只用“正面、背面、侧面、特写”等缩写；必须逐格明确位置、角度、取景和五官要求。

**阶段门槛：** 提示词中没有相互冲突的角度、风格、光线或面部规则，也没有未替换的占位符。

### Step 4 — 选择生成策略

按宿主能力选择以下策略，不要假设每种工具都支持所有步骤：

**路径 A：可生成且支持参考图**
1. 使用参考图与角色锚点生成候选图。
2. 优先保证布局、视角、角色一致性与全身完整度。
3. 若图像工具无法可靠同时处理左上无五官，先生成结构正确且面部清晰的基础图，再进入局部修复路径。

**路径 B：支持局部编辑/蒙版**
1. 保留原始图作为 `SOURCE`，每次编辑另存为新版本。
2. 明确目标面板和目标区域；仅对左上面部皮肤区域做局部编辑。
3. 蒙版须依据实际面部位置定位，不得按整张画布的固定百分比猜测。
4. 若无法可靠识别面部皮肤边界，停止自动修复并标记 `NOT_VERIFIED`。
5. 修复后对比原图，检查目标区域以外是否变化。只有实际执行了像素差异或等效验证，才可声称像素不变。

**路径 C：无法局部编辑，但可生成整图**
1. 可尝试直接生成完整四宫格，但不得承诺局部内容完全锁定。
2. 若无五官规则失败，说明限制并尝试另一种可用策略；不得用模糊、马赛克或遮挡冒充无五官。
3. 若重复生成仍无法满足硬性要求，交付失败诊断与改进提示词，不得将其标记为通过。

**路径 D：没有图像生成能力**
输出完整、可复制的提示词与执行步骤，并明确说明本轮只完成了提示词/工作流，没有实际生成图像。

**分格生成与拼版：** 若模型无法稳定生成准确四格，可分别生成四个面板再拼版。先分别验收源图，再验收拼版；使用等比例缩放与留白，不得拉伸图像。拼版后重新检查面板顺序、尺寸、边界和色彩一致性。

**阶段门槛：** 已获得候选图，或已明确当前环境只能交付提示词；没有虚报工具能力或处理结果。

### Step 5 — 验收候选图

1. 对每个候选图逐项执行第 7 节清单。
2. 将每项标记为 `PASS`、`FAIL` 或 `NOT_VERIFIED`，并写明可观察的原因。
3. 任一硬性项目失败，整个候选图判为 `FAIL`。
4. 无法查看图像、看不清细节或无法验证局部保护时，使用 `NOT_VERIFIED`，不可默认通过。
5. 先修正结构与一致性，再处理材质、锐度和美化。

**阶段门槛：** 每个候选图都有独立验收结论；没有用“整体好看”代替硬性验收。

### Step 6 — 定向修复与回归检查

1. 根据第 9 节选择与失败项对应的最小修复动作。
2. 每次修复只针对明确的失败项；不无必要地重写已通过的设计。
3. 修改后重新检查受影响的项目，并回归检查其他已通过项目。
4. 若其他面板被意外修改，回退到 `SOURCE` 或最近一个已验收版本。
5. 达到合理重试次数仍无法修复时，停止无效重复，说明失败原因与工具限制，并询问用户是否接受替代方案。

### Step 7 — 导出与交付

1. 只交付真实存在且可访问的图像/文件。
2. 明确区分 `GENERATED`（已生成）、`PASS`（已通过验收）、`FAIL`（验收失败）、`NOT_VERIFIED`（无法验证）。
3. 报告硬性项目结果、失败项、参考图不可见区域的推断，以及局部编辑保护是否实际验证。
4. 若没有任何候选图通过，明确说明没有合格版本；可以交付失败诊断或提示词，但不能称为合格图。
5. 仅在当前环境实际创建并确认文件可访问后，提供下载链接。

## 7. 质量检查清单（硬性验收）

对每个候选图逐项检查，并记录 `PASS`、`FAIL` 或 `NOT_VERIFIED`。

### A. 布局与顺序
- [ ] `PASS/FAIL/NOT_VERIFIED` — 9:16 竖版，清楚的 2×2 四宫格。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 四个面板等宽等高，中心分界位置正确，没有面板跨界或上下排高度不等。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 拼版没有拉伸变形；画布适配使用等比例缩放或留白。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 顺序严格为左上正面全身、右上正面面部特写、左下背面全身、右下左侧面全身。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 无文字、水印、标签、边框或分隔线。

### B. 视角与构图
- [ ] `PASS/FAIL/NOT_VERIFIED` — 四格均为平视机位。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 左上人物正对镜头、标准放松站姿、双手自然下垂。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 右上是脸部景别的紧凑正面特写，而非普通肩部以上肖像；头顶至下巴完整入镜，头部约占面板高度 75%–90%。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 右上头部正对镜头、眼睛平视，鼻梁接近中轴、双眼均可见；没有 3/4 角度、侧脸、明显歪头或俯仰机位。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 右上颈部/肩部只在底边少量出现，胸口和上半身没有抢占画面；脸部成为画面主体。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 左下为背对镜头的自然站姿。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 右下为严格 90° 左侧面，人物面朝画面左侧，不是 3/4 侧面。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 左上、左下、右下均从头到脚完整可见，双脚和鞋底未裁切。

### C. 无五官规则
- [ ] `PASS/FAIL/NOT_VERIFIED` — 左上格不显示眼睛、眼球、眉毛、鼻子、鼻孔、嘴唇或嘴线。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 左上格脸部平滑自然，无模糊、马赛克、五官残影、恐怖空洞或面具感。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 左上格脸部轮廓、肤色、光影、发际线、刘海、耳朵和饰品自然保留。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 右上格五官完全清晰，没有模糊、马赛克、遮挡或失焦。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 右上景别确为紧凑脸部特写：头顶至下巴完整，头部占面板高度约 75%–90%，肩部/胸口没有占据明显面积。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 右上为严格正面脸部角度，双眼可见且比例平衡，无 3/4 转头、侧脸、明显歪头或高低机位。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 无五官处理仅发生在左上格，没有扩散至其他格子。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 若采用局部后处理，右上、左下、右下三个面板未被重新生成或意外修改。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 若宣称蒙版外像素未改变，已使用实际像素差异或等效对比方法验证；否则必须标记 `NOT_VERIFIED`。

### D. 角色一致性
- [ ] `PASS/FAIL/NOT_VERIFIED` — 四格为同一角色。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 发型、发色、服装、配饰、体型、肤色与参考图一致。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 背面与侧面合理呈现同一套服装结构，没有无依据地新增或丢失关键设计。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 参考图未展示的区域未被擅自添加醒目的新设计；必要推断保持简洁，并已标注不确定性。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 人体比例自然，无明显肢体畸形或多余肢体。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 三个全身视图的角色高度与镜头尺度接近，差异来自视角而非缩放失控。

### E. 画面质量与美化
- [ ] `PASS/FAIL/NOT_VERIFIED` — 风格与参考图/用户要求一致，没有擅自风格转换。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 光线方向、曝光、白平衡与背景在四格间一致。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 黑色、白色及透明材质均保留可读的层次与纹理。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 头发、蕾丝、缝线、皮革、金属扣具等关键材质清楚且符合参考图。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 无明显低分辨率、压缩块、锯齿、涂抹、重复纹理、锐化白边或假细节。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 人物四周有合理留白，发饰、发梢、手指和鞋底均未贴边或裁切。
- [ ] `PASS/FAIL/NOT_VERIFIED` — 无多余人物或无关元素；没有新增参考图中不存在的装饰。

### 判定规则

- 任一布局、角度、全身完整度、左上无五官规则、右上五官清晰规则或角色一致性硬约束为 `FAIL`，整个候选版本即判定为 **FAIL**。
- 如果无法确认某项是否满足，必须标记 `NOT_VERIFIED`，不得擅自标记 `PASS`。
- 只有所有硬性项目均为 `PASS`，且没有关键项 `NOT_VERIFIED`，才可判定候选版本通过。
- 不要仅凭生成模型的文字说明判定图片合格；应实际检查生成图像。
- 若无法访问图像或无法看清细节，应明确说明该项无法验证。

## 8. 提示词模板

先将 `{角色锚点}` 替换为从参考图中提取的具体信息。不得保留未填写的占位符直接生成。

### 英文主提示词

```text
Create a high-quality 9:16 vertical character reference sheet arranged as a mathematically even 2x2 grid of four equal-width and equal-height panels, with no borders or divider lines. Keep all panel boundaries aligned; no panel may cross the center line. Use the exact same character from the reference image in every panel. Preserve the original design faithfully wherever visible in the reference: outfit construction, hairstyle silhouette, hair color, accessories, body proportions, materials, color palette, makeup style, and visual medium. Do not invent distinctive details in areas hidden or cropped out of the reference; use the simplest neutral continuation only when a full-body view requires it. {CHARACTER ANCHORS}. Clean neutral studio background, consistent soft key light and gentle fill, consistent white balance and exposure, natural contact shadows, crisp material separation, realistic fabric folds and stitching, detailed leather and metal hardware, clean hair strand grouping, natural skin texture in the clear portrait, anatomically correct hands and feet, balanced negative space, eye-level camera in all panels, matched scale across full-body views, no perspective distortion. High-resolution clean output with fine but natural detail; no plastic skin, no fake HDR, no oversharpening, no compression artifacts.

TOP-LEFT PANEL: Full-body front view, standing straight in a neutral relaxed pose, facing directly toward the camera, arms resting naturally at the sides. Show the entire body from the top of the head to the bottoms of both shoes; do not crop any part. The face must have NO visible facial features: no eyes, eyeballs, eyelids, eyebrows, nose, nostrils, lips, or mouth line. Preserve a smooth, natural skin surface with believable facial volume, skin tone, and lighting. This is not blur, mosaic, a mask, or hair covering the face. Keep the face outline, hairline, bangs, ears, neck, and accessories intact. If needed, remove the features with localized inpainting only inside the facial skin region after generation.

TOP-RIGHT PANEL: A TIGHT, FRONT-FACING FACE CLOSE-UP, NOT a generic shoulders-up portrait. The head faces the camera squarely at eye level; no three-quarter turn, no side view, no tilted head, no high or low camera angle. Frame the complete head from the top of the hairstyle to the chin. The head occupies approximately 75–90% of the panel height; the face is the dominant subject, centered with balanced side margins. Only a small amount of neck or shoulder may appear along the bottom edge; do not show a prominent chest or upper torso. Both eyes must be visible and balanced in size, the nose close to the facial centerline, and the eyebrows, eyes, nose, lips, and chin all fully inside the frame. Preserve reference-specific hair, glasses, makeup, and facial features without letting them obscure key features. Neutral expression. Every facial feature must be crisp and clearly focused. Absolutely no blur, mosaic, obstruction, defocus, three-quarter angle, side profile, or distant portrait framing.

BOTTOM-LEFT PANEL: Full-body back view, relaxed natural standing pose, facing away from the camera. Show the complete head-to-toe silhouette and both shoes, including the back hairstyle and back construction of the exact same outfit.

BOTTOM-RIGHT PANEL: Full-body strict 90-degree LEFT SIDE PROFILE, the character's face and body oriented toward the LEFT edge of the image. This must be a true side view, NOT a three-quarter view. Relaxed natural standing pose; show the complete body from head to the bottoms of both shoes.

All four panels show the exact same character and consistent outfit. No text, no watermark, no labels, no extra people, no panel borders, no cropped head or feet, no missing shoes, no altered costume, no invented ornaments, no swapped panel order, no bad anatomy, no extra fingers, no fused limbs, no inconsistent lighting, no mismatched color grading, no plastic skin, no oversharpening, no fake HDR, no compression artifacts. The featureless-face rule applies ONLY to the top-left panel; the top-right face remains fully sharp and shows complete facial features. Prioritize layout, viewing angles, full-body completeness, and featureless-face rules over decorative effects.
```

### 中文主提示词

```text
生成一张高质量 9:16 竖版、严格等分的 2×2 四宫格人物角色参考图，四格等宽等高、中心分界对齐、无边框、无分隔线，任何面板都不得跨越中线。四格必须是参考图中的同一个角色，在参考图可见范围内忠实保留原设计：服装结构、发型轮廓、发色、配饰、体型比例、材质、配色、妆容风格与整体视觉媒介。不得为参考图未展示的区域擅自编造醒目的新设计；必须补全全身时，仅采用最简洁、中性的延续方案。角色锚点：{角色锚点}。背景干净统一，影棚柔和主光与自然补光，四格曝光、白平衡和光线方向一致，接触阴影自然，材质层次清楚；布料褶皱与缝线细致，皮革和金属配件质感明确，头发发束分组自然，清晰特写保留自然皮肤纹理；手脚结构准确，人物四周留白合理，三个全身视图尺度接近。四格全部平视机位，人体比例自然，无透视畸变。高分辨率、细节清楚但不过度锐化，避免塑料皮肤、假 HDR、压缩伪影与重复纹理。

左上格：正面全身图，人物正对镜头，标准放松站姿，双手自然下垂。从头顶到双脚鞋底完整入镜，不得裁切。面部必须完全没有可见五官：不显示眼睛、眼球、眼睑线、眉毛、鼻梁、鼻尖、鼻孔、嘴唇或嘴线。保留自然平滑的肤色、脸部体积感与光影，不得使用模糊、马赛克、头发遮挡或面具效果替代。脸部轮廓、发际线、刘海、耳朵、颈部和饰品保持原样；必要时在生成后仅对面部皮肤区域做局部修复。

右上格：紧凑的正面脸部特写（不是普通肩部以上肖像）。头部正对镜头、眼睛平视，无 3/4 转头、侧脸、明显歪头或俯仰机位。完整保留发顶至下巴，头部约占该格高度的 75%–90%，脸部居中；颈部/肩部仅可在底边少量出现，胸口和上半身不得成为主体。双眼均完整可见且大小自然平衡，鼻梁接近面部中轴，眉毛、眼睛、鼻子、嘴唇和下巴均清晰入镜。保留参考图的发型、眼镜与妆容，但不得遮挡关键五官。中性表情，所有五官锐利清晰。严禁模糊、马赛克、遮挡、失焦或远景肖像构图。

左下格：背面全身图，人物背对镜头，自然放松站立，完整展示从头顶到鞋底的整体轮廓、发型背面与服装后背，双脚和鞋子完整可见。

右下格：严格 90° 左侧面全身图，人物面部与身体朝向画面左边缘，必须是纯侧面，不是四分之三侧面。自然放松站立，从头顶到鞋底完整入镜。

四格必须为同一角色，服装与发型保持一致。禁止文字、水印、标签、额外人物、格子边框、头部或脚部裁切、鞋子缺失、擅自添加装饰、服装改变、四格顺序错乱、人体畸形、多余手指、肢体黏连、光线不一致、色调不一致、塑料皮肤、过度锐化、假 HDR、压缩伪影和重复纹理。无五官规则只应用于左上格；右上格必须完整清晰地显示眼睛、眉毛、鼻子和嘴巴。布局、角度、全身完整度和无五官规则优先于装饰性效果。
```

## 9. 失败后的定向修正

每次修正只针对验收失败项，不要无必要地重写所有角色特征。

| 失败情况 | 修正方向 |
|---|---|
| 四格顺序错误 | 在提示词开头和每格标题中重复固定位置与内容；必要时先分别生成四格再合成 |
| 右下变成 3/4 侧面 | 强化 `strict 90-degree left side profile, facing left, not three-quarter view` |
| 全身格裁切脚部 | 增加人物与画面边缘的留白，明确 `full head-to-sole view, both shoes fully visible, no cropping` |
| 左上仍残留五官 | 不使用模糊；仅在左上格面部皮肤范围进行局部修复，清除眼、眉、鼻、口及残影 |
| 左上脸部出现修补痕迹或轮廓被破坏 | 撤销局部编辑，缩小面部蒙版并重做皮肤融合；不要覆盖发际线、脸部轮廓或耳朵 |
| 左上脸部仍有五官残影或变成恐怖空洞 | 用自然皮肤色调与光影重建面部，去除眼眶/鼻影/嘴线残影；若无法保证自然，标记 `NOT_VERIFIED` |
| 局部修正影响到其他面板 | 撤销本次修改，从原始四宫格恢复其他三格，只替换已处理的左上格 |
| 右上景别过远、变成肩部以上肖像 | 重申 `tight front-facing face close-up, head fills 75–90% of panel height, top of hair to chin fully visible, shoulders only at bottom edge, no torso`；必要时单独生成右上格后再按固定网格拼版 |
| 右上角度不是正脸或变成 3/4 | 重申 `square frontal face, eye-level camera, both eyes visible and balanced, nose centered, no head turn, no tilt, no three-quarter view`；单独生成/修复右上格后重新验收 |
| 右上特写被误删五官 | 恢复原始清晰面部或单独修复右上格，确保完整五官清晰可见 |
| 四格服装细节不一致 | 增强角色锚点，逐项列出关键服装结构；优先基于同一角色图进行局部编辑 |
| 背面服装结构不合理 | 仅补充参考图能够支持的背面结构；不确定部分保持简洁，不凭空添加设计 |
| 风格偏离参考图 | 删除与参考图冲突的风格词，重新强调参考图的媒介、材质、光线和色彩 |
| 画面看起来廉价、塑料感强 | 减少夸张的“超高清/完美皮肤”词汇，强调自然材质响应、真实阴影、细腻但不过度的皮肤纹理 |
| 黑色服装糊成一片或白色衣物过曝 | 降低极端对比与高光强度，要求保留黑色皮革暗部细节、白色布料褶皱和蕾丝层次 |
| 细节过度锐化、出现白边 | 降低锐化和局部对比，优先使用自然细节与高质量放大，不用锐化掩盖结构问题 |
| 四格光线、肤色或色调不统一 | 统一主光方向、色温、曝光与背景；局部修正单格色彩，不重绘角色设计 |
| 擅自增加饰品或花纹 | 加强“faithfully preserve reference, do not invent details”；只保留参考图可见的设计元素 |
| 手脚畸形或材质穿模 | 先修正人体结构与轮廓，再处理材质；不要用细节增强掩盖结构错误 |

## 10. 最终交付格式

交付时简洁报告：

- **版本名称**：版本一 / 版本二 / 版本三（按实际生成数量命名）。
- **验收状态**：PASS / FAIL / NOT_VERIFIED。
- **失败项**：若未通过，列出具体失败的硬性项目。
- **参考图限制**：指出哪些关键区域在原参考图中不可见；若做了推断，明确标注，不宣称完全复原。
- **局部编辑范围**：说明是否实际验证了未编辑区域保持不变；没有验证时写 `NOT_VERIFIED`。
- **文件/图像**：只提供实际存在且可访问的生成结果。

不得把“已生成”当作“已通过验收”，不得声称未检查的项目已合格，也不得为了凑齐版本数量而交付失败版本。