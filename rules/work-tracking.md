# 工作流文档追踪

## 共享文档结构

所有 agent 共享 `work_doc/` 目录下的文档，用于跨会话、跨 agent 的任务追踪：

| 文档 | 用途 | 写入规则 |
|------|------|---------|
| `previous_plan.md` | 历史计划记录 | 仅 append |
| `previous_done.md` | 已完成任务记录 | 仅 append |
| `current_doing.md` | 当前进行中任务 | 仅 append + 可打勾 |
| `latest_detail_plan.md` | 最新详细工作计划 | 可覆写（需人工审批） |

如果 `work_doc/` 目录或其中的文件不存在，在任务开始前自动创建。

### 记录格式

每条记录使用以下格式，便于追溯来源：

```markdown
## [YYYY-MM-DD HH:MM] agent_name (model)
- 任务描述 bullet point 1
- 任务描述 bullet point 2
```

`current_doing.md` 中使用 checkbox 追踪进度：

```markdown
## [YYYY-MM-DD HH:MM] agent_name (model)
- [ ] 待完成的任务项
- [x] 已完成的任务项
```

## 任务前流程（Pre-task）

每次正式任务开始前，按以下顺序执行：

1. **读取上下文** — 读取 `work_doc/` 下所有文档，了解历史计划、已完成内容和当前进展
2. **制定计划** — 基于上下文和用户需求，制定本次任务的 plan
3. **提交审批** — 将 plan 返回给用户，等待人工批准后再开始执行
4. **更新文档** — 审批通过后：
   - 将详细计划写入 `latest_detail_plan.md`（覆写）
   - 将计划摘要 append 到 `previous_plan.md`
   - 将待办事项 append 到 `current_doing.md`（使用 `- [ ]` 格式）

## 暂停审批机制

以下情况必须**暂停执行**，请求用户批准，并提供手动操作指引：

| 触发条件 | 要求 |
|----------|------|
| 需要联网安装 library（npm install, pip install 等） | 暂停，告知包名和安装命令，等待批准 |
| 需要更改项目外的系统环境（全局配置、环境变量等） | 暂停，说明变更内容和手动步骤，等待批准 |
| 需要修改 `latest_detail_plan.md` | 暂停，展示修改前后的 diff，等待批准后更新 |

暂停时的输出格式：

```
⏸️ 需要人工批准

原因：[具体原因]
操作：[将要执行的命令或变更]
手动方式：[用户自行操作的步骤]

请批准后继续。
```

## 进度追踪

任务执行过程中持续更新文档：

- **完成一项任务** → 在 `current_doing.md` 中将对应 `- [ ]` 改为 `- [x]`
- **阶段性完成** → 将完成记录 append 到 `previous_done.md`
- **全部完成** → 确保 `previous_done.md` 包含本次所有完成项的最终记录
