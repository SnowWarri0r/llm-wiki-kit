---
name: study-paper-ingest
description: 给个人学习 wiki 加新 paper / 书 / 章节, 或重写已有页. Use when 写 paper 文章, ingest paper, ingest 论文, ingest 书, ingest 这本书, 做这篇 paper, 做这本书, 学习 paper, 加 paper 到 wiki, 加书到 wiki, 复习 paper, 重写 paper, 写章节精讲, 精讲, bespoke HTML, study wiki, personal wiki, Karpathy LLM wiki, 写 paper 的 wiki, 写 transformer/bert/gpt/resnet/attention 等 paper, 给某本书做精讲, 给 paper 配图, 给 paper 写 bespoke 静态页.
---

# study-paper-ingest · 给个人 wiki 加 paper

> 这是一份**可复用的模板 skill**(Karpathy "LLM Wiki" 模式的实现)。用前先把下面尖括号占位替换成你自己的:
> - `<WIKI_ROOT>` —— wiki 的本地根目录(如 `~/study`)
> - `<GH_REPO>` —— 部署用的 public GitHub repo(如 `you/llm-wiki`)
> - `<PAGES_URL>` —— GitHub Pages 地址
> - `<READER_PROFILE>` —— 读者(通常就是你自己)的背景:会什么、抗拒什么(决定讲解风格,见 3.4)
> - `<DOMAIN_WORDBANK>` —— 你所在领域的口语词库(见 3.5)

Wiki 在 `<WIKI_ROOT>/`，public GitHub `<GH_REPO>`，部署到 `<PAGES_URL>`（GitHub Pages，main 分支 /docs 文件夹）。

## 1 · 文件布局

```
study/
├── CLAUDE.md                  # schema + 工作流（这个 skill 是它的执行版）
├── index.md                   # 顶层目录 md（人读）
├── log.md                     # 时间线，每次 ingest 追加
├── render.py                  # 渲染 docs/index.html + concept/topic/thread/book 自动页 + 注入 nav strip
├── raw/                       # 原始源（只读, raw/books/*.pdf 已 gitignore）
├── wiki/
│   ├── papers/<slug>.md       # 一篇 ML paper 一份 md（→ bespoke HTML）
│   ├── books/<slug>.md        # 一本书概览 + pom-chNN-<title>.md 每章一份（→ 自动 HTML）
│   ├── concepts/<slug>.md     # 跨 paper 概念（→ 自动 HTML）
│   ├── topics/<slug>.md       # 跨源综合（→ 自动 HTML）
│   └── threads/<slug>.md      # 思考线（→ 自动 HTML）
└── docs/                      # GitHub Pages 服务的静态站
    ├── index.html             # render.py 自动生成
    ├── style.css              # render.py 自动生成
    ├── papers/<slug>.html     # bespoke 手工写，纯 ★ 卖点 ★
    └── books|concepts|topics|threads/<slug>.html  # 自动渲染
```

**核心架构原则**：
- md 是内部 ER / 链接 / 给我和 LLM 协作脚手架
- **paper** 面向人的版本是 bespoke HTML —— 每篇独立静态页，独立动画 / 配色 / 排版
- **book / concept / topic / thread** 由 `render.py` 自动渲染，**不**做 bespoke
- `docs/index.html` 由 `render.py` 扫 frontmatter 生成
- nav strip（"← 个人 wiki / index"）由 `render.py` 注入到每个 bespoke 页；不要写死在 bespoke HTML 里
- `render.py` 退出非零 = 有 `[[missing-slug]]` 没补 stub；必须先解决再 commit

## 2 · Ingest 工作流

### 2.A · Paper（ML 论文）

每次新 paper 走这 7 步（**严格按顺序**）；动手写之前先过步 0 的覆盖闸。

**步 0 · 先取全文、以 section outline 当覆盖清单（硬规则 = 广度闸）**

写 paper 前先取**全文**（优先 ar5iv HTML，其次 PDF），并列出论文的顶层 section outline。这份 outline 就是覆盖清单：paper 页要覆盖论文的真实骨架——动机、方法、训练配方、评测（含与谁比的真实数字）、消融、局限——做到**读者单看这页就掌握全貌**。每个核心机制各占一节，配图或数字例；数值一律取论文表格里的真实数。摘要级总结的用途是定位章节，正文内容以全文为准。

这条管**广度**，§3.6 管**深度**，两道闸一起过：覆盖到论文每个顶层 section，且讲到的每个机制都细到能手算。（MRT / Krea 第一版偏薄，就是只过了深度、漏了这道广度闸。）

1. **跟用户对齐做哪篇 paper** —— 如果一篇里有多代（如 GPT-1/2/3），问清楚做哪代或全做
2. **paper md** —— `wiki/papers/<slug>.md`。模板见 `references/paper-md-template.md`
3. **concept md** —— 抽出关键概念到 `wiki/concepts/<slug>.md`。每篇 paper 大约配 3-5 个 concept。已有的 concept 只改 sources 字段不重写
4. **bespoke HTML** —— `docs/papers/<slug>.html`。结构 + CSS 见 `references/bespoke-html-skeleton.html`，图设计原则见 `references/figure-patterns.md`
5. **跑 `python3 render.py`** —— 验证 entries 数量增加 + nav strip 注入成功
6. **更新 `index.md` + `log.md`** —— 加 paper 入口和 concept 分组；log 时间倒序最新在下
7. **git commit + 等用户批准再 push**（关键：见第 6 节）

### 2.B · Book / 章节精讲（非 ML，长篇）

跟 paper 完全不同的形态：

- **不**做 bespoke HTML，全部走自动渲染
- 一本书 = 1 个概览页 `wiki/books/<book-slug>.md` + N 个章节页 `wiki/books/<book-prefix>-chNN-<title>.md`
- 长度：单章 2000-3000 字精讲（深度讲解 + 自由发挥例子），不是 paper 那种 TL;DR
- 写作风格：**直觉先行 + 大量举例 + 中文 native（见 3.5）**
- 用户口令"精讲 / ingest 这本书 / 做这本书" → 走这套

工作流：

1. **跟用户对齐做哪本书 + 一次做几章**（默认 pilot 一章给用户拍板格式 → OK 后继续）
2. **下载源到 `raw/books/`**（PDF/EPUB 自动 gitignore，不进公仓）
3. **PDF 内容抽取**：`pypdf` 读 ToC 找章节边界，按章节范围抽 → 拿到 ideas，**不**照抄
4. **写书概览 `wiki/books/<book-slug>.md`**：一句话 / 它要回答的问题 / 作者背景 / 完整章节 ToC（每章一句话）/ 整本书主线 / 阅读进度 checklist
5. **写章节精讲 `wiki/books/<book-prefix>-chNN-<title-slug>.md`**：见 `references/concept-md-template.md` 但放宽长度。每章固定段落：
   - 本章核心命题
   - 1-3 个书中故事（用自己的话复述）
   - 中国语境对照表（**必备**，3-7 行）
   - 工程师视角 / 类比段落
   - 自检清单（3-5 条）
   - 跟其他章 / 概念的关联
   - 一句话带走
6. **更新 `index.md`** 把章节入口加到对应书下面
7. **更新书概览的阅读进度** checklist
8. **跑 `python3 render.py`** —— 验证 entries++ 且 gate 不报错
9. **更新 `log.md`**
10. **commit + 等用户批准 push**

**重要差异 vs paper**：

- paper 强调 figure，book 没有 figure（看不出来"画什么图"那就别画）
- paper 强调 glossary 闭环，book 不需要
- paper 长度 ~1500 字精炼，book 长度 ~2500 字深度讲
- paper 写好 1 个 commit 推 1 篇；book 一般是 ch 一章一 commit，先拍板再批量
- **paper 配色按领域选；book 默认 brick accent**（已在 render.py 写死）

**版权 / 内容生成边界**（书的特殊提醒）：

- 故事人名、事件、日期、数字这些**事实**可以引用 —— 都是 public knowledge
- 作者的**原创金句 / 长段落论述** 不要逐字翻译，要用自己的话复述 + 改框架
- 用户给的 PDF 链接可能是非官方副本，下载到 raw/ 自用，但仓库里只放笔记不放原文
- 中国版例子必须是自己加的，不抄书

## 3 · 内容写作硬规则

### ⛔ 个人隐私不进公开仓库（硬红线）

chat 里用户分享的**任何个人信息**（薪资、公司名 in 个人语境、职业路径、财务状况、朋友的信息）**绝对不能写进 wiki 页面**。这是公开 GitHub repo。

- **默认不写"套到自己身上"段** —— 这种段落最容易把个人语境写进公开页(实际泄露过)
- 如果要写类比必须**完全泛化**到无法识别个人
- commit 前跑 anti-patterns.md 里的 grep 命令检查

详见 `references/anti-patterns.md` → "内容 · 个人隐私" 段。

### 写作风格

把 `<READER_PROFILE>` 填成读者(通常就是你自己)的背景——会什么、抗拒什么。讲解风格按它定。示例:"懂 Transformer 概念、抗拒数学符号、喜欢工程类比"。下面的规则就是为这种 profile 写的,按你自己的 profile 调:

- **直觉先行**，数学符号尽量替换成代码或类比
- **类比落到熟悉领域**（编程、工程系统、日常物件 —— 已用过的好类比：完形填空、电梯井、办公楼、数据库索引、岗位培训）
- **必给"为什么这么干"** —— 痛点 / 替代方案 / trade-off。永远不写空泛的"X 是一种 Y"定义句
- **每个核心点配一张图** —— 但每张图只表达一件事，别叠加
- 复习时主要看 bespoke HTML —— **HTML 要能脱离 md 单独成立**

### 3.5 · 中文表达必须 native，不要译文味（硬规则）

中文容易带翻译味。把"读起来 native"当成**默认要做的事**，不是被要求才做。每写完一段先自己读一遍，碰到下面这些模式立刻改：

**典型译文味（必改）**：

| 译文味 | 改成 |
|---|---|
| "X 是一种 Y" / "对 X 的 Y" | 直接说"X 就是 Y" / 把定语前置 |
| "让 X 停下来" / "让 X 不再 Y"（moving the goalposts 这种） | 找中文已有的对应说法（"知足"/"喊停"/"胃口别跟着饭量长"） |
| "进门那一刻就立刻退出" | "进门就走" |
| "结果在你眼前展开" | "你能看见" |
| "...这件事能教会我们" | "...这件事告诉你" / 直接结论 |
| "得到更多带来的快感" | "尝到甜头" / "拿到更多之后" |
| "净 X 始终在原地" | "感觉永远在原地" / "你永远觉得没动" |
| "...的程度" | 直接形容 |
| "他大概回答" / "他这样表达" | 直接转述, 不加叙述层 |
| 长定语后置: "我做出的判断主要是从最近 5 年的经历里来的" | 短句串联: "我这判断, 主要来自最近 5 年" |
| 中英混用术语 (distribution/mean/mode/N=1) 没必要时 | 全中文 (分布/规律/噪声/单个样本) |
| "什么都行" / "什么都一样" 这种语义模糊 | 选明确的: "都一样" / "一回事" / "都没区别" |
| 形容词后置: "钱多到不可想象" | 前置: "钱多得没数" |
| "把 X 推到 Y 的临界点" | "把你推过头" / "把你折进去" |

**主动用的口语 / 成语 / 中文味更强的表达**：

斩钉截铁 / 不在一个数量级 / 卷到 burn out / 翻车 / 上车 / 上头 / 心痒得睡不着 / 铤而走险 / 寒酸 / 没头没尾 / 数量级 / 想冲 / 站上山顶 …（这是示例;按你常用领域和母语口癖维护一份 `<DOMAIN_WORDBANK>`,越贴你越好)

**自我检查**：写完一段读三遍，问自己 ——

1. 这一段如果是一个中文母语 + 工程师的朋友说给我听，他会用这些词吗？
2. 有没有"X 的 Y"句式可以拆成短句？
3. 有没有外来概念能找到中文已有的对应说法？

发现一处改一处。**不要等被提醒**。

### 3.6 · 数学/抽象概念必须带"能手算的数字例子"（硬规则）

抽象概念最容易因为"太抽象"被打回（FFT 不举例怎么算 / 公式没例子 / 归一化要真数字 / SVD 只有公式没手算 …）。这是**默认必做**，不是被要求才补。

**规则**：任何带公式、矩阵、变换、分布、概率、损失的概念页，正文里**必须有一段 `## 数字例子`**，要求：

1. **用具体小数字**（2×2 矩阵 / [2,1,0] 这种），不是符号 `a,b,W`
2. **把每一步算给人看**，中间结果写出来，让人能拿笔跟着验算
3. **能自检的收尾**：算回去对得上（"✓ 加回来正好是 W"）、或跟暴力解对照
4. 抽象陈述（"σ 大 = 信息多""低秩 = 可压"）后面**立刻**跟一个数字把它坐实

**反例（禁止）**：只写 `W = UΣVᵀ`、`pᵢ = exp(zᵢ)/Σexp(zⱼ)` 然后就结束 —— 没有代入数字 = 没讲清。
**正例**：取 `W=[[2,1],[1,2]]`，`W·[1,1]ᵀ=[3,3]=3×[1,1]` → σ₁=3，一步步拆成秩 1 块再加回验证。

挑例子的技巧：选**能手算**的特例（对称矩阵省旋转、2×2、整数、归一化 /√2），宁可特殊也要可验算；真实大矩阵再用一句"同理几千个 σ…"带过。

**深度标准（默认要求,以后都这么细）**：例子要做到 AdaLN 那页的程度——

1. **一条贯通的 running example，不要断开的小片段**。从一个具体输入(如 `x=[1,5]`)**从头算到尾**，每一步的中间结果都写出来；别像旧版那样"调制用一个例子、门控又换一个例子"两段拼起来。
2. **先把前置零件讲了再上例子**。算 AdaLN 前先把普通 LayerNorm 三步摊开(归一化→γ缩放+β平移→下一层)；读者不知道"归一化后的 x̂ 哪来的"就跟不动。**别假设读者记得上游概念**——当场用数字带一遍。
3. **每个符号当场用数字坐实它"是什么"**：γ 不只是符号，要说"每通道的音量"并用 `γ=[2,0.5]` 算给看；β 是"底噪"。
4. **要让"为什么需要它"在数字里浮出来**：AdaLN 用 `t=A` 和 `t=B` 两组 γ/β 算出不同的 h，当场坐实"同一个 x̂、t 不同→输出不同 = 条件拧旋钮"，这比一句"它能注入条件"强十倍。
5. **算完 python 复核一遍**再写进去（arithmetic 错了比抽象更糟）。

旧版 AdaLN(粗)→ 新版(达标)的差距就是这五条；以后所有数学/机制页按新版的细度写，**不是被要求才补**。

#### 3.6.1 · 多步公式推导怎么写（标杆参考:一篇 RL 论文里 `v_θ*=v_old+(2/β)Δ` 那种多步最优解推导）

`## 数字例子` 是"拿数字坐实一个概念"；**推导一个结果**（解最优、化简恒等式、证一个公式）是另一种活——一长串代数最容易写成"符号代码块堆叠"，读者看完一头雾水。一次"解最优"的长推导被打回重写后,定下七条：

1. **先给路线图**。开头一句话列出 3-5 个 beat（"①损失是碗→斜率0 ②解出平衡式 ③备两个事实 ④代回解出"），让读者知道要去哪，长推导才不晕。
2. **prose 主导，不是符号代码块堆叠**。每一步先用一句大白话说**这步在干嘛、为什么、得到的东西什么意思**，再上算式。算式塞进 fenced block，**话写在 block 外**。
3. **先备工具 / 记号**。推导要用到读者可能忘了的运算（期望、求导、链式法则、对数）→ 先给两行小抄（"期望=加权平均，两规则：常数拎出 / 加减拆开"），并**当场点明谁是常数谁是变量**。别假设读者记得。
4. **直觉领每一步**。先说直觉再上代数（"损失是开口朝上的碗，碗底斜率=0，所以令导数=0"），代数只是把直觉落实。
5. **一条 running numeric 贯穿到底**。挑一组好算的数（两样本、ρ=0.5、整数），**每一行算式都用它验一次**，让读者能拿笔跟着走；别只在结尾验一次。
6. **不许跳步**。"归拢同类项即得""化简可得"是禁语——代入、乘开、移项、提公因数每步都摊开。读者最容易卡的恰恰是跳步处（如期望的线性、参数化代入）。一行里若有"⟹"跨度大，就拆成 (i)(ii)(iii) 小步。
7. **python 核验，且验平衡+不平衡两种取值**。恒等式只在对称特例（如 ρ⁺=ρ⁻）成立的话会骗过自己；至少再跑一组不对称的数确认通用。

自检：写完一段长推导，问"一个忘了期望/求导的工程师，单看这页能不能一步步跟到底，每步都知道为什么？"——卡住的地方就是要补的地方。

### 3.7 · 页面必须独立成文，不许把聊天语境写进去（硬规则）

页面是给**未来的用户复习**用的，要脱离当下这次对话能独立读懂。**禁止**在 paper md / concept md / bespoke HTML 正文里出现指回聊天的话：

- ❌ "用户问的 X 就是它" / "你问的 X" / "刚才说的" / "前面聊到" / "我们上面讨论的"
- ❌ 把用户的口误 / 提问方式当成正文的一部分（x-vector 翻车："用户问的'Kaldi 的 lvector'指的就是它"）
- ❌ **假设读者已学过其它 wiki 页**："你学的 ResNet" / "你已经懂了" / "接你学的 X" / note 标签 "放进你学的"。个人 wiki 是非线性导航，读者可能直接搜到这页，没按设想顺序读过。这是个**口头禅**，散在一堆页的开头和 closing note 里（flux/sd3.5/qwen-image/qwen3-asr/lstm/mfcc/flash-attention 全中过招）。改成中性引用：链接照给，但说"ResNet 的残差""把这几块拧在一起""放进 wiki 这条线"，别预设读者读过。**例外**：book 页第二人称是原作修辞口吻，保留。

**修法**：把同样的信息改成**独立陈述**——
- ❌ "用户问的 lvector 就是它" → ✅ "它在 Kaldi 里就是那个标准声纹 recipe（容易被记成 l-vector 之类，其实是 x）"
- 想保留"别名 / 易混淆点"是对的（对复习有用），但要写成"X 常被叫成 Y / 容易跟 Y 搞混"，不是"你问的 Y"。

自检：写完每段问一句——**一年后的我没有这次对话记录，单看这页读得通吗？** 出现"用户/你问/刚才/我们聊"立刻删。这跟个人隐私红线（3 节）同源：聊天是私的，页面是公开且要长期独立存活的。

## 4 · Bespoke HTML 关键约束

完整 skeleton 在 `references/bespoke-html-skeleton.html`。要记住的硬规则：

**结构**：
```
masthead → hero (issue / title / byline / dek) → tldr → 
§01 故事/背景 → §02-N 核心内容（每节配一张图）→ §N 历史定位/遗产 → 
glossary → colophon
```

**配色**：每篇 paper 一个 primary accent color（区别于 wiki 其他文章）：
- brick (`#9b2c2c`)·红 · GPT / 生成 / decoder-only / attention
- moss (`#4a6b3a`)·绿 · BERT / 理解 / encoder-only
- deep (`#1f3a5f`)·蓝 · ResNet / foundation / 基础设施
- ochre (`#b8841c`)·金 · 备用 / 强调

调色板和染色逻辑见 `references/colors.md`

**图设计原则**（每张都必须满足）：
1. <strong>一张图一个 point</strong>，别把反向 / 正向 / 比较塞同一张
2. 比较类用<strong>左右并排 panel</strong>，不要叠加箭头到同一图上
3. 多步演示用<strong>独立 stepper</strong>，每步切换 CSS class
4. <strong>所有数值要真</strong>：不写"big" "small"，写 590K · 69K · 1/17
5. <strong>动画用 IntersectionObserver 触发</strong>，stagger delay 让人脑跟得上
6. 看具体演化模式见 `references/figure-patterns.md`，看翻车案例见 `references/anti-patterns.md`

**图：重画 vs 链原图（约定）**：

- **机制 / 示意图**（架构栈、流程、坐标曲线、数据流）→ 一律**重画成内嵌 SVG**。这是 bespoke 的价值：一图一点、中文标注、配能手算的数字例、配色动画统一；重画也逼自己真懂。
- **定性结果**（生成样图、真实分割 mask、照片、真实 benchmark 曲线/榜单截图）→ 重画会失真，但**不要内嵌原图**——llm-wiki 是公开 repo，内嵌论文/博客的图 = 复制受版权素材（红线），且 hotlink 会烂链、风格会跳。正确做法是**链出去**：正文或 colophon 加一行 `↗ 原图/样图见<原文/项目页>`，链到论文或官方主页，让读者点过去看官方的。
- 链接用页面 frontmatter 里已有的 `source` / `upstream` URL，**别杜撰具体 Fig 编号**（没核实就只说"见原文/项目画廊"，别写"见 Fig 3"）。

**Glossary 浮卡（render.py 自动）**：点正文的 `.jr` 现在不再跳页底，而是在**右侧浮出一张卡片**显示该条释义（窄屏=底部卡片）。这段 CSS+JS 由 `render.py` 的 `inject_glossary_popover()` 自动注入到每个有 `class="gloss"` 的 bespoke 页（marker `gloss-popover:start/end`，幂等）——**别手写这段**。底部 `<ol>` glossary + `#g-NN` 锚点保留作 no-JS / 打印兜底。所以 glossary 的 `<li id="g-NN">` 和正文 `.jr` 入口照常写，浮卡白送。

**Glossary**：12 个左右，每个都<strong>必须</strong>有一处正文 `<a class="jr" href="#g-NN">` 跳转入口 —— 没入口的孤儿条目 = bug。提交前必跑：
```bash
diff <(grep -oE 'href="#g-[0-9]+"' docs/papers/<slug>.html | sort -u) \
     <(grep -oE 'id="g-[0-9]+"' docs/papers/<slug>.html | sort -u)
```
有 diff 就有孤儿。

## 5 · 文件相互引用

paper md 链接到 concept 用 `[[concept-slug]]`（Obsidian 风格）。bespoke HTML 之间用相对路径 `<a href="other-paper.html">`。

每个 concept md 的 `sources:` frontmatter 必须包含<strong>所有引用它的 paper slug</strong>。新 paper 引用旧 concept 时记得回去补 sources。

## 6 · Push 流程（用户硬规则）

**任何 push、gh comment、release notes、对外消息都必须先给用户看等批准再发**。流程：

1. 本地 commit 完 → 列 commit summary 给用户
2. 等用户回"要 / push / yes" 之类批准
3. 接到批准再 `git push origin main`

绝对禁止：自己看着 commit 觉得对就直接 push。

## 7 · 提交前 checklist

```
[ ] wiki/papers/<slug>.md  存在且 frontmatter 全 (name/type/source/upstream/ingested/authors)
[ ] 每个新 concept md 存在且 sources 字段对
[ ] docs/papers/<slug>.html 存在
[ ] python3 render.py 跑过且 entries 计数对
[ ] index.md 加入口
[ ] log.md 加 ingest 时间倒序条目
[ ] 跑 glossary 跳转 audit (refs vs ids diff 为空)
[ ] 跑 fig id audit (确认所有 fig id 都在 HTML 中)
[ ] commit message 描述 5 张图分别在演什么
[ ] commit 完先给用户看，不要直接 push
```

## 8 · 翻车案例（一定要避免）

来自 ResNet revamp 教训：
- ❌ 4 步 stepper 在同一 SVG 上叠加正向箭头 + 反向梯度 → 视觉打架
- ❌ 抽象柱状图说"plain 要学恒等 vs res 学 F=0" → 不够 visceral，应该展示<strong>随机初始化下输出实际长什么样</strong>
- ❌ 列参数表不显示计算 → 应该写 `256 × 256 × 9 = 590K` 把账算给人看
- ❌ "SGD 走不到好解"这种空话 → 应该给思想实验（"白送 36 层 identity"）

来自 BERT 教训：
- ❌ glossary 写 12 条但正文只有 5 个跳转 → 7 个孤儿
- ❌ 概念在 md 提到但 HTML 没引用 → 切换语义脱节

## 9 · 示例标杆（都是公开论文,供参考"图做到什么粒度算到位"）

### Papers（bespoke HTML）

| paper | accent | 看 figure |
|---|---|---|
| attention-is-all-you-need | brick | RNN vs Transformer / self-attention 4 步 / multi-head 4 panels / PE 热力图 |
| resnet (revamp v2) | deep | degradation 双曲线 / residual block + 梯度 5 层 bar / 随机 init 输出对比 / 深度 5 栏 / bottleneck flop math / Transformer 同款 |
| bert | moss | bidirectional 双箭头 / MLM 80/10/10 stepper / NSP tokens / Transformer 3 砍法 / pretrain→4 task 扇出 |
| gpt-1/2/3 | brick | CLM 训练 / causal mask / WebText 漏斗 / scaling law / ICL one-shot |
| flow-matching | deep | velocity field + probability path 对照 / OT 直线 vs diffusion 弯路 |
| dmel | ochre | log-mel 单 SVG 数据流（值→bin→token 一条直线）|
| fish-speech-s2-pro | (混) | Dual-AR / Neural codec / RVQ 残差递推 / GRPO / bench |
| interaction-models-tml | (混) | 200ms 流 / dual model / early fusion / flow matching / bench |

新 paper 选 accent 时避开同领域已用的颜色。

### Books（自动渲染，无 bespoke）

| book | 章节进度 | 风格 |
|---|---|---|
| `<某本非 ML 书>` | 每章一页 | 2500 字/章精讲；每章必备本地语境对照表 + 自检 + 一句话带走；2-3 轮 native 润色才稳 |

ingest 书的时候 **3.5 native 规则 + anti-patterns.md "中文表达"段** 是默认必读。

## 引用的子文件

- `references/paper-md-template.md` —— paper md 完整模板
- `references/concept-md-template.md` —— concept md 模板
- `references/bespoke-html-skeleton.html` —— bespoke 页骨架（CSS + 结构 + JS observer 模板）
- `references/figure-patterns.md` —— 已经验证好用的图设计模式 5 种
- `references/anti-patterns.md` —— 翻车案例与教训
- `references/colors.md` —— 调色板 + 已用配色记录
