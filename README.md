# Content Pipeline

> X 新闻扫描 → 产品分析 → 选题 → 写公众号长文 → 五维诊断修正 → HTML 排版，一键完成。

A Claude Code skill that chains 6 content production steps into one automated pipeline. Built on the [2-Hour Builder](https://github.com/kajian) methodology (5 products shipped & profitable in 5 months).

## What it does

```
Phase 1: X News Scan        → Today's AI/startup news from 9+ accounts
Phase 2: Product Analysis   → Top 10-20 product opportunities (PH/Reddit/X/SERP)
Phase 3: Topic Selection    → Combine news + products + methodology → pick 1 topic
Phase 4: Write Article      → WeChat long-form article (1500+ words, multi-source)
Phase 5: 5-Dim Diagnosis    → AI taste / title / efficiency / gap / product fit
Phase 6: HTML Layout        → wechat.html + cover.html → browser preview
```

Each phase shows progress and key outputs. You can skip phases or start from any point.

## Install

```bash
# Copy to your Claude Code skills directory
mkdir -p ~/.claude/skills/content-pipeline
cp SKILL.md ~/.claude/skills/content-pipeline/SKILL.md
```

Or clone and symlink:

```bash
git clone https://github.com/kajian/content-pipeline.git
mkdir -p ~/.claude/skills
ln -s $(pwd)/content-pipeline ~/.claude/skills/content-pipeline
```

## Usage

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

### Skip phases

```
/content-pipeline --skip-news          # Skip X news, use existing digest
/content-pipeline --skip-analysis      # Skip product analysis, go to topic
/content-pipeline --topic "AI替代论"    # Skip to writing with given topic
/content-pipeline --from-phase 4       # Start from Phase 4
```

## Optional: Enhanced Diagnosis

For full content diagnosis with case studies and style guidance, install [dbs-content](https://github.com/kajian/dbs-content) skill separately:

```bash
mkdir -p ~/.claude/skills/dbs-content
cp dbs-content/SKILL.md ~/.claude/skills/dbs-content/SKILL.md
```

The pipeline auto-detects `dbs-content` and uses it when available. Without it, a built-in simplified 5-dimension diagnosis runs instead.

## Customization

Fork this repo and modify these sections in `SKILL.md`:

| Section | What to change |
|---------|---------------|
| Phase 1.1 | X/Twitter accounts to follow |
| Phase 3.2 | Your methodology / knowledge framework |
| Phase 4.4 | Your personal data & credentials |
| Phase 6 HTML | Brand name, colors, author card |
| Footer | Platform-specific CTA copy |

## Output Files

The pipeline generates files in your project's `shortcontent/` directory (organized by week):

```
shortcontent/YYYY-MM-DD - YYYY-MM-DD/
├── YYYY-MM-DD.md              # Source article (all platforms)
├── YYYY-MM-DD-wechat.html     # WeChat formatted HTML
├── YYYY-MM-DD-cover.html      # Cover image template (900x383)
└── YYYY-MM-DD-cover.png       # Generated cover image
```

News digests go to `newsreflect/`, product analysis to `product-analysis/`.

## Content Philosophy

This skill embeds the **2-Hour Builder** content rules:

- **Title sells results/tools/conflict, content hides methodology**
- **Structure: Panic (1/3) → Reversal → Evidence → Action**
- **Material richness check** before writing (5 dimensions, need 3+)
- **Zero AI taste** — auto-detects and removes AI writing patterns
- **Product-first** — content serves monetization, not vanity

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- WebSearch access (for X news and product analysis)
- Browser tool (for cover screenshot, optional)

## License

MIT
