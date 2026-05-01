# Smart Contract Auditor Skill

**[English](README.md) | [中文](README_CN.md)**

一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 自定义技能，将 Claude 打造为结构化的智能合约安全审计员。使用 **Slither** 进行结构分析，**Foundry** 进行漏洞 PoC 开发，遵循 6 阶段工作流程（阶段 0-5），旨在最大限度减少 AI 幻觉。Slither 漏洞扫描为可选功能（默认关闭）。

## 为什么需要这个技能？

AI 在审计智能合约时会产生幻觉 — 误读继承链、凭空捏造访问控制假设、生成无法运行的 PoC。本技能通过 **工具优先、代码其次** 的方法解决这个问题：AI 必须先使用 Slither 的 printer（从 AST 机器生成的事实数据）建立经过验证的代码库结构模型，*然后* 才能阅读任何源代码。

## 工作流程概览

| 阶段 | 名称 | 内容 |
|------|------|------|
| 0 | 环境搭建 | 检测项目类型，验证编译，识别审计场景（DeFi、Vault/ERC4626、代理合约、代币、跨链、质押、治理、永续合约、代币发行、收益聚合器、瞬态存储），选择提交平台，选择是否启用 Slither 扫描 |
| 1 | 结构侦察 | 运行 12 个 Slither printer，映射继承关系、入口点、函数-状态变量关系、存储布局和访问控制。**此阶段不读代码。** |
| 2 | 代码库文档 | 以阶段 1 输出为指引，深入阅读源代码。生成包含 Mermaid 图表的综合代码库文档（13 个章节）。 |
| 3 | 攻击规划 | 基于阶段 1+2 输出规划攻击向量 — 不读源代码。分析资金流、访问控制缺口、领域特定模式和权限/管理风险。可选运行 Slither 检测器。 |
| 4 | 验证与 PoC | 通过深入源代码验证攻击向量。为每个已确认漏洞同时编写 Foundry PoC 和发现报告。权限风险仅需报告无需 PoC。 |
| 5 | 最终报告组装 | 将发现组装为平台特定提交格式。严重性审查、去重、PoC 再验证。 |

**阶段 0-2 严格按顺序执行。** 阶段 1 通过 printer 建立结构模型（不读代码）。阶段 2 以阶段 1 为指引阅读源代码。阶段 3 从文档层面规划攻击。阶段 4 验证并编写 PoC。阶段 5 组装报告。

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
├── SKILL.md                                # 必需 — 技能入口
└── resources/
    ├── tools/
    │   ├── slither.md                      # 27 个 printer、99 个检测器、CLI 参数
    │   └── foundry.md                      # Forge CLI、cheatcode、cast、anvil、PoC 工作流
    ├── templates/
    │   ├── structural-summary.md           # 阶段 1 输出模板
    │   ├── codebase-report.md              # 阶段 2 输出模板（13 个章节）
    │   ├── codebase-report-guide.md        # 阶段 2 填写方法论
    │   ├── attack-plan.md                  # 阶段 3 输出模板
    │   ├── poc.md                          # 5 个 PoC 启动模板
    │   ├── code4rena.md                    # Code4rena 提交格式
    │   └── sherlock.md                     # Sherlock 提交格式
    ├── checklists/
    │   ├── adversarial-framework.md        # 对抗性思维与攻击模式
    │   ├── code-reading-checklist.md       # 25 项安全代码审阅清单
    │   ├── non-standard-tokens.md          # 14 种非标准 ERC20 行为
    │   ├── privilege-risk.md               # 权限角色枚举与密钥泄露分析
    │   └── playbooks/                      # 各领域攻击检查清单
    │       ├── amm-dex.md
    │       ├── vault-erc4626.md
    │       ├── proxy-upgradeable.md
    │       ├── lending-borrowing.md
    │       ├── cross-chain-bridge.md
    │       ├── staking-restaking.md
    │       ├── governance-timelock.md
    │       ├── perpetuals-derivatives.md
    │       ├── token-launch-bonding.md
    │       └── yield-aggregator.md
    └── phases/
        ├── phase-0-setup.md
        ├── phase-1-recon.md
        ├── phase-2-docs.md
        ├── phase-3-attack-planning.md
        ├── phase-4-verify-poc.md
        └── phase-5-report.md
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

AI 将依次执行 6 个阶段：环境搭建、结构侦察、代码库文档、攻击规划、验证与 PoC 开发、报告组装。

### 提交平台选择

在阶段 0 中，AI 会询问目标提交平台。当前支持情况：

| 平台 | 状态 |
|------|------|
| Code4rena | 已完成 |
| Sherlock | 已完成 |
| Immunefi | 计划中 |
| HackerOne | 计划中 |

默认使用 Code4rena。如需添加新平台，在 `resources/templates/` 中创建模板并更新 `SKILL.md` 中的阶段 0 配置。

## 文件结构

```
smart-contract-auditor/
├── SKILL.md                              # 核心技能 — 6 阶段审计工作流（阶段 0-5）
└── resources/
    ├── tools/
    │   ├── slither.md                    # 27 个 printer、99 个检测器、CLI 参数、附加工具
    │   └── foundry.md                    # Forge CLI、cheatcode、forge-std、cast、anvil、PoC 工作流
    ├── templates/
    │   ├── structural-summary.md         # 阶段 1 输出模板
    │   ├── codebase-report.md            # 阶段 2 输出模板（13 个章节，Mermaid 图表）
    │   ├── codebase-report-guide.md      # 阶段 2 填写方法论
    │   ├── attack-plan.md                # 阶段 3 输出模板（6 个章节）
    │   ├── poc.md                        # 5 个 Foundry PoC 启动模板
    │   ├── code4rena.md                  # Code4rena 提交格式与严重性指南
    │   └── sherlock.md                   # Sherlock 提交格式与去重模型
    ├── checklists/
    │   ├── adversarial-framework.md      # 对抗性思维与攻击模式
    │   ├── code-reading-checklist.md     # 25 项安全代码审阅清单
    │   ├── non-standard-tokens.md        # 14 种非标准 ERC20 代币行为
    │   ├── privilege-risk.md             # 权限角色枚举与密钥泄露分析
    │   └── playbooks/                    # 各领域攻击检查清单（10 种协议类型）
    └── phases/
        ├── phase-0-setup.md              # 环境搭建、范围发现、Slither 扫描选择
        ├── phase-1-recon.md              # 结构侦察（12 个 printer）
        ├── phase-2-docs.md               # 代码库文档（13 章节模板）
        ├── phase-3-attack-planning.md    # 攻击规划 + 权限风险分析
        ├── phase-4-verify-poc.md         # 源代码验证 + PoC 开发
        └── phase-5-report.md             # 最终报告组装
```

## 核心特性

### 反幻觉设计

技能强制执行严格的阶段顺序 — 阶段 0 搭建环境，阶段 1 仅使用 Slither printer（从 AST 派生的事实）建立结构模型，此时不阅读代码。阶段 2 以阶段 1 为指引阅读源代码。如果 AI 的代码解读与 printer 输出不一致，以 printer 为准。

### 模块化阶段架构

每个阶段在 `resources/phases/` 中都有独立的详细指令文件。各阶段引用其输入（前序阶段的输出）和所需的资源文件。这使核心 `SKILL.md` 保持简洁，同时在需要时提供深入指导。

### 全面的工具参考

`resources/tools/slither.md` 涵盖全部 27 个 printer、99 个检测器（含严重性/置信度）、CLI 参数和已知局限性。`resources/tools/foundry.md` 涵盖 Forge CLI、所有 cheatcode 签名、forge-std 辅助函数、主网 fork、Anvil、Cast 和不变量测试。

### 检测器到 PoC 的映射

针对每个常见的 Slither 检测器发现（重入、任意转账、未保护的升级等），技能提供了具体的 PoC 编写策略 — 部署什么合约、使用哪些 cheatcode、断言什么结果。

### 领域特定攻击手册

10 种协议类型的攻击检查清单：Vault/ERC4626、AMM/DEX、借贷、代理/可升级、治理、跨链/桥接、质押/再质押、永续合约/衍生品、代币发行/联合曲线、收益聚合器。

### 错误恢复

Slither 编译失败时的三级回退机制：修复并重试、部分分析、仅使用 Foundry 的降级模式。当错误持续存在时，AI 会查阅官方 GitHub 仓库。

## 使用的 Slither Printer

这 12 个 printer 在阶段 1 中运行，在任何代码阅读之前建立结构模型：

| Printer | 用途 |
|---------|------|
| `human-summary` | 合约数量、SLOC、ERC 标准、复杂度概览 |
| `function-summary` | **反幻觉基准真值。** 每个函数的可见性、修饰符、读写的状态变量 |
| `entry-points` | 所有状态变更的 external/public 函数及其访问的变量 |
| `vars-and-auth` | 各函数修改的状态变量 + 授权检查 |
| `inheritance` | 合约间文本继承关系 |
| `inheritance-graph` | 继承层次结构可视化（DOT） |
| `variable-order` | 存储槽位布局 — 每个合约的变量排序 |
| `modifiers` | 各函数应用的修饰符 |
| `require` | 各函数的所有 `require` 和 `assert` 条件 |
| `not-pausable` | 缺失 `whenNotPaused` 保护的函数 |
| `call-graph` | 跨函数调用关系（DOT） |
| `function-id` | Keccak256 函数选择器（检查代理模式中的碰撞） |

## 参与贡献

欢迎贡献，特别是：

- **新平台模板**（`resources/templates/immunefi.md` 等）
- **更多检测器的 PoC 映射**
- **新协议类型的领域特定攻击手册**

## 参考资料

- [Slither 文档](https://github.com/crytic/slither/wiki)
- [Foundry Book](https://book.getfoundry.sh/)
- [Code4rena 提交指南](https://docs.code4rena.com/competitions/submission-guidelines)
- [Code4rena 严重性分类](https://docs.code4rena.com/competitions/severity-categorization)
- [Sherlock 评判规则](https://docs.sherlock.xyz/audits/judging/judging)

## 许可证

MIT
