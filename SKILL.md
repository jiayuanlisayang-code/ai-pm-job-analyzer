---
name: ai-pm-job-analyzer
description: >
  AI产品经理招聘岗位分析报告生成器。当用户请求分析特定行业/方向的AI产品经理招聘市场、
  对标自身背景与岗位要求的差距、或生成求职能力分析报告时使用此技能。
  Trigger: user asks to analyze AI PM job market, benchmark their profile against job requirements,
  or generate a career analysis report. Supports any industry, location, company type, and experience level.
---

# AI PM Job Market Analyzer

## Overview

通用求职分析工具：收集用户画像 → 多平台抓取真实岗位 → 按公司实力四层分级 →
提炼能力模型 → 差距分析 + 分层投递策略 → 生成结构化 HTML 报告。

## 阶段零：读取用户画像

**运行前，先尝试读取本地画像配置**（如果该文件存在）：

```
Read: config/user-profile.yaml
```

如果读到有效配置，将其作为默认用户画像，后续阶段直接使用。
如果文件不存在或为空，进入「输入信息收集」流程向用户询问。

> config/ 目录下的实际画像文件已在 .gitignore 中排除，不会被上传。
> 新用户可复制 `config/user-profile.example.yaml` 填入自己的信息。

## 输入信息收集

如果用户画像不可用，按以下维度收集（不逐条追问，一次性列出缺失项让用户补充）：

- **工作背景**：行业、年限、具体产品/模块
- **学历背景**：学校级别、专业、学位
- **目标岗位**：AI产品经理 或 其他AI相关角色
- **目标行业**：物流/金融/医疗/电商/制造/通用等
- **公司类型偏好**：外企/大厂/独角兽/创业/远程
- **工作地点**：城市名 或 "远程"
- **当前AI能力水平**：项目经验、技术概念熟悉度

用户提供新信息时覆盖对应字段，未提供则保持默认。

## 平台兼容性

此 skill 跨平台运行。核心逻辑在所有平台一致；数据抓取层由阶段一自动适配。

### 核心原则

**数据源优先使用 mcp-jobs，但绝不强依赖它。**
阶段一会自动检测 → 安装 → 回退，确保在任何环境下都能生成完整报告。

### 数据源优先级

| 优先级 | 数据源 | 适用条件 | 岗位数据质量 |
|--------|--------|----------|-------------|
| 🥇 | **mcp-jobs MCP** | Node.js + Playwright 可用 | ⭐⭐⭐⭐⭐ 结构化、实时、覆盖广 |
| 🥈 | **WebSearch + 公司官网** | 任何有网络的环境 | ⭐⭐⭐⭐ 可覆盖大部分岗位 |
| 🥉 | **WebSearch only** | 无 fetch 能力的环境 | ⭐⭐⭐ 基本覆盖 |

### 报告输出

无论哪种数据源，生成的 HTML 报告格式、样式、结构完全一致。
仅在岗位数量和结构化程度上可能有差异。

> 完整的多平台安装指南见 `references/mcp-jobs-setup.md`。

## 核心工作流程

## 阶段一：数据源检测 + 自动配置 mcp-jobs

**核心原则：优先使用 mcp-jobs（结构化的真实岗位数据），检测不到则自动安装，安装失败则 WebSearch 回退。**

### 三步检测与安装流程

```
┌─ 步骤1：检测 mcp_search_job 是否可调用
│   ├─ 可用 → 直接使用（跳到阶段二）
│   └─ 不可用 → 进入步骤2
│
├─ 步骤2：尝试自动安装 mcp-jobs
│   │
│   │  判断当前平台：
│   │
│   ├─ WorkBuddy 平台（特征：存在 ~/.workbuddy/ 目录）
│   │   ├─ 安装 npm 包：
│   │   │   cd ~/.workbuddy/binaries/node/workspace
│   │   │   npm install mcp-jobs
│   │   │   npx playwright install chromium
│   │   ├─ 写入 MCP 配置到 ~/.workbuddy/mcp.json：
│   │   │   {
│   │   │     "mcpServers": {
│   │   │       "mcp-jobs": {
│   │   │         "command": "npx",
│   │   │         "args": ["-y", "mcp-jobs"],
│   │   │         "disabled": false
│   │   │       }
│   │   │     }
│   │   │   }
│   │   └─ 提示用户：「mcp-jobs 已安装，请到连接器管理页面点击 Trust 启用，
│   │   │              然后回复'继续'我将重新检测并开始搜索岗位」
│   │   └─ 等待用户确认后重新检测
│   │
│   └─ 其他平台（Claude Code / Cursor / 通用）
│       ├─ 指导用户安装（参考 references/mcp-jobs-setup.md）：
│       │   npm install -g mcp-jobs
│       │   并配置 MCP 客户端
│       └─ 如用户表示无法/不想安装 → 跳过，使用步骤3
│
└─ 步骤3：WebSearch 回退（任何平台可用）
    └─ 使用搜索引擎 + site:zhipin.com/liepin.com 限定范围
       能覆盖大部分岗位，质量略低但完全可用
```

### 安装成功后的验证

mcp_search_job 可用后，验证搜索是否正常：

1. 尝试搜索 `keyword: "AI产品经理"` + `location: "上海"`
2. 如果返回结果 → 安装成功，进入阶段二
3. 如果返回空或报错 → 检查 Chromium 是否已安装：`npx playwright install chromium`，重试

### 详细安装指南

完整的多平台安装文档见 `references/mcp-jobs-setup.md`。
包括 Claude Code、Cursor、Docker 等所有环境的配置方法。

## 阶段二：多渠道抓取真实岗位

根据「目标行业 + 公司类型 + 地点」并行搜索。

**路径 A：mcp-jobs 可用时（最优，结构化数据）**

1. `mcp_search_job` 搜索 `keyword: "AI产品经理 [行业]"` + `location: [地点]`
2. `mcp_search_job` 搜索 `keyword: "AI产品经理 [行业相关场景]"` + `location: [地点]`
3. 对关键岗位调用 `mcp_job_detail` 获取完整 JD 描述

**路径 B：WebSearch 回退（通用，覆盖中文平台 + 外企）**

并行搜索以下关键词组：

```
组1（中文平台）: "AI产品经理 招聘 [行业] [地点] site:zhipin.com OR site:liepin.com"
组2（外企英文）: "AI product manager [industry] [city] hiring [year]"
组3（目标公司）: "[公司名] AI product manager [location] hiring"
组4（能力分析）: "AI产品经理 [行业] 核心能力要求 技能清单"
```

**路径 C：目标公司 careers 页面直搜（所有平台通用**）

根据用户公司偏好，针对性搜索 careers 页面。模板示例（按行业可替换公司名）：

```
外企物流/供应链: site:careers.dhl.com OR site:amazon.jobs OR site:maersk.com "AI product manager"
外企金融: site:careers.jpmorgan.com OR site:goldmansachs.com "AI product manager"
大厂电商: site:alibaba.com OR site:meituan.com "AI产品经理"
独角兽: site:zhipin.com "[公司名] AI产品经理"
```

### 阶段三：岗位筛选与四级分层

#### 筛选标准

| 匹配度 | 标准 |
|--------|------|
| **强匹配** ⭐ | 行业匹配 + 经验区间匹配 + 公司类型匹配 |
| **中匹配** | 行业相近 或 经验接近 或 有部分交集 |
| **目标岗** | 当前不匹配但可作为发展目标的高级岗位 |

#### 四层分级体系（按公司实力）

每层使用独立的视觉标识：

| 层级 | 标准 | 示例公司（物流领域） | CSS类 |
|------|------|---------------------|-------|
| **Tier 1 外企巨头** | 全球500强 / 行业全球Top3 | Amazon, Apple, Maersk, DHL, Flexport | `tier-1` 金色 |
| **Tier 2 互联网大厂** | 千亿市值 / 平台级 | 阿里巴巴/蚂蚁, 美团, 拼多多, 字节, NIO | `tier-2` 蓝色 |
| **Tier 3 行业独角兽/上市** | 细分赛道头部 / 快速成长 | 极兔, 壹米滴答, 申通, 满帮 | `tier-3` 绿色 |
| **Tier 4 远程/灵活** | 国际化远程团队 / 创业 | 按目标行业动态确定 | `tier-4` 紫色 |

**行业适配规则**：根据用户的目标行业动态替换每层示例公司：

| 目标行业 | Tier 1 示例 | Tier 2 示例 | Tier 3 示例 |
|----------|-----------|-----------|-----------|
| 物流/供应链 | Amazon, Maersk, DHL, Flexport | 蚂蚁, 美团, 拼多多, NIO | 极兔, 壹米滴答, 满帮, 货拉拉 |
| 金融 | Goldman Sachs, JPMorgan, Bloomberg | 蚂蚁, 微众, 京东金融, 东方财富 | 同花顺, 老虎证券 |
| 医疗健康 | Siemens Healthineers, Philips, GE Healthcare | 腾讯医疗, 百度健康, 阿里健康 | 微医, 丁香园, 医联 |
| 电商/零售 | Amazon, Walmart, Shopify | 阿里, 拼多多, 京东, 抖音电商 | SHEIN, 得物 |
| 企业服务/SaaS | Microsoft, Salesforce, SAP, Oracle | 字节飞书, 腾讯企点, 金蝶 | 钉钉, 用友 |
| 通用AI | Google, Microsoft, Meta, Apple | 字节, 阿里, 腾讯, 百度 | 商汤, 旷视, Minimax |

### 阶段四：提炼能力模型（6维度）

从所有 JD 中提炼高频能力要求，归纳为通用框架：

1. **AI/ML 技术理解力**：LLM原理、RAG、Agent/智能体、Prompt Engineering、模型评测
2. **数据能力**：SQL、数据闭环、指标体系与A/B实验、成本意识（Token/推理优化）
3. **产品基本功**：PRD、路线图、用户研究、项目管理、商业化思维
4. **行业领域知识**：根据目标行业定制（从JD中的业务关键词提取）
5. **软技能**：跨部门协作、英文/双语能力、利益相关方管理、变革管理
6. **AI 治理与合规**：AIGC合规、隐私保护、AI安全护栏、可解释性

每个维度列出 4-5 个知识点，标注来源于哪些公司的 JD。

### 阶段五：差距分析

**优势分析（绿色 `.strength-box`）：**
- 列举用户与市场要求明确匹配的点
- 重点突出用户独特组合的稀缺性（如"行业×产品×学历"）

**短板分析（黄色 `.gap-box` + 层级匹配度表格）：**

| 列名 | 说明 |
|------|------|
| 层级 | Tier 1-4 |
| 代表公司 | 该层级的 2-3 家典型公司 |
| 经验门槛 | 该层级岗位通常要求的经验年限 |
| AI经验要求 | 对AI项目经验的要求程度 |
| 你的匹配度 | 🟢🟢 / 🟢 / 🟡 / 🔴 |
| 投递策略 | 针对性的投递建议 |

### 阶段六：行动建议（3步分层法）

1. **第一步：补能力**（1-2个月）— 按用户AI能力现状定制课程/项目推荐
2. **第二步：分层投递**（持续）— 从最匹配层级开始，逐层向上冲刺
3. **第三步：长期关注** — 目标公司列表、平台推荐、简历关键词

### 阶段七：薪资参考表

按层级 × 经验级别给出薪资矩阵（月薪范围 + 年包参考，标注货币单位）。

### 阶段八：核心结论

一段有力总结，帮用户看清定位、最优路径和关键行动。

## 报告样式规范

使用 `assets/report-template.html` 中的 CSS 体系生成报告。核心设计：

- **配色**：主色 `#2563eb`（蓝）；渐变 header `#1e3a5f → #2563eb → #7c3aed`
- **卡片**：`.job-card`（白底边框）、`.skill-card`（灰底）、`.summary-card`（蓝左边条）
- **分层标识**：`.tier-badge` — tier-1 金色 / tier-2 蓝色 / tier-3 绿色 / tier-4 紫色
- **推荐标记**：`.rec`（蓝紫渐变）、`.rec-star`（金色左边条，最佳匹配）
- **分析框**：`.strength-box`（绿色）、`.gap-box`（黄色）
- **链接**：`.job-link`（蓝色虚线，hover变实线）
- **响应式**：`@media (max-width: 640px)` 移动端适配
- **字体**：系统字体栈 `-apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Microsoft YaHei', sans-serif`

### 岗位卡片字段规范

每个 `.job-card` 必须包含：
- 公司名 + 实力标签 + 匹配度标签 + **真实可点击的招聘链接**（至少1个）
- 岗位标题（保留原语言）
- 元信息行：📍地点 💰薪资 📋经验要求
- 核心要求摘要（3-5条要点）
- 匹配度评语（`.highlight` 黄色高亮）

## 通用注意事项（跨平台）

1. **所有链接必须真实可点击**：搜索获取的URL直接嵌入，不使用占位符
2. **薪资统一为月薪**：年包换算为月薪（÷12~16），外币标注原始货币
3. **数据时效**：footer 标注搜索日期 +「职位实时变化请以官方页面为准」
4. **语言规则**：正文用中文，岗位标题和外企JD关键字段保留原语言
5. **文件命名**：`AI产品经理招聘分析报告.html`，放工作目录下
6. **报告展示**：生成后提示用户打开 HTML 文件（WorkBuddy 平台自动调用 preview）

## 报告逻辑检查清单

生成报告前逐项校验：

- [ ] 每个 Tier 的岗位数量合理（Tier 1: 3-5，Tier 2: 4-6，Tier 3: 2-4，Tier 4: 1-3）
- [ ] 能力模型「行业领域知识」维度与用户目标行业一致
- [ ] 四级分层示例公司与行业适配规则匹配
- [ ] 差距分析表各Tier匹配度评估与用户经验年限对得上
- [ ] 行动建议投递优先级与匹配度评估表自洽
- [ ] 薪资参考表区间与JD实际薪资一致
- [ ] 所有招聘链接URL格式正确
