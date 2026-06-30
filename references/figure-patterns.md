# 已验证好用的图设计模式

5 种核心图模式，覆盖 90% 的可视化需求。混搭即可，不要凭空发明。

## 1 · Side-by-side panel · 对比类

两栏，左边 plain/before/A，右边 with-feature/after/B。最常用、最不容易翻车的图。

**用法**：
- BERT Fig 01 · GPT causal 单向 vs BERT 双向（同一句话两种箭头）
- ResNet Fig 03 · 随机 init 下 plain 输出 vs residual 输出
- GPT-1 Fig 02 · causal mask matrix vs bidirectional matrix
- Transformer Fig 04 · BERT 24 enc vs GPT 12 dec vs Transformer 6+6

**实现要点**：
- 左右各一个独立 `.cell` / `.panel` div
- 共享一组数据（同一个 input），看不同处理的输出
- 用 IntersectionObserver 同时触发动画
- 标题色用各自 accent 颜色（plain 用 brick，res 用 moss）

## 2 · Stepper · 多步演化类

按钮控制 step 1 → step 2 → step 3，或 IntersectionObserver 自动序播。

**用法**：
- attention Fig 02 · Self-attention 4 步（Q/K/V → score → softmax → weighted V）
- BERT Fig 02 · MLM 三步（选 15% → 80/10/10 → 双向预测）

**实现要点**：
- 用 CSS class `.s1 .s2 .s3` 切换可见状态
- 每步显示新内容（用 opacity transition 渐入）
- <strong>不要把多步内容叠加在同一画面</strong>（ResNet 原版 Fig 02 的翻车点）
- 自动序播 setInterval 间隔 1300-1700ms 比较合适

## 3 · Bar chart · 数值比较类

横向或纵向条形，展示量的对比。SGV 或 div + height 都可以。

**用法**：
- ResNet Fig 02 · 梯度通过 5 层（plain 0.5^k vs residual 恒为 1）
- ResNet Fig 03 · 随机 init 下 input/output 模式对比
- attention Fig 04 · PE 热力图（20 pos × 64 dim 颜色矩阵）

**实现要点**：
- 真实数据 —— 不要凭印象画
- bar height 动画用 CSS transition (1.0-1.2s ease-in-out)
- stagger delay（每个 bar 延 30-80ms）让眼睛跟得上
- 颜色按"赢家 vs 输家"分（brick 通常表示问题，moss 表示解）

## 4 · Matrix / heatmap · 二维结构类

N×N 网格，每个 cell 颜色或亮度表达一个值。

**用法**：
- GPT-1 Fig 02 · attention causal mask 矩阵（6×6 三角）
- attention Fig 04 · positional encoding sin/cos 热力图

**实现要点**：
- 用 grid-template-columns: repeat(N, 1fr)
- 每 cell aspect-ratio: 1
- 用 JS 循环生成 cell + 决定 active / masked / 颜色强度
- masked cell 用 dashed border 表示，加 ::after { content: '×' }

## 5 · Flow / arch diagram · 结构类

用 SVG 画 block + 箭头连线。

**用法**：
- attention Fig 05 · Encoder/Decoder 6+6 堆叠
- BERT Fig 04 · 三个架构柱状对比 (transformer / bert / gpt)
- BERT Fig 05 · Pretrain → BERT → 4 task heads
- ResNet Fig 02 · 残差 block 内部 (x → F → +x → y)

**实现要点**：
- SVG 用固定 viewBox（e.g. 360 350）然后 100% width responsive
- 用 `<defs><marker id="...">` 定义箭头头，`marker-end="url(#tri)"`
- box 用 `<rect>`，文字用 `<text>` 配 text-anchor="middle" dominant-baseline="middle"
- 不同语义的 box 用不同颜色（input 默认色，F 用 brick-soft，identity 用 moss-soft）
- 箭头连线最好直角或大圆弧，不要锐角折线（容易乱）

## 通用约束

**一张图一个 point**。如果想表达两件事，用 panel 拆开（pattern 1）或 stepper 拆开（pattern 2）。

**所有数值要真实**：
- 算力对比：`590K` `69K` `1/17` 写明确
- 错误率：8.7%, 6.5% 用论文真实数
- 时间复杂度：O(N²) 不写成 "quadratic"
- 不要写"big / small / fast"这种空话

**动画用 IntersectionObserver 自动触发**：
```javascript
const obs = new IntersectionObserver((es) => {
  es.forEach(e => {
    if (e.isIntersecting) { animate(); obs.disconnect(); }
  });
}, { threshold: 0.3 });
obs.observe(fig);
```
threshold 0.25-0.4 都行，看 fig 大小。
