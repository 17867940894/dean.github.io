
可以把系统拆成四层：

|层|记录什么|适合的工具|
|---|---|---|
|知识层|概念、规范、经验、背景|Obsidian|
|项目层|项目目标、里程碑、任务、风险、进度|数据库/结构化文件|
|上下文层|当前对话、近期决策、工作记忆|Agent context|
|执行层|Agent 实际执行、提交、测试、产物|Git、工单系统、脚本|

关键原则是：

> Obsidian 负责“是什么”，项目状态库负责“现在做到哪了”，Agent 上下文只是缓存，不是事实来源。

## 1. 实际工作中如何管理多个项目

每个项目应有一个唯一的 `project_id`，并且使用统一结构：

```javascript
项目
├── 项目目标
├── 当前阶段
├── 里程碑
├── 任务列表
├── 风险与阻塞
├── 决策记录
├── 交付物
└── 活动日志
```

例如：

```javascript
project_id: CRM-2026
name: CRM系统升级
objective: 完成客户管理系统重构
status: active
phase: development
health: yellow
owner: alice
deadline: 2026-09-30
milestones:
  - id: M1
    name: 需求确认
    status: done
  - id: M2
    name: 核心开发
    status: in_progress
progress: 42
next_actions:
  - 完成客户导入接口
  - 等待产品确认字段定义
blockers:
  - 产品字段定义尚未确认
updated_at: 2026-08-03T10:00:00+08:00
version: 18
```

多个项目的管理，本质上不是让 Agent 读完所有笔记，而是让它先回答：

```javascript
当前有哪些项目？
每个项目的状态是什么？
哪些任务正在进行？
哪些任务被阻塞？
今天最重要的下一步是什么？
```

因此需要有一个“项目总览”：

```javascript
项目总览
- CRM系统升级：进行中，42%，黄色，有1个阻塞
- 官网改版：进行中，75%，绿色
- 数据平台迁移：暂停，18%，红色
```

Agent 每次工作前只需要读取项目总览和目标项目的当前状态，而不是重新扫描整个知识库。

## 2. 如何让 Agent 知道“做到哪里了”

不能只依靠对话历史。对话会截断、丢失、分叉，也可能被不同 Agent 读取。

每个任务至少应包含这些字段：

```javascript
task_id: CRM-142
project_id: CRM-2026
title: 完成客户导入接口
status: in_progress
assignee: agent-backend
priority: high
depends_on:
  - CRM-131
acceptance_criteria:
  - 支持 CSV 导入
  - 重复客户能够识别
  - 单元测试通过
  - API 文档已更新
deliverables:
  - src/import_customer.ts
  - tests/import_customer.test.ts
progress_note: 已完成解析和重复检测，正在补充异常处理
next_action: 增加导入失败回滚
updated_at: 2026-08-03T09:30:00+08:00
version: 7
```

这里有三个重要区别：

- `status`：任务处于什么阶段
- `progress_note`：已经做了什么
- `next_action`：下一步做什么

建议任务状态不要过于复杂：

```javascript
backlog
ready
in_progress
blocked
submitted
in_review
done
cancelled
```

尤其不要让 Agent 仅凭一句“完成了”就把任务标记为 `done`。

更可靠的流程是：

```javascript
ready
  ↓
in_progress
  ↓
submitted
  ↓
in_review
  ├── 通过 → done
  └── 不通过 → in_progress
```

`submitted` 表示 Agent 声称完成，并提交了证据；`done` 表示验收通过。

## 3. 多个项目如何下发任务，以及完成后如何更新状态

建议把“任务下发”和“状态修改”变成标准协议，而不是依赖自然语言约定。

### 任务下发

人只需要指定：

```javascript
项目：CRM系统升级
任务：完成客户导入接口
验收标准：支持CSV、去重、失败回滚、测试通过
截止时间：本周五
```

系统或 Agent 将其转换成结构化���务：

```javascript
{
  "task_id": "CRM-142",
  "project_id": "CRM-2026",
  "status": "ready",
  "assignee": "agent-backend",
  "acceptance_criteria": [
    "支持CSV导入",
    "识别重复客户",
    "失败时能够回滚",
    "测试通过"
  ]
}
```

Agent 开始任务前，必须读取：

1. 项目目标
2. 当前阶段
3. 任务详情
4. 依赖任务
5. 相关知识和决策
6. 最近的活动记录

### 任务完成提交

Agent 不应该直接写：

```javascript
status: done
```

而应该提交一个“完成报告”：

```javascript
{
  "task_id": "CRM-142",
  "new_status": "submitted",
  "summary": "已完成CSV解析、重复检测和失败回滚",
  "evidence": [
    "commit: a81f3c2",
    "test: 48 passed",
    "file: src/import_customer.ts",
    "document: docs/customer-import.md"
  ],
  "remaining_risks": [],
  "suggested_next_action": "进入代码审查"
}
```

然后由验收 Agent、项目负责人或自动化检查决定是否进入 `done`。

### 项目进度如何更新

不要让 Agent 手工随意填写项目进度。最好从任务和里程碑计算：

```javascript
项目进度 =
已完成任务权重 ÷ 全部任务权重
```

或者：

```javascript
里程碑进度 =
已完成验收项 ÷ 全部验收项
```

项目的健康状态也可以自动计算：

```javascript
绿色：无阻塞，关键路径正常
黄色：存在风险或延期可能
红色：关键路径阻塞，或已经延期
```

Agent 每次提交任务后，可以触发一个项目更新流程：

```javascript
任务提交
→ 验证产物
→ 更新任务状态
→ 重算里程碑进度
→ 重算项目总体进度
→ 识别新阻塞
→ 更新项目总览
→ 写入活动日志
```

活动日志很重要：

```javascript
{
  "event_id": "evt-8831",
  "type": "task_status_changed",
  "project_id": "CRM-2026",
  "task_id": "CRM-142",
  "from": "in_progress",
  "to": "submitted",
  "actor": "agent-backend",
  "evidence": ["a81f3c2"],
  "timestamp": "2026-08-03T10:00:00+08:00"
}
```

这样可以追溯：谁在什么时候因为什么修改了什么。

## 4. 多个人同步读写，没有本地上下文时如何保证一致性

核心做法是：

> Agent 不拥有项目数据，Agent 只能通过项目状态库读取和提交变更。

本地上下文丢失并不可怕，只要 Agent 能重新读取规范化状态。

### 使用唯一事实来源

可以选择：

- 数据库：适合多人、频繁更新
- Git + YAML/JSON：适合技术团队和需要审计
- Linear、Jira、Plane、Trello 等项目系统：适合直接使用成熟工单能力
- Obsidian：继续作为知识和文档层，但不承担并发项目状态

不建议让多人同时直接修改同一个 Markdown 项目文件，除非使用 Git 版本控制和明确的合并规则。

### 使用版本号防止覆盖

每条任务带有 `version`：

```javascript
{
  "task_id": "CRM-142",
  "status": "in_progress",
  "version": 7
}
```

Agent 修改时必须声明：

```javascript
我基于 version=7 修改
```

如果其他人已经修改到 version=8，系统拒绝这次更新：

```javascript
更新失败：数据已变化，请重新读取后再提交
```

这叫乐观并发控制，比简单地“最后写入者覆盖前一个人”安全得多。

### 用原子操作修改状态

不要让 Agent 直接整文件覆盖。提供有限的操作：

```javascript
claim_task(task_id)
update_progress(task_id, note)
submit_task(task_id, evidence)
approve_task(task_id)
block_task(task_id, reason)
add_dependency(task_id, dependency_id)
```

例如：

```javascript
{
  "operation": "submit_task",
  "task_id": "CRM-142",
  "expected_version": 7,
  "evidence": ["commit:a81f3c2", "test:48 passed"]
}
```

系统负责检查：

- 当前状态是否允许提交
- 当前版本是否匹配
- 验收证据是否存在
- 必填字段是否完整
- 是否有权限修改
- 修改后是否会产生非法依赖

### 保留事件日志

当前状态可以被修正，但事件日志不应该被静默覆盖。

这样即使 Agent 判断错误，也能看到：

```javascript
10:00 Agent A 将任务改为 submitted
10:03 Agent B 发现测试失败，退回 in_progress
10:20 Agent A 补充测试后再次提交
```

### 让 Agent 每次重新建立工作上下文

Agent 没有本地上下文时，启动流程应固定为：

```javascript
读取全局项目索引
→ 读取目标项目状态
→ 读取目标任务
→ 读取相关决策和依赖
→ 检查最新版本
→ 执行工作
→ 提交结构化结果
```

可以把它固化成系统提示词：

```javascript
你不是项目状态的所有者。
开始工作前必须读取项目和任务的最新状态。
不得根据记忆推断当前状态。
不得直接将任务标记为 done，必须提交验收证据。
所有状态修改必须带 expected_version。
如果版本冲突，重新读取后再操作。
```

## 一个适合你当前阶段的最小方案

你不必马上搭建复杂系统，可以先这样分工：

```javascript
Obsidian
├── 项目背景
├── 技术知识
├── 会议记录
├── 决策记录
└── 方案文档

项目状态库
├── projects.yaml
├── milestones.yaml
├── tasks.yaml
├── risks.yaml
└── events.jsonl
```

先把任务状态放到结构化文件或工单系统中，Obsidian 只保存链接和说明：

```javascript
# CRM系统升级

项目状态：[CRM-2026](项目系统链接)

当前状态：进行中，42%，黄色

最新阻塞：
- 产品字段定义尚未确认

相关知识：
- [[客户数据模型]]
- [[导入接口规范]]
```

Agent 的工作方式变成：

```javascript
查询项目状态
→ 领取任务
→ 执行任务
→ 提交证据
→ 更新任务
→ 自动刷新项目进度
```

最重要的转变是：

> 让对话负责表达意图，让项目系统负责保存状态，让 Agent 负责执行和提交证据，让验收机制负责确认完成。

这样即使换人、换 Agent、丢失对话上下文，项目仍然能够继续运行。
