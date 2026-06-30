# llm-wiki-kit

一份**可复用的 Agent Skill**,用来把论文 / 书 / 概念吃进一个"个人学习 wiki"——[Karpathy "LLM Wiki" 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)的实现:**LLM 维护、人类导航**。

核心理念:每篇 ML 论文做成一页**手工打磨的 bespoke 静态页**(独立配色 / 内嵌 SVG 图 / 滚动动画 / 术语浮卡),概念 / 书 / 综述页走自动渲染。讲解**直觉先行、数学换成代码或类比、每个抽象都配能手算的数字例子**。

> 这是从一个长期自用的 wiki 里抽出来的方法论与模板,已脱敏。它**不含**任何具体 wiki 内容,只含"怎么写"的纪律 + 模板 + 翻车教训。

## 里面有什么

```
SKILL.md                              主纪律:文件布局 / ingest 工作流 / 写作硬规则 / 推送流程 / 提交前 checklist
references/
├── paper-md-template.md              paper 笔记 md 模板
├── concept-md-template.md            concept 笔记 md 模板
├── bespoke-html-skeleton.html        bespoke 页骨架(CSS + 结构 + IntersectionObserver 动画模板)
├── figure-patterns.md               5 种验证过好用的内嵌 SVG 图设计模式
├── colors.md                         配色板 + 按领域选 accent 的逻辑
└── anti-patterns.md                  翻车案例与教训(视觉 / 内容 / 中文表达 / 工程 / 隐私)
```

## 怎么用

1. 把整个目录放进你的 agent skills 目录(如 Claude Code 的 `~/.claude/skills/<name>/`)。
2. 全局替换 `SKILL.md` 里的尖括号占位:
   - `<WIKI_ROOT>` —— wiki 本地根目录
   - `<GH_REPO>` / `<PAGES_URL>` —— 部署用的 public repo 与 GitHub Pages 地址
   - `<READER_PROFILE>` —— 读者背景(会什么、抗拒什么),决定讲解风格
   - `<DOMAIN_WORDBANK>` —— 你所在领域 + 母语口癖的口语词库(让中文更 native)
3. 自备一个 `render.py`(扫 md frontmatter 生成索引页 + 自动渲染 concept/topic/book、给 bespoke 页注入导航和术语浮卡、解析 `[[wikilink]]`)。本仓只给方法论,不含 render 实现——它跟你的目录结构强耦合,自己写或让 agent 按 `SKILL.md` 的布局生成。
4. 按 `SKILL.md` 的工作流 ingest。**提交前务必跑 `anti-patterns.md` 里的隐私 grep**:把尖括号换成会暴露你身份的词(雇主内部名、个人信息),公开仓里这是硬红线。

## 设计取向(几条贯穿全程的硬规则)

- **直觉先行**:数学符号尽量换成代码 / 类比;永远给"为什么这么干"(痛点 / 替代 / trade-off),不写空泛定义句。
- **能手算的数字例子**:任何带公式 / 矩阵 / 概率 / 损失的页,正文必须有一段能拿笔跟着算到底的数字例子;多步推导按 §3.6.1 的七条写(先路线图、prose 主导、不跳步、python 核验)。
- **一图一点**:每张图只表达一件事,比较类用左右并排 panel,数值要真;重画成内嵌 SVG 而不是截原图。
- **页面独立成文**:不写"你学过的 X / 刚才聊到"这类预设阅读路径或对话语境的话。
- **隐私红线**:公开仓不带雇主内部信息、不带个人身份信息。

## License

MIT,见 [LICENSE](LICENSE)。
