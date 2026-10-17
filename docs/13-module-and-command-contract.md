# 模块与命令合同

实现可以换语言和目录名。对外行为和失败条件不变。HTTP 与命令行接收同一组字段，返回同一组错误码。

## 模块

| 模块 | 职责 |
| --- | --- |
| iteration | 创建档案、解析机器可读头、计算阶段是否可进入 |
| lock | 获取、续期、释放、报告陈旧。不自动抢占 |
| branch | fetch 主干、创建 `feat/<id>`、核对当前分支 |
| confirm | 写入回执、按产物哈希判断有效 |
| alignment | 独立审查、服务端状态、指纹与历史 |
| queue | 已确认需求的排队、启动、撤回。与 K1 同一事务 |
| publish | 生成交接；仅在确认和身份满足时推送允许的分支 |
| coding-gate | 运行凭证、路径、旧哈希、验证指纹 |
| dialogue | 需求整理与版本。不直接 new-iteration |
| host | 会话、API、事件流 |
| web | 业务语言界面 |
| evidence | 可选。只读工具与证据编号 |

## 命令

### status

只读。返回活动迭代、锁、心跳、队列计数、执行器是否就绪。支持机器可读输出。

### check

只读，可写 `health-check.md`。范围：`workspace`、`business` 或两者。发现符号链接、绑定与文档不一致、活动投影与锁不一致时失败。`--allow-active-iteration` 仅忽略「已有活动迭代」这一项，其它失败仍失败。

### init

修复 check 中标记为可自动处理的项：目录、按绑定 clone、安装已声明的依赖。超过修复轮次仍失败则停。需要人的项写入外部阻塞，不反复重试。

### new-iteration

输入标题、请求人、通道。K1 失败时不创建目录。成功时创建 `iterations/<id>/`，写入头，获取锁，更新 `.active.md`，追加事件。

Web 主路径通常不直接调用它。对话确认后由队列启动调用。保留命令是为了研发旁路和测试。

### enqueue

输入已确认的需求快照。未确认拒绝。已有相同版本的队列项则返回原项。

### checkout-from-base

输入迭代 id 与仓库 key 列表。执行 K2、K3。把基线提交写入元数据。

### confirm

输入迭代、阶段、决定、操作者角色、产物版本。核对：角色允许、阶段顺序、对齐状态、产物哈希。`approve` 写入回执。`reject` 或 `comment` 不产生下游可用的批准。

### dev

输入恰好一个仓库 key。核对 K7、K8、K10 和功能分支。启动绑定中的该仓开发命令。`--plan` 只打印将要执行的命令。

### coding 工具

| 工具 | 效果 |
| --- | --- |
| read_repo | 读取允许仓库内的已提交或工作区文件。拒绝环境文件 |
| write_file | 满足编码运行时的全部写入条件才落盘 |
| run_verification | 只执行该仓已配置的验证 |
| list_changes | 返回当前功能分支相对基线的路径和状态 |

不提供通用 Shell。

### test-report

把最近一次验证的退出码、范围、指纹写入 `test-report.md` 和事件。不自行把失败改成通过。

### complete / abandon

`complete` 要求验收记录有效，且工作区状态与交付说明一致。`abandon` 要求取消确认已经完成（Web 的两次确认对应一次带标记的 abandon 调用）。两者都释放锁，并在没有其他活动任务时把 `.active.md` 写成 none。

`complete` 的角色是 operator 或 admin。提出人可以在 Web 上发起验收，由 Kernel 以验收事务记录；若实例把验收和 complete 分成两步，配置里写明，默认验收通过后才允许 complete。

### publish

无确认：只写计划。有确认：核对身份、保护分支策略、干净工作区、源码指纹，然后按环境 `mode` 执行 handoff 或 push。

### pause-agent / resume-agent

把锁持有者改为 human 或 host。resume 时工作区脏则拒绝。

## HTTP

```
POST /v1/session
POST /v1/messages                 对话。可附带 iterationId
GET  /v1/iterations/active
GET  /v1/iterations/:id
POST /v1/iterations/:id/confirmations
POST /v1/iterations/:id/acceptance
POST /v1/queue/:id/start
POST /v1/queue/:id/withdraw
POST /v1/iterations/:id/cancel    两次确认用同一个资源的两个步骤令牌
GET  /v1/events                   按序号续传
GET  /v1/health
GET  /v1/ready
```

`/v1/messages` 不接收 Git 参数。取消的第一步返回后果说明和一次性步骤令牌；第二步提交令牌和复述结果。

错误体包含稳定的 `code` 与 `requestId`。K1 的错误码固定为 `active_iteration`，并带当前迭代摘要。

## 阶段进入条件

进入某阶段的函数只读文件：

- 需求版本等于本阶段声明所依据的版本
- 前序必经阶段具有有效回执，或按规则跳过且跳过原因已记录
- 若对齐开启，当前产物的对齐状态为 passed 且指纹新鲜
- 开发额外要求：实施方案回执有效、分支与基线匹配、执行器就绪或已显式改为人工接管

任何「页面上点了下一步」都先经过这个函数。

## 并发

同一 Host 上，队列启动、确认、写入、验收共用一把工作区锁。重复的确认和入队请求若版本相同，返回第一次的结果。

## 测试夹具

闸门测试使用临时目录和临时 Git 仓库。不读取实例的 `.env`，不连接真实模型也可以测 Kernel。模型行为另有可选的冒烟测试，且不得写入真实业务分支。
