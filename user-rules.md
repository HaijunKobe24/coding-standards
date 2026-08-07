---
description: 用户级通用偏好规则（语言、工具、文档风格、Skills 使用等），适用于所有项目
alwaysApply: true
enabled: true
updatedAt: 2026-08-07T00:00:00.000Z
---

# 关键原则

## 通用

- 返回中文


## 工具相关

- 若要了解某个服务的 HTTP 接口时，优先访问其 Swagger，v3/v2
- 若要了解某个服务的 gRPC 接口时，优先使用起服务端反射机制
- 要获取开源项目源码时，优先从网络获取，若网络不通，告知我来解决
- 要了解某个 git 项目时，使用 git 命令直接读取项目内容，不能 clone
- Maven 仓库的存储路径在 settings 文件中
- 若项目中使用 JDK1.8，则 Maven 构建时，使用此 JDK：~/MyFiles/Program/JDK/jdk1.8.0_431.jdk/Contents/Home
- 需要 debug 本地 java 进程时，可以使用 Arthas，确保本地安装了 as.sh


## 文档相关

- README.md 文件，要使用 github 风格

## Skills/Commands 使用

### OpenSpec/opsx

- design.md
  1. 必须包含必要的设计图，Mermaid 语法
- tasks.md
  1. TDD 驱动，必须有明确的 Phase 划分，必须包含但不限于：接口契约 -> 测试用例 -> 功能实现 -> 集成测试
  2. 每个 Task 必须包含：关联的需求 ID、完成标准
