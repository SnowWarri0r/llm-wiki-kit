# 翻车案例与教训

每条都是真实 ingest 时被指出 / 发现的问题。

## 视觉

### ❌ 多步动画叠在同一 SVG

ResNet 原版 Fig 02 把正向箭头（step 1-3）和反向梯度（step 4）画在同一个 SVG 上，反向闪烁线穿过正向箭头 —— 视觉打架，看不出哪个箭头表达什么。

**修法**：拆成两个独立 panel。左边静态画 block 结构，右边专门展示"梯度通过 5 层"的 bar chart。每个 panel 表达一件事。

### ❌ 抽象柱状图说"plain 学恒等 vs res 学 F=0"

ResNet 原版 Fig 03 用两组等长的彩色条说"plain 要学 H=x（输入和输出一样长） vs res 要学 F=0（输出全是矮条）"。看起来对，但<strong>不够 visceral</strong> —— 读者不知道这跟"SGD 起点"有什么关系。

**修法**：换成"随机初始化下输出实际长什么样"：
- plain 输出 = 噪声（随机条）
- res 输出 ≈ 输入（保持模式）
- 直接展示"SGD 起点"的差异，不需要解释

### ❌ 列参数表不显示计算

ResNet 原版 Fig 05 列了 basic block ~74K params / 110M flops 和 bottleneck ~70K / 110M。看起来差不多，没传达"为什么 bottleneck 省"。

**修法**：把计算账写出来：
```
直接 3×3 256→256: 256 × 256 × 9 = 590K (×2 = 1.18M)
bottleneck: 16K + 37K + 16K = 69K
ratio: 1/17
```
让人亲眼看到 1×1 多便宜、3×3 在窄腰多省。

### ❌ 三栏 grid 把"input → process → output"切成三块独立面板

dMel 原版 Fig 02 用 grid 1.5fr/1fr/1.2fr 三栏：左 channel 值条形，中间 16 bin 切片，右 token 输出。读者会反馈"看不出这东西变成那东西" —— 三块之间<strong>没有视觉连接</strong>，读者得自己脑内补"哦 mel 3 的值 .82 对应中间 bin 13，然后变成右边 token 13"。

**修法**：用<strong>单 SVG 把整条数据流装进一个画面</strong>：
- 6 bar 跟 6 hit-cell 横向高亮跟 6 token box 在同一画布上
- bar 顶部所在的 bin 行整行高亮 → 直接延伸到右侧 bin idx label
- ochre 虚线 connector 从 bar 顶画到右侧 label
- "值 → bin → token" 三个阶段视觉上一条直线

**通用原则**：<strong>"过程图"用单画布</strong>。如果要表达 A 变成 B 变成 C，A/B/C 要在同一 viewBox 里有空间关系（哪怕只是横向排列 + 箭头连接），而不是 3 个独立 grid 单元让读者脑补对应。

### ❌ 在数据网格上叠 "channel" / "value" / "bin idx" 文字标签

dMel Fig 02 第一版加了 "channel:" "value:" "bin idx:" 三个 header 标签想说明"上面这行是 channel 名字、下面是 value 值"。这些 header 用 text-anchor=start 从 x=58 起，跟 chart 列上的 "mel 0" 标签直接撞了。

**通用原则**：<strong>如果数据本身能自解释，就别加 header</strong>。"mel 0" + ".72" 上下排列，谁都看得出第一行是 channel、第二行是 value。强加 header 反而增加视觉噪声 + 容易撞位。

### ❌ y 轴 / x 轴 / divider 位置 / 文本位置只看 viewBox 坐标，不预估实际像素重叠

dMel Fig 02 第二版 "→ TOKEN" 标签在 y=358，section-divider 在 y=338。math 上 20px gap 够，但<strong>视觉上虚线穿过文字</strong>因为没考虑文本高度 + viewBox 缩放后的实际像素密度。

**修法**：
- viewBox 留充足 vertical breathing room（原本 400 → 420 留 token 区独立 zone）
- 每个语义层之间至少 20px viewBox y gap
- 文本和虚线之间至少 14px viewBox y gap（考虑文字高度 + 行高）
- 复杂 SVG 提交前 mental render 一遍，或者真的打开浏览器看

### ❌ 文字标签被线段 / 曲线 / 箭头穿过

多次翻车(gradient-backprop 的"现在 w"被 loss 曲线穿过、covariance 长轴标签压住箭头+散点、lumine FIG02 箭头压住"控制 30Hz")。

**关键认知:光靠 z-order 不够。** 把文字画在线之后(DOM 靠后)只能让字"在上层",但**细线照样会从笔画的缝里透出来**,中文字尤其(横竖之间空隙大),看着还是脏。结论:**文本层级要比线段高,要不就挪到旁边去——挪开才是真解**。

**修法(按优先级)**:
1. **把标签挪到明显空白处**——优先做成**图形占一侧 + 标签做另一侧的图例栏**(色块 + 文字),图例区跟图形区完全不交叠(见 covariance 重排:左椭圆 / 右图例)。
2. 标签必须贴近某元素时,放到该元素**确实空的那一侧**(先在脑里 render 周围曲线/箭头的走向),别拍脑袋。
3. 兜底:文字底下垫一个 **paper 色背景 rect**(`fill` 用该 figure 的背景色)盖住线,再画文字。
4. **曲线/路径上的点必须按方程算坐标**(贝塞尔就代 t 算 x,y),不能目测 y——gradient 那个"现在 w"点第一版就是 y 拍脑袋写、浮在碗中间半空。

字号:图例 / 标签正文 ≥ 11(viewBox 单位),标题 13-14;8-9 太小,显得局促。

**回环 / 反馈弧的 caption(relat fig-loop / fig-relat 踩过两次)**:带反馈弧的图(A→…→B 再一条弧拐回 A),弧线常横扫底部、跟那行说明文字撞一起。修法:弧线的最低点收在文字**上方**,caption 放在弧线**下方**的空白带,各占一条水平带、互不交叠;viewBox 高度留够容下"弧 + 字"两层。

### ❌ 反馈弧/曲线箭头是竖直三角,但线斜着到达 → 头线不同向

relat 两条反馈弧踩过:虚线斜着拐进目标框,箭头却是个朝正上的竖直三角,头与线明显错位。修法二选一:
1. **让弧线 end 切线对准到达方向**(贝塞尔 control2 相对 end 做对角偏移,使最后一段是斜的),再用 `marker orient="auto"` 让箭头**随路径切线自动旋转**——头永远跟可见线段同向。**适用 opacity 淡入图**(`.rl-rev` 这类整体 fade),marker 跟着一起淡入,无抢跑。
2. `.draw`(stroke-dashoffset 描边)动画的线**仍用独立 polygon**(marker 会在描边前就显形=抢跑);此时手算切线方向、按方向摆 polygon 三角。

### ❌ 在 SVG `<text>` 里用 HTML `<strong>` 加粗

cosmos-3 FIG02 图例写了 `<text ...>… → 为<strong>省算力</strong></text>`，结果"省算力"被吞，只剩前面的"为"——**SVG `<text>` 不认 HTML `<strong>`/`<b>`/`<em>`**，非 SVG 元素的文字内容不渲染。

**修法**：SVG 里加粗用 `<tspan font-weight="700" fill="...">省算力</tspan>`，或整个 text 元素 `font-weight="700"`。提交前 grep：
```bash
grep -rnE '<text[^>]*>.*<strong>|<tspan[^>]*>.*<strong>' docs/papers/*.html docs/concepts/*.html
```
（注意：正文 `<p>` 里的 `<strong>` 是合法 HTML，别误改；只查 SVG `<text>`/`<tspan>` 内的。）

### ❌ Glossary 写 12 个，正文只有 5 个跳转

BERT 原版做了 12 个 glossary 但正文只 sup 引用了前 5 个 —— 6-12 是孤儿，没人能从正文跳过去看。

**修法**：commit 前必跑：
```bash
diff <(grep -oE 'href="#g-[0-9]+"' docs/papers/<slug>.html | sort -u) \
     <(grep -oE 'id="g-[0-9]+"' docs/papers/<slug>.html | sort -u)
```
有 diff 就是有孤儿，找正文合适的句子塞 `<a class="jr" href="#g-NN">N</a>`。

### ❌ Inline 写死 wiki-nav

最早某篇 paper 把"← 个人 wiki / index"导航栏写死在 bespoke HTML 里,其他 paper 没写,导致部分页缺"回 wiki"按钮。

**修法**：导航是<strong>系统层的 chrome 不是内容</strong>。让 render.py 统一注入。bespoke 页只写正文，nav strip 由 `inject_nav_strip()` 在 `<body>` 后注入（marker bounded，幂等）。

## 内容

### ❌ 默认读者懂领域缩写 · 不加术语扫盲

flow-matching ingest 原版 TL;DR 直接抛 ODE / SDE / OU / score / Tweedie's
formula 这些术语，假定读者已读过 diffusion paper。<strong>读者会反馈"好多名词
看不懂"</strong>。

**修法**：在 TL;DR 跟 §01 之间加一个 "vocab-box" 术语速查盒：
- 2×2 网格放 N 个核心缩写
- 每张卡：黑底 code 缩写 + Fraunces 斜体中文 + 一句话直觉 + 谁用
- ODE → "粒子在风场里飞，起点+风场 → 轨迹完全确定"
- SDE → "ODE + 每步随机扰动，像粒子被空气分子撞"
- 不展开数学，只给 visceral 类比

**判断什么 paper 需要 vocab box**：
- ✅ 数学/物理向 (flow matching / diffusion / continuous normalizing flow)
- ✅ 涉及 3+ 缩写不展开就难懂的领域 (audio codec / RL specific terms)
- ❌ 架构/方法向已经有 KEY figure 把核心机制讲明白的 (BERT / GPT / ResNet)
- 简单判断：如果 TL;DR 第二句话需要查另一篇 paper 才看得懂 → 加 vocab box

### ❌ "SGD 走不到好解"这种空话

ResNet 原版 §02 closing note 写"SGD 在 plain net 的优化几何里走不到那个本来该存在的好解"。听起来对，但<strong>读者并不真的明白这是个问题</strong>。

**修法**：给思想实验。"白送 36 层 identity"：
- 手上有 20 层网络，train loss 9%
- 给你免费加 36 层，且这 36 层<strong>什么都不做</strong>（identity）
- 你的 56 层网络最差也该 9%
- <strong>但 plain CNN 没有"什么都不做"这个选项</strong>
- 残差就是<strong>把"什么都不做"做成架构里的选项</strong>

这个思想实验让 degradation problem 立刻变得有形。

### ❌ "F(x) 就是要学的差量"没解释

ResNet 原版 §03 直接抛公式 `y = F(x) + x`，配一句"命名误导，跟统计 residual 没关系"。读者还是不懂 F 是啥。

**修法**：先讲对照。"原版 plain block 学 H(x)；残差 block 输出 F(x) + x，所以 F = H - x。F 学的是 H 跟 identity 的差。"再配一个具体例子（"提亮 1.05x"），让人看到 F 比 H 永远小。

### ❌ 列"4 个 head 学不同模式"没具体例子

attention Fig 03 原本只画了 4 个 panel 各自连不同的边。没说<strong>每个 head 各自在学什么</strong>。

**修法**：4 个 panel 配 4 个具体语言学角度：
- head 1: 句法（主谓宾）
- head 2: 邻接（相邻词）
- head 3: 词性
- head 4: 语义角色

这样读者知道"多头不是冗余，而是分工"。

### ❌ 把聊天语境写进页面正文（"用户问的 X 就是它"）

x-vector ingest 翻车：正文写了"用户问的'Kaldi 的 lvector'，指的就是它"——把这次对话里用户的提问（还带口误 lvector）当成正文的一部分。页面是给未来的用户复习用的，**一年后他没有这次对话记录**，看到"用户问的"完全摸不着头脑；而且这等于把私的聊天语境泄进了公开长期页面。

**修法**：改成独立陈述。想保留"别名/易混点"是对的（对复习有用），但写成"X 常被记成 Y，其实是 X"而不是"你问的 Y"：

```
❌ "用户问的'Kaldi 的 lvector'指的就是它"
✅ "它在 Kaldi 里就是那个标准声纹 recipe（容易被记成 l-vector 之类，其实是 x）"
```

### ❌ 假设读者"已经学过"其它 wiki 页（"你学的 X""你已经懂了"）

同一个病根的高频变体，而且是个**口头禅**——散落在一堆页的开头和 closing note 里：

```
❌ "前面那几步…你已经懂了"          (mfcc)
❌ "接你学的 ai-memory-hierarchy"     (flash-attention)
❌ "FLUX 把你学的几块重新排了次序"     (flux / sd3.5 / qwen-image / qwen3-asr 全中招)
❌ "这跟你学的 ResNet 残差…"          (lstm)
❌ <span class="l">放进你学的</span>   (ode-sde 的 note 标签)
```

个人 wiki 是**非线性导航**的——读者可能从搜索直接落到这页，没按你设想的顺序读过 ResNet / log-mel。"你学的 X"假设了一条根本不存在的阅读路径。

**修法**：把其它概念**中性引用**，链接照给，但别预设读者读过：
- ❌ "你学的 ResNet 残差" → ✅ "ResNet 的残差"（链接保留）
- ❌ "把你学的几块拧在一起" → ✅ "把这几块拧在一起"（"这几块"指就近列的那些链接）
- ❌ "你已经懂了，出来是 log-mel" → ✅ "合起来就是 log-mel"
- ❌ note 标签 "放进你学的" → ✅ 统一用 "放进 wiki 这条线"

**例外**:book 页里的第二人称是**原作的修辞口吻**("你学过的理论""不花你已经有的钱"),那是源内容不是 wiki 串联,保留。所以 grep 要**排除 books/**。

**自检**：写完问一句"脱离这次对话 + 没读过别的页，单看这页读得通吗？"。出现 `用户问 / 你问 / 刚才 / 前面聊 / 你学的 / 你已经懂 / 放进你学的` 立刻改。commit 前 grep（排除 books/，那里第二人称是原作的）：

```bash
grep -rnE '用户问|你问的|刚才说|前面聊|我们(上面|刚才)聊|你之前(说|问)|你已经懂|你学的|你学过|放进你学的|你.{0,4}页学|点回来|一路点|终点就是这|你从.{0,8}(线|页|那条)' \
  wiki/papers/ wiki/concepts/ wiki/topics/ docs/papers/ docs/concepts/ docs/topics/ 2>/dev/null
```

**导航预设子变体**（reintroduced 在 sam 页）：closing note 爱写"所以你从 OCR 那条线**点回来**，**终点就是这**""从 X / Y **一路点回来**，就到这"——预设了读者的点击路径。改成中性陈述事实：✅"SAM 是 unlimited-ocr / optical-context-compression 那条 OCR 压缩线的**上游**"。表达"这几页有关系"用"X 是 Y 的上游/下游/姊妹"，不要用"你从…点回来"。注意 grep 会误命中合法的修辞 `你从碎片里提炼`、analogy `顺着箭头滑`——逐条人工确认，只杀"指代 wiki 导航路径"的。

详见 SKILL.md 3.7。

## 内容 · 个人隐私（公开仓库硬红线）

如果 wiki 是 public repo,有**两种不同的泄露**要防:一是**雇主的内部信息**(内部服务名 / 内部 API,见下一节);二是**你本人在聊天里随口说的个人信息**(薪资、现/前雇主、职业路径、财务状况、朋友的事)被写进 wiki 页面。这一节讲第二种。

### ❌ 把 chat 里随口说的个人信息写进 wiki 页面

最常出在"把这篇/这本书套到自己身上"那种段落里。聊天里为了讲解打的比方(收入数字、在哪家公司、跟谁不对付、攒了多少钱)很容易顺手写进正文,推到 public GitHub 就成了永久公开。

### 判断规则:泛指可以,指向具体个人不行

| 类型 | 能不能写进公开 wiki |
|---|---|
| 某公司作为通用商业案例(财报 / 战略) | ✅ 可以 |
| 某公司作为"我之前在那工作"的私人信息 | ❌ 绝对不行 |
| "大厂年包从 X 涨到 Y"(泛指行业现象) | ✅ 可以 |
| 指向具体个人的薪资数字(我的 / 朋友的) | ❌ 绝对不行 |
| "留 6 个月 buffer"(通用建议) | ✅ 可以 |
| "我已经攒到 N 个月 buffer"(我的财务状况) | ❌ 绝对不行 |
| 我的职业路径 / 现雇主 / 前雇主 / 离职原因 | ❌ 绝对不行 |
| 朋友 / 家人的任何具体信息 | ❌ 绝对不行 |

### 修法 · 怎么写"套到自己身上"段而不泄露

1. **默认不写"套到自己身上"段** —— 这种段落 99% 的风险来自这里。
2. 实在要写类比就**全部泛化**(把具体数字/公司/人换成"如果你在一家…"的通用条件句);即便泛化了也再问一句:**读者能不能从这段文字 + 公开 commit 历史推出是谁?** 能 → 删。
3. **聊天里说的个人信息永远不进 wiki** —— 硬红线,跟"内部服务名不进公仓"同级。

### commit 前 grep 命令(按你自己的敏感词改)

```bash
# 把下面替换成会暴露你个人身份的词:薪资数字、现/前雇主名、职业身份词、财务数字等
grep -rnE '[0-9]+k|<我的雇主>|<前雇主>|buffer.*攒|我朋友|你朋友|我选.*不选' \
  wiki/ docs/ index.md log.md 2>/dev/null | grep -v '\.html:.*style\|\.html:.*script'
```

任何输出都人工确认是泛指还是指向具体个人。

## 内容 · 中文表达（必看，每次都要应用）

一旦读者反馈"还行但不够 native",说明**默认写出来的中文有强翻译味**,必须主动改。下面是实际 ingest 过程里被点名修过的典型译文味(每条都改进过 2-3 轮),可当通用检查表。

### ❌ 英文俚语直译

| 原文 | 直译 | 改后 |
|---|---|---|
| moving the goalposts | "让球门停下来" | "胃口别跟着饭量长" / "知道在哪喊停" |
| getting accustomed to | "习惯于...这件事" | 直接动词 "适应 / 跟上" |
| the only way to win | "唯一的赢法是" | "唯一能赢的玩法是" / "想赢就只能" |
| exit as soon as you enter | "进门那一刻就立刻退出" | "进门就走" |
| break the news | "公布这个消息" | "把消息告诉" |
| broke and ashamed | "破产并且羞愧" | "又破产又抬不起头" |

**判断**：搜中文里有没有现成对应表达。中文有"知足常乐 / 见好就收"就别用 "moving the goalposts" 的直译。

### ❌ 长定语后置 / "X 的 Y" 句式连环

```
❌ "我做出的判断主要是从最近 5 年的经历里来的"
✅ "我这判断, 主要来自最近 5 年"

❌ "这是一个想拿回声誉但知道拿不回来的人在给自己找台阶的说法"
✅ "这听着像一个想拿回声誉但知道拿不回来的人, 在给自己找最舒服的台阶下"

❌ "把潜在的收益留在桌上"
✅ "把潜力留在桌上没拿"
```

**修法**：写完一段，每出现"X 的 Y 的 Z" 三层嵌套，强制拆成两个短句。

### ❌ 中英术语混杂没必要

```
❌ "把 N=1 的样本当成整个分布的 mode 来抄, 几乎一定踩雷"
✅ "把单个样本当成普适规律去抄, 几乎一定栽跟头"

❌ "工作过劳到 burn out"
✅ "卷到 burn out" (burn out 已经中文化, "工作过劳到" 是译文味)

❌ "你看到的每个'案例'都是从某个分布里抽出来的样本"
✅ "你看到的每个'成功案例'都是一次抽样"
```

**判断**：术语只在有共识时保留（GPU / RL / LLM / token）。distribution / mean / mode / regression / probability 这些有正经中文（分布/均值/众数/回归/概率）的都翻。

### ❌ 译文式连接词 / 加叙述层

```
❌ "他大概回答" + 引号
✅ 直接转述

❌ "Housel 给一条很硬的方法论:" + 长引号块
✅ "Housel 这章给一条挺硬的规则:" + 直接用我的话讲

❌ "对此 Housel 觉得这是这件事能得出来的最差结论"
✅ "Housel 觉得这是从这件事里能得出的最差结论"
```

### ❌ 形容词后置 / 副词后置

```
❌ "钱多到不可想象"     → "钱多得没数"
❌ "富到不可想象的人"   → "钱多到没数的人" 
❌ "净满足感始终在原地" → "感觉永远在原地踏步"
❌ "饭好吃的程度大太多" → "那顿饭好吃带来的爽"
```

**修法**：中文倾向 "形容词 + 得 + 程度副词" 而不是 "X 到 Y 的程度"。

### ❌ 给读者"输出错处"或者"作业感"过重

```
❌ "想清楚就会发现 80% 来自最近 5 年"  → 偏说教
✅ "想清楚常常会发现: 八成来自最近 5 年"  → 给条件 + 留余地

❌ "这是这章最实操的一条作业"  → 学校腔
✅ 保留, 但口语就是"作业 / 自检 / 拷问自己" 这类词
```

### ✅ 主动用的中文味更强的口语 / 成语

斩钉截铁 / 不在一个数量级 / 翻车 / 上车 / 上头 / 心痒得睡不着 / 铤而走险 / 寒酸 / 没头没尾 / 想冲 / 站上山顶 / 一回事 / 这事 / 这一关 / 闭嘴等着 / 老油条 / 卷不动 / 上桌 / 不上桌 / 留余地 …（这是示例;按你自己常用领域补一份 `<DOMAIN_WORDBANK>`，越贴你母语口癖越好）

### 写完一段的自我检查（必做）

读完一段问自己 3 个问题：

1. 这一段如果是一个中文母语 + 工程师的朋友说给我听, 他会用这些词吗？
2. 有没有 "X 的 Y" 句式可以拆成短句？
3. 有没有外来概念可以找中文对应说法？

任何一条答 "不会 / 有 / 有" → 改。**不要等被提醒**, 这是默认要做的工作。

## 工程

### ❌ Push 不问一声

最早一次 commit 完直接 `git push`，没等批准。规则:所有 push / 对外消息必须先经人类操作者批准。

**修法**：commit 完只列 summary 给用户看，等回"要 / push / yes"再推。<strong>memory 已记录的硬规则，每次都按它做</strong>。

### ❌ 一次 commit 太多无关改动

某次同一 commit 里既改了 ResNet 内容又改了 BERT 引用又加了新 figure。回头 review 困难。

**修法**：一次 commit 一个语义单元：
- 一个 paper ingest = 一个 commit
- 一个 revamp = 一个 commit
- 一组 citation fix = 一个 commit
- log.md / index.md 跟随 paper commit 一起，不单独 commit

### ❌ 在公开 wiki 里写雇主的内部信息 / 业务对照

如果 wiki 是 public repo,"我的批注 / 类比"段落最容易顺手写"跟我们公司那个 XXX 服务一样"这种类比,把雇主的内部信息带出去。最容易漏的几类:

- 公司名 / 产品代号
- 内部服务名、内部 API 名、MQ topic、内部协议名
- 上游供应商、内部项目代号
- 本机绝对路径(`/Users/<你的用户名>/...` 会暴露用户名)

**修法 · 预防**:写"我的批注 / 类比"段时,<strong>只用业界通用术语</strong>:
- ✅ "前台 BFF + 后台 worker + 消息队列"
- ✅ "把外挂的预检 / 计费门收进调用方自身责任"
- ✅ "本地有 clone(软链到 raw/...)"
- ❌ 任何只有你公司内部才懂的服务名 / 类名 / 路径

**修法 · 补救(已发生时)**:
1. 文本去敏(保留分析骨架,只换 identifier 部分)
2. `git checkout --orphan` 重写 history 成单 commit
3. `git push --force-with-lease`
4. 提示残留风险:GitHub dangling ref ~90 天 / fork / clone 副本仍有旧版本

**预防 grep 命令**(commit 前跑,把尖括号换成你自己的敏感词):
```bash
grep -rnE '<公司名>|<内部服务名1>|<内部服务名2>|<内部API名>|/Users/<你的用户名>' \
  wiki/ docs/papers/ index.md log.md 2>/dev/null | \
  grep -vE '\.html:.*-svg'
```
任何输出都是潜在泄露。
