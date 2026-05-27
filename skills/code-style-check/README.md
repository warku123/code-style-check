# code-style-check Skill

## 是什么

一个 Claude Code Skill，在提交 PR 前或 reviewer 审查时提供**语义级**代码风格第二意见。它不替代 Checkstyle，而是补充 Checkstyle 无法机械执行的规则。

## 触发方式

以下方式均可触发：

1. **直接指令**：输入 `style check`、`code style review`、`/code-style-check`
2. **粘贴 diff**：直接将 `git diff` 输出贴入对话，Skill 自动识别
3. **分支模式**：说 `check my current branch`，Skill 自动执行 `git diff develop..HEAD`

## 输出格式

每条发现格式为：`文件:行号 [TAG] 问题描述 + 修复建议`

三个严重等级：

| Tag | 含义 | 操作建议 |
|---|---|---|
| `[MUST]` | 违反规范或存在风险 | 必须修复 |
| `[SHOULD]` | 影响可读性/一致性 | 建议修复 |
| `[NIT]` | 微小偏好 | 可忽略 |

结尾一行总结：`LGTM` / `LGTM with nits` / `N [MUST] / M [SHOULD] findings`

## 审查清单

共 7 个类别，详见 [SKILL.md](./SKILL.md)：

1. **语义命名** — 名称是否准确传达意图
2. **跨文件检查** — 是否有跨文件的常量/逻辑不一致
3. **注释质量** — 注释是否有用、是否过时
4. **测试规范** — 测试是否有效、是否完整
5. **重构建议** — 方法是否过长、嵌套是否过深
6. **安全/正确性** — 是否有 NPE 风险、资源泄漏等模式
7. **TRON 项目规范** — 是否符合 java-tron 代码库约定

## 压制规则

如果不同意某个发现，在相关代码行的上一行加：

```java
// SKILL:OFF <category> -- reason: <简短原因>
```

## 与 Checkstyle 的分工

| 检查内容 | Checkstyle | code-style-check |
|---|---|---|
| 命名格式（camelCase 等） | 是 | 否 |
| 行长度、尾随空格 | 是 | 否 |
| import 检查、拼写 | 是 | 否 |
| 命名是否有意义 | 否 | 是 |
| 代码是否重复 | 否 | 是 |
| 注释是否过时/无用 | 否 | 是 |
| 安全风险模式 | 否 | 是 |
| 项目特有约定 | 否 | 是 |
