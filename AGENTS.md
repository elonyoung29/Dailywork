# AGENTS.md

## Language

- 默认使用中文回答。
- 回答先给结论，再给必要说明。
- 保持简洁，除非用户要求详细展开。

## Memory

- 当前目录的长期记忆文件是：`MEMORY.md`。
- 新会话或上下文不足时，先读取 `MEMORY.md` 恢复用户偏好和项目背景。
- 涉及京东外卖超级月卡、品牌饭卡、代我充、数据口径、PRD、JoySpace 时，优先参考 `MEMORY.md` 中列出的原始 Claude memory 文件。

## PRD Rules

- 写 PRD 必须严格按 JoySpace 上「yyl的PRD模板」结构输出。
- 模板 pageId：`2R2xg2rwlhD0PgQFIMY1`。
- 不改变章节顺序，不自创结构，不省略表格列。
- 信息不足处使用 `[待补充]` 占位，但保留完整结构。

## JoySpace Rules

- PlantUML 必须使用 `plantuml` Markdown 代码块，不转 PNG。
- 梳理文档时先本地改完，最后用户确认或明确要求后再统一发布 JoySpace。

## Workspace Rules

- 本项目less线程的临时文件、草稿、生成物默认放在当前目录。
- 不主动写入用户 home 目录，除非用户明确要求。
