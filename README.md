# Content Pipeline

> X 新闻扫描 → 产品分析 → 选题 → 写公众号长文 → 五维诊断修正 → HTML 排版，一键完成。

A Claude Code skill that chains 6 content production steps into one automated pipeline.

## Features

- **Two modes**: Integrated (2HB project) or Standalone (any project)
- **Bundled defaults**: X accounts, methodology, HTML templates all included — no setup required
- **Fully customizable**: Override defaults via `config/user-data.md` and `config/brand-info.md`
- **Zero dependencies**: No CLI tools required

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- WebSearch access (for X news and product analysis)
- Browser tool (optional, for cover screenshots)

## Installation

### Option 1: Symlink (Recommended)

```bash
# Clone or copy to .agents/skills (git-tracked source of truth)
# cp -r /path/to/content-pipeline ~/.agents/skills/content-pipeline

# Create symlink in .claude/skills
ln -s ~/.agents/skills/content-pipeline ~/.claude/skills/content-pipeline
```

### Option 2: Standalone (For non-2HB projects)

```bash
# Clone anywhere
git clone https://github.com/kajian/content-pipeline.git ~/content-pipeline-skill

# Create symlink
ln -s ~/content-pipeline-skill ~/.claude/skills/content-pipeline
```

The skill will create a `content-pipeline-workspace/` directory for all outputs.

### Option 3: Manual Copy

```bash
# Copy entire content-pipeline/ directory to your Claude Code skills directory
cp -r content-pipeline/ ~/.claude/skills/content-pipeline
```

## Quick Start

In Claude Code:

```
/content-pipeline
```

Or use natural language:
```
今天的流水线
跑一遍内容
pipeline
```

The skill auto-detects whether you're in a 2HB project:

| Mode | Detected by | Output directory |
|------|------------|-----------------|
| **Integrated** | 2HB project structure (core/, todolist/, shortcontent/, newsreflect/, product-analysis/) | Uses existing 2HB directories |
| **Standalone** | No 2HB structure found | Creates `content-pipeline-workspace/` |

---

## Configuration (Optional)

All configuration is optional. The skill works out-of-the-box with bundled defaults.

### User Data (`config/user-data.md`)

Fill in your own products and stats to personalize the content:

```markdown
## 个人产品/服务数据
- **产品 1**: My Product - 一句话定位 - $29/月 - 100+付费用户

## 个人成果数据
- 爆款案例：知乎 1万阅读、500赞

## 品牌信息覆盖
- **公众号名称**: 我的公众号
- **作者名**: myname
```

### X Accounts (`config/x-accounts.md`)

Replace with your own X/Twitter account list to follow.

### Brand Info (`config/brand-info.md`)

Customize the brand name, CTA text, and colors used in HTML templates.

---

## Output Files

**Integrated mode** (2HB project):
- Digests: `newsreflect/YYYY-MM-DD - YYYY-MM-DD/`
- Articles: `shortcontent/YYYY-MM-DD - YYYY-MM-DD/`
- Analysis: `product-analysis/YYYY-MM/`

**Standalone mode** (non-2HB project):
- All files go to `content-pipeline-workspace/`

---

## Phase Skip Options

```
/content-pipeline --skip-news          # Skip X news, use existing digest
/content-pipeline --skip-analysis      # Skip product analysis
/content-pipeline --topic "AI替代论"    # Skip to writing with given topic
/content-pipeline --from-phase 4       # Start from Phase 4
```

---

## Customization Reference

| Section | File | What to change |
|---------|------|----------------|
| X accounts | `config/x-accounts.md` | 博主列表 |
| Personal data | `config/user-data.md` | 产品、成果、品牌 |
| Brand info | `config/brand-info.md` | 公众号名、引流文案 |
| Methodology | `templates/methodology-modules.md` | 6 Module 框架 |
| Article format | `templates/article-template.md` | 内容结构说明 |
| Digest format | `templates/digest-template.md` | Digest 输出格式 |
| WeChat HTML | `templates/html/wechat-template.html` | 排版样式 |
| Cover HTML | `templates/html/cover-template.html` | 封面设计 |

---

## Content Philosophy

- **Title sells results/tools/conflict, content hides methodology**
- **Structure: Panic (1/3) → Reversal → Evidence → Action**
- **Material richness check** before writing (5 dimensions, need 3+)
- **Zero AI taste** — auto-detects and removes AI writing patterns
- **Product-first** — content serves monetization, not vanity

---

## File Structure

```
content-pipeline/
├── SKILL.md                          # Main skill definition
├── README.md                         # This file
├── config/
│   ├── x-accounts.md                # X account list (bundled default)
│   ├── user-data.md                 # User personalization (override)
│   └── brand-info.md                # Brand configuration
├── templates/
│   ├── methodology-modules.md       # 6 Module reference
│   ├── article-template.md          # Article structure
│   ├── digest-template.md           # Digest output format
│   └── html/
│       ├── wechat-template.html     # WeChat article HTML
│       └── cover-template.html      # Cover image HTML
└── examples/
    ├── sample-digest.md             # Phase 1 output example
    ├── sample-article.md            # Phase 4 output example
    └── sample-wechat.html           # Phase 6 output example
```

---

## License

MIT
