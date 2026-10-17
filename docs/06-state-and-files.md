# 状态与文件

可恢复的事实都落在文件上。编排进程的内存只保存当前一轮运行。

## 迭代

标识：`YYYYMMDD-HHMM-<slug>`。`slug` 来自标题的稳定短名，冲突时加短后缀。

状态：`working | blocked | completed | abandoned`。

对提出人的说法：进行中、等你处理、已完成、已取消。

```
Iteration
  id, title, status, stage
  createdAt, requester, channel
  repos[]                  可以为空，直到影响面判断完成
  branchName               feat/<id>，各仓同名
  base[key]                { name, sha }
  requirementVersion
  confirmations[]
```

目录：

```
iterations/.active.md                 只写当前 id；无活动任务时写 none
iterations/<id>/iteration.md          机器可读头 + 给人看的正文
iterations/<id>/activity.md
iterations/<id>/inbox/                用户原文
iterations/<id>/product-design.md
iterations/<id>/ux-design.md
iterations/<id>/technical-design.md
iterations/<id>/use-case-design.md
iterations/<id>/implementation-plan.md
iterations/<id>/health-check.md
iterations/<id>/pre-iteration.md
iterations/<id>/test-report.md
iterations/<id>/publish-report.md
iterations/<id>/external-blockers.md
iterations/<id>/alignment/<stage>.json
iterations/<id>/alignment/history/    只追加
```

阶段产物每版全文保留。新版本另存，不覆盖旧文件的历史副本。至少记录：版本号、产生时间、对应的需求版本、确认回执、相对上一版的变化。

机器可读头使用 [`contract/iteration.schema.json`](../contract/iteration.schema.json)。正文给人看。只有正文、没有合法的头，Kernel 视为不可执行。

## 锁

`runtime/workspace.lock`：

```
iterationId, holder: "host" | "human:<id>", since, pid, heartbeatAt
```

`new-iteration`、`checkout-from-base`、`dev` 和编码写入必须持有锁。`complete` 与 `abandon` 释放。心跳续期。

锁文件存在但进程已死、且超过 `staleLockMs` 没有心跳：检查报陈旧锁，**不自动删除、不自动抢占**。由 operator 或 admin 显式处理。

## 队列

`runtime/queue.json` 保存已确认、尚未开始的需求。字段见 [`contract/queue-item.schema.json`](../contract/queue-item.schema.json)。

K1 拒绝未确认的新想法时，不写入这条队列。未确认的对话留在会话存储里。

队列项可打开全文、等待原因和下一步。启动、撤回、重试使用同一串行事务。

## 确认

`runtime/confirmations.json` 或等价的按迭代分片文件，保存回执。执行下游命令时读取的是这份记录，不是页面状态，也不是模型回复。

回执绑定阶段产物的版本哈希。产物变化后旧回执失效。

## 事件与心跳

`runtime/events.jsonl`：每次 Kernel 调用追加一行，包含迭代、阶段、命令、退出码、耗时、通道、请求编号。不写提示词全文、密钥和源码正文。

`runtime/heartbeat.json`：默认 60 秒一次。超过 5 分钟无心跳，状态视为 Host 不可用。这只影响就绪判断，不推进阶段。

## 会话

需求对话、咨询对话、证据原文若含业务数据，加密后放在 `runtime/secrets/` 一类的受限目录，权限仅所有者可读写。换访问凭证后，旧密文不可用。

刷新、切换任务、关闭页面不丢已保存的对话和正在运行的回复。失败可以重试。提交和确认幂等。

发给模型的历史有明确预算。页面说明哪些内容被缩短，并允许展开已保存的原文。摘录不构成新的确认。

## 记忆的寿命

| 寿命 | 位置 | 用途 |
| --- | --- | --- |
| 瞬时 | 进程内的本轮运行 | 当前模型上下文 |
| 会话 | 加密对话与 `activity.md` | 恢复 |
| 迭代 | `iteration.md` 与各阶段产物 | 审计 |
| 项目 | 领域总览、改动索引、各仓介绍、外部阻塞惯例 | 跨迭代 |

恢复时重读锁、活动迭代、最近活动和当前产物版本。不依赖某一次 IDE 聊天。

禁止把密钥、完整仓库或依赖目录送进检索索引。

## 投影冲突

| 冲突 | 以谁为准 |
| --- | --- |
| `.active.md` 与锁不一致 | 锁 |
| 页面显示的阶段与产物/回执不一致 | 产物与回执 |
| 模型总结与工具输出不一致 | 工具输出 |
| 本地主干与刚 fetch 的远端主干不一致 | 刚 fetch 的 `origin/<baseBranch>` |
