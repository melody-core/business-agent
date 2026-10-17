# 绑定模板

实例化时复制本目录的结构到新仓库，再替换占位内容。

| 路径 | 用途 |
| --- | --- |
| `automation.config.example.json` | 配置骨架。仓库数组按 [repository schema](../contract/repository.schema.json) 填写 |
| `.env.example` | 环境变量名。复制为 `.env` 后填写值 |
| `AGENTS.instance.md` | 实例根目录 `AGENTS.md` 的结构 |
| `domain/` | 领域总览、改动索引、单仓介绍 |
| `templates/` | 迭代档案的阶段正文 |

示例见 [`examples/bookstore/`](../examples/bookstore/README.md)。
