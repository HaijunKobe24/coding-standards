# Coding Standards

个人编码规范与偏好规则统一管理仓库，供 AI 编程助手和开发者参考。

## 规范文档

| 文档 | 加载方式 | 说明 |
|------|----------|------|
| [java-standards-lite.md](java-standards-lite.md) | 始终加载 | Java 编码规范精简版，提取 AI 编码时最易违反的核心规则（~290 行） |
| [java-standards.md](java-standards.md) | 按需加载 | Java 编码规范完整版，涵盖所有细节和示例（~630 行） |
| [user-rules.md](user-rules.md) | 始终加载 | 用户级通用偏好规则（语言、工具、文档风格等） |

## 使用方式

本仓库通过软链接接入 `~/.agents/rules/`，AI 编程助手每次对话自动加载：

```bash
# 初始化软链接（仅首次）
ln -s $(pwd)/java-standards-lite.md ~/.agents/rules/java-standards-lite.md
ln -s $(pwd)/java-standards.md ~/.agents/rules/java-standards.md
ln -s $(pwd)/user-rules.md ~/.agents/rules/user-rules.md
```

## 维护者

haijun.gao
