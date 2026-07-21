# AI PM Job Analyzer 🧠📊

[![Platform](https://img.shields.io/badge/platform-WorkBuddy%20%7C%20Claude%20Code%20%7C%20Cursor-blue)]()
[![License](https://img.shields.io/badge/license-MIT-green)]()

**AI产品经理招聘岗位分析报告生成器** — 从多平台抓取真实岗位，按公司实力四层分级，
提炼能力模型，对标用户差距，生成结构化 HTML 求职分析报告。

## 快速开始

1. **复制用户画像模板并填写你的信息：**

```bash
cp config/user-profile.example.yaml config/user-profile.yaml
# 编辑 config/user-profile.yaml，填入你的背景信息
```

2. **触发 Skill：**

向 AI 助手说：「帮我分析AI产品经理的招聘市场」

Skill 会自动：
- 读取你的画像
- 检测并安装 mcp-jobs（如需）
- 搜索匹配岗位
- 生成 HTML 报告

## 功能架构

```
用户画像 → 多平台岗位抓取 → 四层分级 → 能力提炼 → 差距分析 → 行动建议 → HTML报告
```

### 四层分级体系

| 层级 | 公司类型 | 分析视角 |
|------|---------|---------|
| **Tier 1** 🥇 | 全球500强外企 | 目标岗，2-3年发展方向 |
| **Tier 2** 🥈 | 互联网大厂 | 跳板岗，1-2年可达 |
| **Tier 3** 🥉 | 行业独角兽/上市 | 主攻岗，当前最佳匹配 |
| **Tier 4** 💜 | 远程/国际化团队 | 灵活选择 |

### 支持行业

物流供应链 / 金融 / 医疗健康 / 电商零售 / 企业服务(SaaS) / 通用AI — 各行业有对应的 Tier 公司列表。

## 数据源

| 优先级 | 数据源 | 适用条件 |
|--------|--------|----------|
| 🥇 | **mcp-jobs** (猎聘/Boss/智联/51job) | 自动检测+安装 |
| 🥈 | **WebSearch + 官网** | 任何环境 |
| 🥉 | **WebSearch only** | 零依赖回退 |

> 详细安装指南见 [`references/mcp-jobs-setup.md`](references/mcp-jobs-setup.md)

## 报告示例

生成的 HTML 报告包含：

- 📋 **精选岗位卡片**（含真实招聘链接）
- 🧠 **六大维度能力模型**（AI技术/数据/产品/行业知识/软技能/合规）
- 📊 **差距分析表**（四层级匹配度评估）
- 🎯 **三层投递行动路线图**
- 💰 **薪资参考矩阵**

## 文件结构

```
ai-pm-job-analyzer/
├── SKILL.md                          # 核心工作流程
├── assets/
│   └── report-template.html          # HTML 报告 CSS 模板
├── config/
│   └── user-profile.example.yaml     # 用户画像模板（复制后填写）
└── references/
    └── mcp-jobs-setup.md             # 多平台 mcp-jobs 安装指南
```

## 兼容性

| 平台 | 状态 |
|------|------|
| WorkBuddy | ✅ 完全支持（含 mcp-jobs 自动安装） |
| Claude Code / Cursor | ✅ 支持（需手动安装 mcp-jobs） |
| 通用环境 | ✅ WebSearch 回退可用 |

## License

MIT
