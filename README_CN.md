# Smart Contract Auditor Skill

**[English](README.md) | [中文](README_CN.md)**

一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 自定义技能，将 Claude 打造为结构化的智能合约安全审计员。使用 **Slither** 进行静态分析，**Foundry** 进行漏洞 PoC 开发，遵循 7 阶段工作流程，旨在最大限度减少 AI 幻觉。

## 为什么需要这个技能？

AI 在审计智能合约时会产生幻觉 — 误读继承链、凭空捏造访问控制假设、生成无法运行的 PoC。本技能通过 **工具优先、代码其次** 的方法解决这个问题：AI 必须先使用 Slither 的 printer（从 AST 机器生成的事实数据）建立经过验证的代码库结构模型，*然后* 才能阅读任何源代码。

## 工作流程概览

| 阶段 | 名称 | 内容 |
|------|------|------|
| 0 | 环境搭建 | 检测项目类型，验证编译，识别审计场景（DeFi、代理合约、代币、跨链），选择提交平台 |
| 1 | 结构事实建立 | Slither printer 映射继承关系、入口点、函数-状态变量关系和存储布局。**此阶段不读代码。** |
| 2 | 攻击假设生成 | 更多 printer 映射访问控制、暂停保护缺口、验证条件、数据流。生成优先级排序的目标列表。**仍然不读代码。** |
| 3 | 定向代码审阅 | 现在开始读 `.sol` 文件，由阶段 1-2 的输出引导。每个判断都与 printer 输出交叉验证。 |
| 4 | 自动漏洞扫描 | Slither 检测器（全量扫描 + 定向扫描：重入、访问控制、预言机/DeFi）。与阶段 3 并行运行。 |
| 5 | 分类与深度分析 | 将发现与代码审阅交叉对比。分类为：已确认 / 疑似 / 误报。 |
| 6 | PoC 开发 | 为每个已确认的 High/Medium 漏洞编写 Foundry 漏洞利用测试。 |
| 7 | 报告生成 | 按平台格式提交（默认 Code4rena）。 |

## 安装

### 前置要求

- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI 或桌面应用
- 已安装 [Foundry](https://getfoundry.sh/)（`forge`、`cast`、`anvil`）
- 已安装 [Slither](https://github.com/crytic/slither)（`pip install slither-analyzer` 或 `uv tool install slither-analyzer`）

安装 Foundry：

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

安装 Slither：

```bash
pip install slither-analyzer
# 或
uv tool install slither-analyzer
```

### 方式 A：全局安装（推荐 — 所有项目通用）

安装一次，在所有项目中都可使用：

```bash
# 克隆仓库
git clone https://github.com/ARIS2333/smart-contract-auditor.git

# 将技能文件夹复制到 Claude Code 的全局技能目录
cp -r smart-contract-auditor/smart-contract-auditor ~/.claude/skills/smart-contract-auditor
```

或使用符号链接（仓库更新后自动同步）：

```bash
ln -s $(pwd)/smart-contract-auditor/smart-contract-auditor ~/.claude/skills/smart-contract-auditor
```

### 方式 B：项目级安装（仅当前项目生效）

安装到特定审计项目中。适合将技能随项目一起 git commit，团队成员 clone 后即可使用：

```bash
# 在审计项目根目录下
mkdir -p .claude/skills
cp -r /path/to/smart-contract-auditor/smart-contract-auditor .claude/skills/smart-contract-auditor
```

### 验证安装

打开 Claude Code（全局安装在任意项目，项目级安装在目标项目中）：

1. 输入 `/` — 自动补全菜单中应出现 `smart-contract-auditor`
2. 或直接输入 `/smart-contract-auditor`

如果没有出现，请检查文件夹结构是否正确：

```
~/.claude/skills/smart-contract-auditor/    # 全局
# 或
.claude/skills/smart-contract-auditor/      # 项目级
├── SKILL.md                                # 必需
└── resources/                              # 参考文档和模板
    ├── slither-audit-guide.md
    ├── foundry-audit-guide.md
    ├── poc-template.md
    └── templates/
        └── code4rena.md
```

## 使用方法

### 开始审计

在目标智能合约项目中打开 Claude Code，调用技能：

```
/smart-contract-auditor
```

或用自然语言描述需求 — Claude 会自动识别并启用该技能：

```
审计这个 Foundry 项目的安全漏洞
```

```
对这个代码库运行 slither，并为所有关键发现编写 PoC
```

AI 将依次执行 7 个阶段：环境搭建、结构分析、攻击面映射、代码审阅、漏洞扫描、分类筛选、PoC 开发和报告生成。

### 提交平台选择

在阶段 0 中，AI 会询问目标提交平台。当前支持情况：

| 平台 | 状态 |
|------|------|
| Code4rena | 已完成 |
| Sherlock | 计划中 |
| Immunefi | 计划中 |
| HackerOne | 计划中 |

默认使用 Code4rena。如需添加新平台，在 `resources/templates/` 中创建模板并更新 `SKILL.md` 阶段 0.6 中的表格。

## 文件结构

```
smart-contract-auditor/
├── SKILL.md                              # 核心技能 — 7 阶段审计工作流
└── resources/
    ├── slither-audit-guide.md            # 28 个 printer、103 个检测器、CLI 参数、Python API
    ├── foundry-audit-guide.md            # Forge CLI、cheatcode、cast、anvil、PoC 开发
    ├── poc-template.md                   # Foundry PoC 模板
    └── templates/
        └── code4rena.md                  # Code4rena 提交格式和检查清单
```

## 核心特性

### 反幻觉设计

技能强制执行严格的阶段顺序 — 阶段 0-2 仅使用 Slither printer（从 AST 派生的事实）建立结构模型，然后 AI 才能阅读代码。如果 AI 的代码解读与 printer 输出不一致，以 printer 为准。

### 内联 Slither 参考

SKILL.md 在每个阶段中直接嵌入了最常用的 Slither 命令（18 个 printer、26 个高/中危检测器、预设扫描命令），AI 无需翻阅完整指南即可完成常规操作。

### 检测器到 PoC 的映射

针对每个常见的 Slither 检测器发现（重入、任意转账、未保护的升级等），技能提供了具体的 PoC 编写策略 — 部署什么合约、使用哪些 cheatcode、断言什么结果。

### 错误恢复

Slither 编译失败时的三级回退机制：修复并重试、部分分析、仅使用 Foundry 的降级模式。当错误持续存在时，AI 会查阅官方 GitHub 仓库。

## 使用的 Slither Printer

这些 printer 是反幻觉工作流的核心：

| Printer | 阶段 | 用途 |
|---------|------|------|
| `human-summary` | 1 | 范围和复杂度概览 |
| `loc` | 1 | 代码行数统计 |
| `inheritance-graph` | 1 | 完整合约继承关系（DOT 格式） |
| `inheritance` | 1 | 文本继承摘要 |
| `c3-linearization` | 1 | 菱形继承解析 |
| `entry-points` | 1 | 所有状态变更入口点 |
| `function-summary` | 1 | 函数可见性、修饰符、状态变量访问 |
| `variable-order` | 1 | 存储槽位布局 |
| `vars-and-auth` | 2 | 状态写入 + 权限检查 |
| `modifiers` | 2 | 各函数的修饰符覆盖情况 |
| `not-pausable` | 2 | 缺失暂停保护的函数 |
| `data-dependency` | 2 | 用户输入到状态变更的追踪 |
| `call-graph` | 2 | 跨函数调用关系 |

## 参与贡献

欢迎贡献，特别是：

- **新平台模板**（`resources/templates/sherlock.md` 等）
- **更多检测器的 PoC 映射**
- **特定场景检查清单**（借贷协议、AMM、跨链桥等）

## 参考资料

- [Slither 文档](https://github.com/crytic/slither/wiki)
- [Foundry Book](https://book.getfoundry.sh/)
- [Code4rena 提交指南](https://docs.code4rena.com/competitions/submission-guidelines)
- [Code4rena 严重性分类](https://docs.code4rena.com/competitions/severity-categorization)

## 许可证

MIT
