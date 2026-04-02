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

基于 2-Hour Builder 方法论。

---

## 环境检测

> 技能启动时自动检测运行环境，确定路径变量。

### 第 1 步：检测 2HB 项目结构

在执行任何 Phase 前，用 Bash 检测运行环境：

```bash
if [ -d "core" ] && [ -d "todolist" ] && [ -d "shortcontent" ] && [ -d "newsreflect" ] && [ -d "product-analysis" ]; then
  echo "MODE=integrated"
else
  echo "MODE=standalone"
fi
```

### 第 2 步：设置路径变量

**Integrated 模式**（2HB 项目检测到）：
- `PATH_DIGEST={CWD}/newsreflect/`
- `PATH_ARTICLES={CWD}/shortcontent/`
- `PATH_ANALYSIS={CWD}/product-analysis/`
- `PATH_TODOLIST={CWD}/todolist/`

**Standalone 模式**（无 2HB 结构）：
- `PATH_WORKSPACE={CWD}/content-pipeline-workspace/`
- `PATH_DIGEST={PATH_WORKSPACE}/newsreflect/`
- `PATH_ARTICLES={PATH_WORKSPACE}/shortcontent/`
- `PATH_ANALYSIS={PATH_WORKSPACE}/product-analysis/`
- 自动创建所需目录

### 第 3 步：加载配置

按以下优先级加载：

1. **读取 `config/user-data.md`**（用户覆盖）
   - 如果存在且有非空字段，用用户值覆盖默认值
   - 如果不存在或字段为空，使用内置默认值

2. **读取 `config/x-accounts.md`**（X 账号列表）
   - 如果 `config/user-data.md` 指定了自定义账号，使用那个路径

3. **读取 `config/brand-info.md`**（品牌配置）
   - 用于 Phase 6 HTML 模板变量替换

4. **读取 `templates/methodology-modules.md`**（方法论）
   - 用于 Phase 3 选题的 Module 角度

### 第 4 步：向用户展示检测结果

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Content Pipeline 启动
模式：Integrated / Standalone
X 账号：内置列表 / 自定义列表
品牌：{{BRAND_NAME}} · {{BRAND_AUTHOR}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

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

每个 Phase 完成后向用户展示进度条和关键产出摘要。

---

## Phase 1: X 新闻扫描

目标：获取今日 AI/创业圈动态，为选题提供新闻素材。

### 1.1 读取博主列表

**优先级**：
1. 如果 `config/user-data.md` 指定了自定义账号文件，读取该文件
2. 否则读取 `config/x-accounts.md`

如果两个都不存在，使用以下内置默认列表（来自 config/x-accounts.md）：

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

按以下格式整理（参考 `templates/digest-template.md`）：

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

保存到 `{PATH_DIGEST}YYYY-MM-DD - YYYY-MM-DD/YYYY-MM-DD.md`（按周文件夹组织，周一-周日）。

在 Standalone 模式下，如果目录不存在，先创建。

### 1.5 展示 Phase 1 摘要

向用户展示：
- 检索了多少博主
- 今日重要动态数
- 最有内容价值的 2-3 条新闻（后续选题用）

---

## Phase 2: 产品分析

目标：扫描 Product Hunt / Toolify / Reddit 等，产出今日候选方向池。

### 2.1 读取当前状态

- 检查 `{PATH_ANALYSIS}YYYY-MM/YYYY-MM.md` 是否存在
  - 如果存在，读取了解已分析产品
  - 如果不存在，跳过月度汇总读取（Standalone 模式或首次运行）
- 确认今天日期，避免重复分析

### 2.2 建立 Seed 池

从以下来源建立或更新 seed 池：
- 月度汇总里已出现的高潜力方向（如果存在）
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

（Standalone 模式下：创建 `{PATH_ANALYSIS}YYYY-MM/` 目录结构并写入汇总文件）

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

读取 `templates/methodology-modules.md` 获取完整 Module 素材库。

摘要表格：

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
3. **反转后立刻 = 个人实战证据**：不要放末尾
4. **最后 = 可执行出路**：Input→Build→Ship 或具体行动步骤
5. **信息密度 ≥ 1500 字**，至少 5 个具体数据点

### 4.2 内容形式匹配

| 选题特性 | 推荐形式 | 理由 |
|---------|---------|-----|
| 观点输出、认知冲突 | 公众号长文 | 短内容装不下 |
| 工具清单、操作教程 | 图文（知乎） | 用户需保存反复查看 |
| 深度分析、多信源 | 公众号长文 / 知乎回答 | 短内容装不下 |
| 案例复盘、数据展示 | 图文 + 公众号组合 | 图文放数据，长文讲故事 |
| 热点事件、新闻解读 | 公众号（时效性强） | 快速发布抢时间窗口 |

### 4.3 生成各平台版本

#### 公众号版
- 承担信任沉淀和付费转化角色
- 结构遵循：恐慌→反转→证据→出路
- 文末固定引流：从 `config/brand-info.md` 读取 `{{CTA_TEXT}}`

#### 知乎版（备选）
- 找相关问题回答
- 模板：反差开头 → 正确方法 → 实战数据 → {{ZHIHU_CTA}}

### 4.4 可引用的实战数据

**优先级**：
1. 如果 `config/user-data.md` 存在且有产品数据，使用用户数据
2. 否则使用以下内置默认值（2HB 案例）：

- 5个月4款产品并盈利
- MvpFast: NextJS快速开发模板，50+付费用户，定价59元
- LogoCook: 免费Logo生成工具
- WeFight: 打卡互相监督产品
- IMessageU: 用户反馈收集产品
- 知乎爆款：6.1万阅读、759赞、1883收藏、50+付费转化
- 小红书爆款：1.2万浏览、201赞、182收藏、34评论

**使用格式**：引用时根据用户实际数据替换，保持「X个月X款产品」的成功框架结构。

### 4.5 写入文件

写入 `{PATH_ARTICLES}YYYY-MM-DD - YYYY-MM-DD/YYYY-MM-DD.md`（按周文件夹）。

在 Standalone 模式下，如果目录不存在，先创建。

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

### 6.0 读取品牌配置

读取 `config/brand-info.md` 获取以下变量值：
- `{{BRAND_NAME}}` = 技术人自救宝典（默认）
- `{{BRAND_AUTHOR}}` = kajian（默认）
- `{{CTA_TEXT}}` = 回复「2HB」获取 2-Hour Builder 方法论精华（默认）
- `{{ZHIHU_CTA}}` = 关注公众号「技术人自救宝典」，回复「2HB」获取完整方法论（默认）
- `{{BRAND_FOOTER}}` = 技术人自救宝典 · kajian（默认）

### 6.1 生成公众号排版 HTML

读取 `templates/html/wechat-template.html`，将占位符替换为品牌配置值：

| 占位符 | 替换为 |
|---|---|
| `{{BRAND_NAME}}` | 品牌名称 |
| `{{BRAND_AUTHOR}}` | 作者名 |
| `{{CTA_TEXT}}` | 公众号引流文案 |

将文章内容插入到模板的 `<!-- 文章内容 -->` 位置。

输出文件名：`{PATH_ARTICLES}YYYY-MM-DD - YYYY-MM-DD/YYYY-MM-DD-wechat.html`

### 6.2 生成封面 HTML

读取 `templates/html/cover-template.html`，将占位符替换为品牌配置值：

| 占位符 | 替换为 |
|---|---|
| `{{BRAND_FOOTER}}` | 底部署名 |

输出文件名：`{PATH_ARTICLES}YYYY-MM-DD - YYYY-MM-DD/YYYY-MM-DD-cover.html`

### 6.3 截图生成 cover.png

使用 agent-browser 或 Playwright 打开 cover.html → 截图 → 保存为 `YYYY-MM-DD-cover.png`（900x383）。

将封面图插入到 wechat.html 的 `<body>` 后第一个元素位置。

### 6.4 浏览器预览

```bash
open {PATH_ARTICLES}YYYY-MM-DD - YYYY-MM-DD/YYYY-MM-DD-wechat.html
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

从 `config/brand-info.md` 读取平台引流文案：
- **公众号**：文末 `{{CTA_TEXT}}`
- **知乎**：回答末尾引导 `{{ZHIHU_CTA}}`
- **小红书**：文末 `{{XIAOHONGSHU_CTA}}`

---

## 跳过某个 Phase

用户可以在触发时指定跳过：
- `/content-pipeline --skip-news` — 跳过 Phase 1，直接用已有 newsreflect
- `/content-pipeline --skip-analysis` — 跳过 Phase 2，直接选题
- `/content-pipeline --topic "指定选题"` — 跳过 Phase 1-3，直接写文章
- `/content-pipeline --from-phase 4` — 从指定 Phase 开始（假设前面已完成）

---

## 自定义指南

Skill 所有可替换内容都集中在 `config/` 和 `templates/` 目录。

### 配置替换

| 配置项 | 文件路径 | 说明 |
|---|---|---|
| X 账号列表 | `config/x-accounts.md` | 替换博主列表 |
| 个人产品/成果 | `config/user-data.md` | 替换 Phase 4.4 实战数据 |
| 品牌信息 | `config/brand-info.md` | 替换品牌名、引流文案 |
| 方法论 Module | `templates/methodology-modules.md` | 替换选题框架 |
| 文章模板 | `templates/article-template.md` | 替换内容结构说明 |
| 摘要模板 | `templates/digest-template.md` | 替换 Digest 输出格式 |

### HTML 模板替换

| 模板 | 文件路径 | 说明 |
|---|---|---|
| 公众号 HTML | `templates/html/wechat-template.html` | 替换 CSS 样式和排版 |
| 封面 HTML | `templates/html/cover-template.html` | 替换封面设计 |

### 安装到其他项目

1. 复制整个 `content-pipeline/` 目录到目标项目的 `.agents/skills/`
2. 在目标项目的 `.claude/skills/` 创建 symlink
3. 填写 `config/user-data.md` 个性化数据
4. 可选：调整 `config/x-accounts.md` 和 `templates/` 内容
