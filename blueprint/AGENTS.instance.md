# Agent Guide

本仓库是 **<产品名>** 的任务型 Agent。业务代码在 `target-repos/<key>` 的独立 Git clone 中。真正改任务状态和业务仓库的只有 Kernel 命令。

## 必读

1. `README.md`
2. `automation.config.json`
3. `docs/domain-overview.md`
4. `docs/change-map.md`
5. `docs/repositories/*.md`
6. `iterations/.active.md` 与当前迭代目录
7. `status` 的机器可读输出

架构不变量以 Business Agent 蓝图的 K1–K10 为准。本文件只补充这个业务的事实。

## 这个业务

- 领域：<一句话>
- 已登记仓库：<key 与入口名>
- 保护分支：<列表>
- 提出人：<角色>
- 验收与发布：<谁确认>

## 边界

- 做：已登记仓库上、已确认的需求。
- 不做：<该业务明确排除的系统>。未登记仓库不写入。
- 业务代码默认只读。没有该次需求的确认和实施授权时，不修改 `target-repos`。
- 调研、审查、巡检和未授权的优化不构成改码授权。
- 自动编码是否开启以配置为准。开启后仍逐次校验版本、分支和文件范围。
- 编码必须跑在够得到这些仓库及其构建依赖的 Host 上。

## 硬闸门

- K1：至多一条 `working` / `blocked`。未收口时拒绝新任务，并说明当前 id、阶段和如何 complete 或 abandon。
- K2：从各仓远端最新 `baseBranch` 创建 `feat/<iteration-id>`。
- K3：脏工作区不切分支。
- K7：阶段确认必须写成回执。口头同意不够。
- K8：开发一次一个仓库，且已在本期功能分支上。
- K9：保护分支只有带确认的发布命令可以推送。默认只写交接计划。

## 开工

- 先 `check`。因未收口失败时，继续当前任务或收口，不开新需求。
- 仓库缺失或文档与配置不一致时先 `init`。
- 只有绑定里列出的外部条件才请人处理。

## 迭代

- 新需求先对话。确认当前版本后才入队。
- 每个阶段的开始和结束写入 `activity.md`。
- 不提交 `target-repos/` 到本仓库。
- 维护任务有界。做完当前批次就 complete 或 abandon，待办留在文档里。
