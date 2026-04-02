---
name: content-pipeline
description: |
  端到端内容生产流水线：X新闻扫描 → 产品分析 → 选题 → 写公众号长文 → 五维诊断修正 → HTML排版。
  基于 2-Hour Builder 方法论，将 6 个独立步骤串成一键流水线。
  触发方式：/content-pipeline、「今天的流水线」「跑一遍内容」「pipeline」「一键生成」
  End-to-end content production pipeline: X news scan → product analysis → topic selection → WeChat article → 5-dimension diagnosis → HTML layout.
  Trigger: /content-pipeline, "run pipeline", "one-click content"
argument-hint: "[可选：指定主题方向或跳过某个Phase]"
allowed-tools: [Read, Write, Edit, Glob, Bash, WebSearch, WebFetch, Agent]
user-invocable: true
disable-model-invocation: false
---

# Content Pipeline — 端到端内容生产流水线

将「X 新闻扫描 → 产品分析 → 选题 → 写公众号长文 → 五维诊断修正 → HTML 排版」六步串成一键流水线。

基于 2-Hour Builder 方法论（5 个月 4 款产品盈利的实战体系）。

---

## 流水线总览

```
Phase 1: X 新闻扫描        → 今日 AI/创业动态
Phase 2: 产品分析           → 今日候选方向 Top 10-20
Phase 3: 选题决策           → 综合新闻+产品+方法论，确认 1 个选题
Phase 4: 生成公众号长文      → 恐慌→反转→证据→出路
Phase 5: 五维诊断+修正      → 文字洁癖/标题/效率/落差/产品关联
Phase 6: HTML 排版+预览     → wechat.html + cover.html → 浏览器打开
```

每个 Phase 完成后向用户展示进度条和关键产出摘要：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Phase X/6 完成 ████████░░░░
[Phase 名称]
关键产出：...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 1: X 新闻扫描

目标：获取今日 AI/创业圈动态，为选题提供新闻素材。

### 1.1 读取博主列表

读取 `tools/x-digest/x-accounts.md` 获取关注的 X 博主 handle。

如果文件不存在，使用默认列表：

**必查（每次都查）：**
- @karpathy — AI 技术动态
- @swyx — AI 工程社区
- @gregisenberg — 创业想法
- @kevinweil — OpenAI 产品动向
- @rauchg — Vercel/前端生态
- @amasad — Replit/AI coding
- @alexalbert__ — Claude/Anthropic 动态
- @garrytan — YC/创业趋势
- @venturetwins — 消费 AI 趋势

**选查（根据热点选 3-5 个）：**
- 其余博主根据当前 AI 热点选择性检索

### 1.2 批量搜索

对每个博主使用 WebSearch：
```
from:@handle since:YYYY-MM-DD until:YYYY-MM-DD site:x.com
```
或：
```
"@handle" AI [关键词] site:x.com [日期]
```

### 1.3 整理 Digest

按以下格式整理：

```markdown
# X Daily Digest — YYYY-MM-DD

## 重要动态（影响大/讨论多）
- **@handle**：[推文摘要]（[原文链接]）
  → 分析：这意味着什么

## 产品更新
- ...

## 值得关注的观点
- ...

## 无重要更新的博主
@handle1, @handle2, ...

## 对内容创作的启示
- 是否有内容素材可用
- 是否有工具/趋势值得跟进
```

### 1.4 保存记录

保存到 `newsreflect/YYYY-MM-DD - YYYY-MM-DD/YYYY-MM-DD.md`（按周文件夹组织，周一-周日）。

### 1.5 展示 Phase 1 摘要

向用户展示：
- 检索了多少博主
- 今日重要动态数
- 最有内容价值的 2-3 条新闻（后续选题用）

---

## Phase 2: 产品分析

目标：扫描 Product Hunt / Toolify / Reddit 等，产出今日候选方向池。

### 2.1 读取当前状态

- 读取月度汇总文件（`product-analysis/YYYY-MM/YYYY-MM.md`），了解已分析产品
- 确认今天日期，避免重复分析

### 2.2 建立 Seed 池

从以下来源建立或更新 seed 池：
- 月度汇总里已出现的高潜力方向
- 最近 3-7 天反复出现的话题词
- Phase 1 新闻中出现的新产品/新工具/新趋势
- 用户手工提供的主题

seed 写成「可扩展的需求词」：
- 产品词：`AI sales copilot`
- 痛点词：`reply to reddit comments automatically`
- 人群词：`solopreneur seo reporting`
- 替代词：`alternative to [product]`

### 2.3 系统初筛 Pipeline

按顺序执行：

1. **种子词 → 扩词**
   - Google Autocomplete、相关搜索、站内搜索建议
   - 合并同义词、单复数、近义表达

2. **社区验证**
   - 搜 Product Hunt、Reddit、X、Indie Hackers、Hacker News
   - 记录是否有人在抱怨、求替代、晒方案、问"有没有工具"
   - 没有真实用户表达 → 降权

3. **SERP 竞争分析**
   - 搜索结果是否被大站垄断
   - 自然结果是否已有成熟 SaaS
   - 结果页偏"信息词"还是"购买词"

4. **商业信号验证**
   - 是否出现 SaaS 定价页 / App Store / Chrome Extension
   - 是否有广告投放或商业化 landing page
   - 是否有明确付费产品在卖这个需求

5. **综合评分**
   - 输出排序，候选方向池 Top 10-20

### 2.4 评分输出

每个方向输出：
- **方向/关键词**
- **一句话定位**：用「用XX让**XX人**不再XX」格式
- **需求信号**：搜索/社区/趋势是否成立
- **商业信号**：CPC、定价页、App Store、SaaS 信号
- **竞争强度**：低/中/高
- **空白度**：1-5 星
- **Clone Score**（0-10）：能不能直接做竞品
- **判断**：值得复核 / 观望 / 不做

**Clone Score 5 维度**（每项 0-2 分）：

| 维度 | 0 分 | 1 分 | 2 分 |
|------|------|------|------|
| 已验证 | 无付费用户 | 有用户但规模不清 | 有明确付费数据 |
| 团队规模 | VC/大团队 | 小团队（5-10人） | 独立开发者（1-3人） |
| 技术壁垒 | 自研模型/专利 | 需领域知识 | API 组合/应用层 |
| 市场可分 | 赢家通吃 | 中等集中 | 长尾可切细分 |
| 海外可做 | 仅限本地 | 部分可进 | 全球化产品 |

### 2.5 更新月度汇总

在月度汇总文件中追加当日候选方向表格、关键发现、需求方向。

### 2.6 展示 Phase 2 摘要

向用户展示：
- 今日候选方向数
- Top 3 高分方向（一句话 + Clone Score）
- 与 Phase 1 新闻的交叉发现

---

## Phase 3: 选题决策

目标：综合 Phase 1 新闻 + Phase 2 产品 + 方法论素材库，确认 1 个选题。

### 3.1 素材来源矩阵

| 来源 | 用法 |
|------|------|
| Phase 1 新闻 | 热点钩子——用新闻事件引入话题，增加时效性 |
| Phase 2 产品 | 实战案例——用真实产品做论据，增加可信度 |
| 6 个 Module 素材库 | 方法论内核——内容的核心观点来源 |
| 用户实战数据 | 个人证据——5个月4款产品、知乎6.1万阅读等 |

### 3.2 六大 Module 素材库

| Module | 可切入角度 | 适合类型 |
|---|---|---|
| M1 世界观 | 工作者vs生产者、发布即测试 | 金句型、对比型 |
| M2 2小时系统 | Input→Build→Ship、最小闭环 | 清单型、方法型 |
| M3 反Demo地狱 | 可售性>完整性、3天验证法 | 诊断型、对比型 |
| M4 内耗系统 | 三类内耗识别、三步止损 | 诊断型、金句型 |
| M5 现金流 | 先收钱再做产品、30天路径 | 方法型、故事型 |
| M6 Life OS | 系统战胜意志力 | 金句型、清单型 |

### 3.3 生成 3-5 个候选选题

每个候选选题必须包含：
- **标题**（必须通过标题自检，见 3.5）
- **新闻钩子**：用哪条新闻/产品引入
- **方法论内核**：对应哪个 Module
- **目标平台**：公众号 / 知乎 / 小红书
- **预估素材丰富度**（见 3.4）

### 3.4 素材丰富度预检

确定选题前，检查素材够不够：

| 素材维度 | 检查 | 示例 |
|---------|------|------|
| **冲击数据** | 有没有大数字/对比/百分比？ | 140 万亿、增长 1000 倍 |
| **转变故事** | 有没有之前 vs 之后的反差？ | 42 浏览 → 1.2 万浏览 |
| **金句** | 有没有能独立传播的观点？ | "差的不是技术，是把水卖给渴的人" |
| **权威背书** | 有没有人物/机构/数据来源？ | 国家数据局、Karpathy |
| **痛点共鸣** | 有没有目标人群的焦虑？ | "你还在准备中" |

**判定**：
- 3+ 维度 → 可以生成，内容会有冲击力
- 1-2 个 → 可以生成但效果有限，建议补充
- 0 个 → **停止**，先补充素材。「素材不够，硬写出来也没人看」

### 3.5 标题自检（必须全部通过）

#### 基础三类（至少命中一类）
- [ ] **赚钱/结果承诺**：包含具体数字、收入、产品数量
- [ ] **省时间的工具**：承诺清单、模板、可直接用的东西
- [ ] **反差/冲突/反常识**：制造认知冲突

**核心公式：标题卖结果/工具/冲突，内容藏方法论。**

数据验证：
- "赚取100万" = 1.2万浏览
- "5个月4款产品盈利" = 851浏览
- "卖不出第一单" = 42浏览（方法论标题，失败）

#### 升级检查
- [ ] **悬念**：标题保留悬念，答案不在标题里
- [ ] **数字冲击**：优先用大数字（6000万 > 342）
- [ ] **3 秒痛感**：开头前 3 句直接给最刺激数据

#### 开头公式
```
开头 = 话题 + Hook + 可信度

示例：
  话题：国家免费发 3000 万 Token
  Hook：但 99% 的人不知道怎么花
  可信度：过去 5 个月我做了 4 款产品盈利
```

### 3.6 用户确认

将 3-5 个候选选题展示给用户，让用户选择或调整。**必须等用户确认后再进入 Phase 4。**

---

## Phase 4: 生成公众号长文

目标：基于确认的选题，生成完整的公众号文章。

### 4.1 内容结构公式

**恐慌（前1/3）→ 反转（中间）→ 证据（实战）→ 出路（方法）**

1. **前 1/3 = 恐慌**：堆数据、堆岗位、堆信源，制造焦虑。至少 2-3 个信源
2. **中间 = 反转**：但有一种人不怕 / 这里面藏着机会
3. **反转后立刻 = 个人实战证据**：5 个月 4 款产品。不要放末尾
4. **最后 = 可执行出路**：Input→Build→Ship 或具体行动步骤
5. **信息密度 ≥ 1500 字**，至少 5 个具体数据点

### 4.2 内容形式匹配

| 选题特性 | 推荐形式 | 理由 |
|---------|---------|------|
| 观点输出、认知冲突 | 公众号长文 | 短内容装不下 |
| 工具清单、操作教程 | 图文（知乎） | 用户需保存反复查看 |
| 深度分析、多信源 | 公众号长文 / 知乎回答 | 短内容装不下 |
| 案例复盘、数据展示 | 图文 + 公众号组合 | 图文放数据，长文讲故事 |
| 热点事件、新闻解读 | 公众号（时效性强） | 快速发布抢时间窗口 |

### 4.3 生成各平台版本

#### 公众号版
- 承担信任沉淀和付费转化角色
- 结构遵循：恐慌→反转→证据→出路
- 文末固定引流：关注「技术人自救宝典」，回复「2HB」

#### 知乎版（备选）
- 找相关问题回答
- 模板：反差开头 → 正确方法 → 实战数据 → 引导公众号

### 4.4 可引用的实战数据

- 5个月4款产品并盈利
- MvpFast: NextJS快速开发模板，50+付费用户，定价59元
- LogoCook: 免费Logo生成工具
- WeFight: 打卡互相监督产品
- IMessageU: 用户反馈收集产品
- 知乎爆款：6.1万阅读、759赞、1883收藏、50+付费转化
- 小红书爆款：1.2万浏览、201赞、182收藏、34评论

### 4.5 写入文件

写入 `shortcontent/YYYY-MM-DD - YYYY-MM-DD/YYYY-MM-DD.md`（按周文件夹）。

如果项目中有 `builder` CLI，使用 `builder content template --date YYYY-MM-DD` 创建模板文件。

---

## Phase 5: 五维诊断 + 修正

目标：对生成的文章做质量诊断，根据结果自动修正。

### 5.1 检测 dbs-content skill

检查 `.claude/skills/dbs-content/SKILL.md` 是否存在：
- **存在** → 调用 `/dbs-content` 做完整诊断（包含案例库、说话风格、下一步建议）
- **不存在** → 使用下面的内联精简版诊断

### 5.2 内联五维诊断

对文章做以下 5 个维度诊断：

#### 维度 1：文字洁癖检测
- 有没有 AI 味？（Emoji 堆叠、晦涩词汇、空洞排比句）
- 清除以下 AI 味特征：
  - 「请你记住」「真相是」「值得注意的是」= 无能创作者的祈使句
  - Emoji 超过 3 个 = 过度装饰
  - 「首先…其次…最后」= 模板化结构
  - 「让我们一起来看看」= 废话
- 判断：✅ 干净 / ⚠️ 有 AI 味需清洗 / ❌ 重写

#### 维度 2：封面/标题诊断
- 平铺直叙能不能吸引人？
- 标题的情绪是信息传递还是认知劫持？
- 判断：✅ 自带吸引力 / ⚠️ 需优化 / ❌ 重做

#### 维度 3：表达效率检测
- 能不能一句话说清核心观点？
- 有没有 99% 时间包装 1% 的内容？
- 判断：✅ 高效 / ⚠️ 有冗余 / ❌ 本末倒置

#### 维度 4：认知落差检测
- 同行讲清楚了吗？你比同行好在哪？
- 读者会不会觉得「我知道了」？
- 判断：✅ 有明显落差 / ⚠️ 较小 / ❌ 无落差

#### 维度 5：产品关联检测
- 内容是否服务于产品变现？还是在自嗨？
- 判断：✅ 有转化路径 / ⚠️ 弱关联 / ❌ 纯自嗨

### 5.3 诊断结果展示

```
## 五维诊断报告
| 维度 | 判断 | 说明 |
|------|------|------|
| 文字洁癖 | ✅/⚠️/❌ | {具体问题} |
| 封面/标题 | ✅/⚠️/❌ | {具体问题} |
| 表达效率 | ✅/⚠️/❌ | {具体问题} |
| 认知落差 | ✅/⚠️/❌ | {具体问题} |
| 产品关联 | ✅/⚠️/❌ | {具体问题} |
```

### 5.4 自动修正

**任何维度出现 ❌ → 停止发布，先修复。**

根据诊断结果自动修改：
- 标题 ⚠️ → 重写标题（加冲突/悬念）
- 表达效率 ⚠️ → 调整结构（核心论点前移、压缩重复）
- 文字洁癖 ⚠️ → 清除 AI 味句式
- 认知落差 ⚠️ → 补充差异化观点或数据
- 产品关联 ⚠️ → 加强转化路径

修改后更新 shortcontent 文件，确保后续 Phase 基于终稿。

---

## Phase 6: HTML 排版 + 预览

目标：将终稿生成排版好的公众号 HTML，可直接复制粘贴到公众号编辑器。

### 6.1 生成公众号排版 HTML

生成 `YYYY-MM-DD-wechat.html`，使用以下 CSS 样式体系：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
  body {
    font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
    max-width: 580px;
    margin: 0 auto;
    padding: 24px 16px;
    font-size: 15px;
    line-height: 2;
    color: #374151;
    letter-spacing: 0.4px;
    background: #ffffff;
  }
  p { margin: 1em 0; text-align: justify; }
  strong { color: #111827; }
  ul { padding-left: 1.2em; margin: 1em 0; }
  li { margin: 0.65em 0; }
  .divider {
    text-align: center;
    margin: 2em 0;
    color: #d1d5db;
    font-size: 14px;
    letter-spacing: 8px;
  }
  .big-num {
    font-size: 32px;
    font-weight: 800;
    color: #ea580c;
    letter-spacing: 1px;
  }
  .hl {
    background: linear-gradient(to top, #ffedd5 52%, transparent 52%);
    padding: 0 3px;
    font-weight: 700;
    color: #111827;
  }
  .hl-cyan {
    background: linear-gradient(to top, #cffafe 52%, transparent 52%);
    padding: 0 3px;
    font-weight: 700;
    color: #111827;
  }
  .hl-red {
    background: linear-gradient(to top, #fee2e2 52%, transparent 52%);
    padding: 0 3px;
    font-weight: 700;
    color: #991b1b;
  }
  .section-title {
    font-size: 17px;
    font-weight: 700;
    color: #111827;
    margin: 2em 0 0.8em;
    padding-left: 12px;
    border-left: 4px solid #f97316;
  }
  .data-card {
    background: linear-gradient(135deg, #fff7ed 0%, #fffbeb 100%);
    border: 1px solid #fed7aa;
    border-radius: 10px;
    padding: 18px 22px;
    margin: 1.4em 0;
  }
  .data-card p { margin: 0.45em 0; }
  .quote-box {
    background: #f8fafc;
    border: 1px solid #e5e7eb;
    border-radius: 10px;
    padding: 18px 22px;
    margin: 1.4em 0;
  }
  .quote-box p { margin: 0.4em 0; }
  .compare-box {
    background: linear-gradient(135deg, #f8fafc 0%, #eff6ff 100%);
    border: 1px solid #dbeafe;
    border-radius: 10px;
    padding: 18px 22px;
    margin: 1.4em 0;
  }
  .compare-box p { margin: 0.45em 0; }
  .pit-box {
    background: #ecfeff;
    border: 1px solid #a5f3fc;
    border-radius: 10px;
    padding: 18px 22px;
    margin: 1.4em 0;
  }
  .pit-label {
    display: inline-block;
    margin-bottom: 8px;
    padding: 2px 8px;
    border-radius: 999px;
    background: #0891b2;
    color: #ffffff;
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.8px;
  }
  .danger-box {
    background: linear-gradient(135deg, #fef2f2 0%, #fff1f2 100%);
    border: 1px solid #fecaca;
    border-radius: 10px;
    padding: 18px 22px;
    margin: 1.4em 0;
  }
  .danger-label {
    display: inline-block;
    margin-bottom: 8px;
    padding: 2px 8px;
    border-radius: 999px;
    background: #dc2626;
    color: #ffffff;
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.8px;
  }
  blockquote {
    border-left: 4px solid #f97316;
    margin: 1.5em 0;
    padding: 12px 18px;
    background: #fff7ed;
    border-radius: 0 10px 10px 0;
    color: #9a3412;
  }
  blockquote p { margin: 0.35em 0; }
  .golden {
    text-align: center;
    font-size: 16px;
    font-weight: 700;
    color: #9a3412;
    margin: 1.8em 0;
    line-height: 2.1;
  }
  .author-card {
    background: linear-gradient(135deg, #fff7ed 0%, #ffedd5 100%);
    border-radius: 12px;
    padding: 20px 24px;
    margin: 2em 0 0;
    text-align: center;
    font-size: 14px;
    line-height: 2;
    color: #9a3412;
  }
  .author-card strong { color: #7c2d12; }
  .checklist {
    background: #f0fdf4;
    border: 1px solid #bbf7d0;
    border-radius: 10px;
    padding: 18px 22px;
    margin: 1.4em 0;
  }
  .checklist li { list-style: none; padding-left: 0; }
  .checklist li::before { content: "✅ "; }
</style>
</head>
<body>
  <!-- 文章内容 -->
</body>
</html>
```

**排版规则**：
- 关键数据用 `.big-num` / `.hl` / `.hl-cyan` / `.hl-red` 高亮
- 小节标题用 `.section-title`（左橙色边框）
- 数据展示用 `.data-card`（渐变背景卡片）
- 引用/金句用 `<blockquote>`（橙色左边框）
- 对比内容用 `.compare-box`
- 踩坑/警示用 `.pit-box` + `.pit-label` 或 `.danger-box` + `.danger-label`
- 行动清单用 `.checklist`
- 段落间用 `.divider`（`· · ·`）分隔
- 文末用 `.author-card`（作者卡片 + 引流）

### 6.2 生成封面 HTML

生成 `YYYY-MM-DD-cover.html`（900x383px），深色科技风：

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 900px; height: 383px;
    display: flex; align-items: center; justify-content: center;
    background:
      radial-gradient(circle at 20% 18%, rgba(249,115,22,0.18), transparent 34%),
      radial-gradient(circle at 78% 82%, rgba(34,211,238,0.14), transparent 30%),
      linear-gradient(135deg, #0b1220 0%, #111827 45%, #060b16 100%);
    font-family: -apple-system, "PingFang SC", "Microsoft YaHei", sans-serif;
    overflow: hidden; position: relative;
  }
  .grid {
    position: absolute; inset: 0;
    background-image:
      linear-gradient(rgba(148,163,184,0.06) 1px, transparent 1px),
      linear-gradient(90deg, rgba(148,163,184,0.06) 1px, transparent 1px);
    background-size: 44px 44px;
  }
  .glow-line {
    position: absolute; left: 42px; top: 92px;
    width: 4px; height: 196px; border-radius: 999px;
    background: linear-gradient(to bottom, rgba(249,115,22,0), rgba(249,115,22,0.95), rgba(220,38,38,0.75), rgba(220,38,38,0));
    box-shadow: 0 0 24px rgba(220,38,38,0.2);
  }
  .bg-word {
    position: absolute; right: -10px; top: 50%; transform: translateY(-50%);
    font-size: 140px; font-weight: 900; line-height: 1; letter-spacing: -8px;
    color: rgba(220,38,38,0.06); user-select: none;
  }
  .content {
    position: relative; z-index: 2;
    max-width: 780px; text-align: center; padding: 0 50px;
  }
  .tags { display: flex; justify-content: center; gap: 10px; margin-bottom: 18px; }
  .tag {
    padding: 4px 14px; border-radius: 999px;
    font-size: 11px; font-weight: 700; letter-spacing: 1.4px;
  }
  .tag-red { color: #fca5a5; border: 1px solid rgba(220,38,38,0.35); background: rgba(220,38,38,0.1); }
  .tag-orange { color: #fdba74; border: 1px solid rgba(249,115,22,0.35); background: rgba(249,115,22,0.1); }
  .headline {
    font-size: 38px; font-weight: 900; line-height: 1.28;
    letter-spacing: -1.2px; color: rgba(255,255,255,0.96);
  }
  .headline .accent {
    background: linear-gradient(135deg, #fb923c 0%, #fdba74 45%, #fde68a 100%);
    -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  }
  .headline .accent-red {
    background: linear-gradient(135deg, #ef4444 0%, #fca5a5 100%);
    -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  }
  .subline {
    margin-top: 16px; font-size: 18px; line-height: 1.6;
    letter-spacing: 1px; color: rgba(255,255,255,0.58);
  }
  .subline .keyword { color: #f8fafc; font-weight: 700; }
  .metrics { display: flex; justify-content: center; gap: 16px; margin-top: 18px; }
  .metric {
    min-width: 126px; padding: 10px 14px; border-radius: 14px;
    background: rgba(15,23,42,0.48); border: 1px solid rgba(148,163,184,0.16);
    backdrop-filter: blur(4px);
  }
  .metric .num { display: block; font-size: 24px; font-weight: 900; color: #f8fafc; }
  .metric .label {
    display: block; margin-top: 2px; font-size: 11px;
    letter-spacing: 1.2px; color: rgba(255,255,255,0.45);
  }
  .source {
    position: absolute; right: 28px; bottom: 18px;
    font-size: 12px; letter-spacing: 1px; color: rgba(255,255,255,0.24);
  }
</style>
</head>
<body>
<div class="grid"></div>
<div class="glow-line"></div>
<div class="bg-word"><!-- 月份英文 --></div>

<div class="content">
  <div class="tags">
    <span class="tag tag-red"><!-- 标签1 --></span>
    <span class="tag tag-orange"><!-- 标签2 --></span>
  </div>
  <div class="headline">
    <!-- 主标题，用 accent/accent-red 标记关键词 -->
  </div>
  <div class="subline">
    <!-- 副标题 -->
  </div>
  <div class="metrics">
    <!-- 2-3 个数据指标 -->
  </div>
</div>

<div class="source">技术人自救宝典 · kajian</div>
</body>
</html>
```

### 6.3 截图生成 cover.png

使用 agent-browser 或 Playwright 打开 cover.html → 截图 → 保存为 `YYYY-MM-DD-cover.png`（900x383）。

将封面图插入到 wechat.html 的 `<body>` 后第一个元素位置。

### 6.4 浏览器预览

```bash
open shortcontent/周文件夹/YYYY-MM-DD-wechat.html
```

向用户确认：「浏览器已打开，Cmd+A → Cmd+C → 公众号编辑器粘贴。」

---

## 特别警告（遇到就直说）

- 用户说「帮我想个标题」→ **「标题是内容的试用装。你的内容值不值得看，决定了标题好不好写。先看内容。」**
- 用户说「我想做干货内容」→ **「所有让你讲干货的建议都是不专业的。你要做的是把事情搞清楚，然后说清楚。」**
- 用户说「AI 写的内容被限流了」→ **「不是 AI 的问题，是你对文字没有洁癖。」**
- 用户没有产品就想做内容 → **「先有产品后有内容。你的付款链接在哪？」**
- 素材不够就想开始写 → **「素材不够，硬写出来也没人看。先补充素材再来。」**

---

## 内联案例库

### 正面案例

**案例 1：结果标题 + 恐慌结构 = 1.2 万浏览**
> 标题："赚取 100 万"类结果标题 + 文章内多信源数据堆叠 + 反转给出路
- 要点：标题卖结果，内容藏方法论。恐慌→反转→证据→出路公式生效

**案例 2：修改封面标题，播放量涨 3 倍**
> 内容不变，只改封面和标题：播放量涨 3 倍、互动涨 3 倍、评论涨 4 倍。封面点击率 5% → 19%
- 要点：内容不变，封面标题决定生死

**案例 3：140 万亿词元文章 — 热点 + 方法论融合**
> 国家数据局发布数据 + 3 组大数字冲击 + 成本归零分析 + 2HB 行动步骤
- 要点：热点事件做钩子，方法论做内核。信息密度 1800 字，5+ 数据点

### 反面案例

**反面 1：方法论标题 = 42 浏览**
> 标题"卖不出第一单"——无结果承诺，无数字冲击。同期结果标题 851 浏览，差 20 倍

**反面 2：800 字单信源 vs 3000 字多信源**
> 同主题文章，多信源 3000 字爆了，单信源 800 字没爆。信息密度是硬指标

---

## 引流规则

- 公众号：文末"回复「2HB」获取xxx"
- 知乎：回答末尾引导关注公众号
- 小红书：文末"看我主页简介"引导

---

## 跳过某个 Phase

用户可以在触发时指定跳过：
- `/content-pipeline --skip-news` — 跳过 Phase 1，直接用已有 newsreflect
- `/content-pipeline --skip-analysis` — 跳过 Phase 2，直接选题
- `/content-pipeline --topic "指定选题"` — 跳过 Phase 1-3，直接写文章
- `/content-pipeline --from-phase 4` — 从指定 Phase 开始（假设前面已完成）

---

## 自定义指南

如果你 fork 这个 skill 用于自己的内容体系：

1. **替换博主列表**：修改 Phase 1.1 的博主 handle
2. **替换方法论**：修改 Phase 3.2 的 Module 素材库，换成你的知识体系
3. **替换实战数据**：修改 Phase 4.4 的可引用数据
4. **替换品牌信息**：修改 HTML 模板中的「技术人自救宝典 · kajian」
5. **替换引流规则**：修改末尾的引流文案
6. **替换平台策略**：如果不发公众号，修改 Phase 4 和 Phase 6 的平台逻辑
