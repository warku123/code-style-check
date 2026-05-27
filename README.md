# Code Style Check — Claude Code Skill

Code style self-review skill for Claude Code，用于补充 Checkstyle 硬门禁无法覆盖的**语义级**代码审查。

## 定位

Checkstyle 负责机械规则（命名格式、行长度、import 检查等），本 Skill 负责语义规则（命名是否清晰、有无重复代码、注释是否过时、安全风险模式等）。

## 审查能力（7 大类）

| 类别 | 检查内容 | 示例 |
|---|---|---|
| 语义命名 | 变量名是否含糊、方法名是否与实际行为一致 | `temp`、`data` → 更具体的名称 |
| 跨文件检查 | 硬编码常量是否已有定义、重复逻辑 | `"solid"` → 引用 `Args` 常量 |
| 注释质量 | 过时注释、可有可无的注释、拼写 | `// increment i` → 删除或改写 |
| 测试规范 | 无断言测试、仅测 happy path、flaky 模式 | `test1()` → `shouldReject_whenBalanceInsufficient()` |
| 重构建议 | 方法过长、嵌套过深、重复 switch | 80 行/4 层嵌套为阈值 |
| 安全/正确性 | equals 不对称、资源泄漏、空 catch | try-with-resources、防御性拷贝 |
| 项目规范 | TRON 特有约定 | 日志参数化、energy 计算用安全数学 |

## 安装

```bash
claude plugins install code-style-check@warku123/code-style-check
```

## 实测数据

对 java-tron framework 模块 5 个最大文件（共 13,257 行）的审查结果：

- Token 消耗：~143K total
- 发现问题：8 MUST / 54 SHOULD / ~88 NIT
- 误报率：< 2%
- 详细数据见 [reports/code-style-review-sample-5-files.md](reports/code-style-review-sample-5-files.md)

## 目录结构

```
code-style-check-repo/
├── .claude-plugin/          # Marketplace 注册
│   └── marketplace.json
├── skills/
│   └── code-style-check/    # Skill 主体
│       ├── SKILL.md          # Skill 定义（审查规则）
│       └── README.md         # 使用说明
└── reports/                  # 实测报告（gitignore）
```
