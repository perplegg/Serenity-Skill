# Serenity Research Skill

Serenity Research Skill 是一个 Claude Code workflow skill，用于按“Serenity / 紫苏叶”框架做产业链研究。

它的目标不是荐股，而是把一个行业、赛道、技术路线或产业主题拆成可验证的研究问题：从终端需求出发，逐层下钻产业链，寻找不可替代、供应商少、替代周期长、市场可能尚未充分认知的关键瓶颈环节。

> 重要声明：本项目仅用于产业链研究和研究假设生成，不构成任何投资建议。不得将输出解读为买入、卖出、持有、目标价或仓位建议。

## 功能

- 从终端需求出发建立产业链地图
- 逐层追问不可替代依赖
- 筛选候选“紫苏叶”环节
- 按成熟供应商数量和商业化程度降权/提权
- 制定数据源计划
- 使用公开资料、行业报告、公司公告、研报、Tavily MCP、Baostock 等做证据核验
- 主动生成反证清单、风险和失效条件
- 支持二阶段 A 股验证：从已有产业链报告映射到 A 股公司池，并选出 1-2 个研究优先级最高的候选标的
- 强制保存研究报告，避免只在对话中输出

## 目录结构

```text
serenity-research/
├── SKILL.md
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
├── examples.md
├── .gitignore
└── templates/
    ├── research-report.md
    ├── source-plan.md
    ├── evidence-log.md
    ├── rebuttal-checklist.md
    ├── a-share-mapping.md
    ├── research-report-review.md
    └── a-share-final-candidates.md
```

## 安装

把本目录复制到 Claude Code 项目的 skills 目录：

```text
your-project/
└── .claude/
    └── skills/
        └── serenity-research/
            ├── SKILL.md
            └── templates/
```

重启 Claude Code 或开启新会话后，skill 会被 Claude Code 发现。

## 使用示例

### 一阶段：产业链紫苏叶研究

```text
用 serenity-research 研究一下人形机器人产业链里的紫苏叶环节。
```

```text
按紫苏叶理论分析 AI 算力产业链，重点找不可替代且供应商少的环节。
```

```text
帮我找固态电池产业链里的紫苏叶环节，不要荐股，只要研究框架和反证清单。
```

一阶段默认输出并保存：

```text
research_YYYYMMDD_主题slug.md
```

### 二阶段：A 股验证

```text
基于 research_20260607_ai-compute.md，结合 A 股研报和 Baostock，进入二阶段验证，选出 1-2 个研究优先级最高的 A 股候选标的。
```

二阶段默认输出并保存：

```text
a_share_validation_YYYYMMDD_主题slug.md
```

## 研究原则

- 从终端需求出发，不从热门股票出发
- 每一层都追问：这一层要运转，下一层什么东西不可替代
- 优先寻找小而关键、被忽视、替代难、供应少的环节
- 区分“声称能做”和“已量产、已认证、能进入头部客户供应链”
- 超过 3 家成熟供应商默认降权；2 家重点研究；1 家或实质垄断高优先级
- 必须主动寻找反证，不能只证明原假设
- 必须写明风险、失效条件和下一步验证动作

## 数据源

推荐优先级：

1. 官方与监管文件：年报、季报、公告、招股书、专利、认证、监管披露
2. 产业一手资料：行业协会、展会资料、技术白皮书、供应商产品手册、客户认证
3. 专业研究机构：券商深度报告、行业数据库、咨询报告、会议材料
4. 金融与市场数据：Baostock、交易所披露、财务报表、历史行情
5. 技术与社区证据：论文、GitHub、开发者社区、招聘信息、专家访谈
6. 低置信度线索：自媒体、社交平台、无出处截图、市场传闻

Tavily MCP 和 Baostock 不是必需依赖；如果环境不可用，skill 会要求用户提供资料或标注“待验证”。

## Tavily MCP 配置

本 skill 在联网研究时会优先使用 Tavily MCP 搜索行业报告、公司公告、官网资料、研报、论文、展会材料和供应商信息。Tavily MCP 是可选依赖，但如果希望获得更完整的联网证据链，建议配置。

### 1. 准备 Tavily API Key

你需要先准备 Tavily API Key，并把它配置为本地环境变量。不要把 API Key 写入仓库文件，也不要提交到 GitHub。

推荐使用环境变量名：

```text
TAVILY_API_KEY
```

### 2. Claude Code MCP 配置示例

在 Claude Code 的 MCP 配置中添加 Tavily server。配置位置取决于你的使用方式，可以是用户级或项目级 Claude Code 配置。

示例配置：

```json
{
  "mcpServers": {
    "tavily": {
      "command": "npx",
      "args": ["-y", "tavily-mcp@latest"],
      "env": {
        "TAVILY_API_KEY": "${TAVILY_API_KEY}"
      }
    }
  }
}
```

如果你的 MCP server 包名、启动命令或部署方式不同，请以 Tavily 官方 MCP 文档和你本地 Claude Code 配置为准。

### 3. 配置后验证

在 Claude Code 中确认 Tavily MCP 工具可用后，可以用类似问题测试：

```text
测试一下 Tavily MCP 是否可用，搜索 AI 光模块 1.6T EML 行业报告。
```

如果 Tavily MCP 可用，skill 会优先使用它检索：

- 官方公告、年报、招股书
- 公司官网、产品手册、白皮书
- 行业报告、券商研报、会议材料
- 论文、专利、标准组织资料
- 竞争对手扩产、替代技术、负面反证

### 4. 不可用时的降级行为

如果 Tavily MCP 未配置或不可用，本 skill 仍可运行，但会：

- 优先使用用户提供的资料
- 使用已有本地文件和模板进行研究框架搭建
- 对无法联网核验的信息标注“待验证”
- 不编造成熟供应商数量、客户名单、产能或财务数据

## 输出边界

本 skill 明确禁止：

- 输出买入、卖出、持有、目标价、仓位建议
- 把券商评级或目标价作为入选理由
- 用 K 线走势替代产业链证据
- 因为公司热门就倒推其为紫苏叶
- 把单一来源信息当作确定结论
- 编造供应商数量、客户名单、产能、专利或财务数据

## License

MIT License
