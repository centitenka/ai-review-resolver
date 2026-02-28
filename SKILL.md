---
name: ai-review-resolver
description: 处理当前 GitHub PR 中非人类 reviewer（Copilot、Gemini、CodeRabbit 等）的 code review 闭环。用于“逐条处理 AI review”“按 review 单独提交并 resolve”“用 review 原语言回复处理结果”等场景。先通读全部 review 建立整体处理顺序，再逐条以工程判断评估是否采纳：采纳则改动并提交，不采纳则说明原因后直接 resolve。回复必须可追溯到具体 commit（short SHA）。
---

# AI Review Resolver

## Workflow

1. 收集当前 PR 的 review 对话，仅保留非人类 reviewer（bot/AI 工具账号），忽略人类 reviewer。
2. 先完整阅读所有未解决对话，按文件和逻辑依赖分组，规划处理顺序。
3. 按顺序逐条处理每个 review：
   - 先基于项目目标、现有架构和风险进行工程判断，决定是否采纳。
   - 若采纳：仅实现当前 review 对应改动，运行最小验证，立即提交一次 commit（一个 review 对应一个 commit），再用同语种精简 comment 说明“做了什么改动 + 在哪个 commit”，然后 resolve。
   - 若不采纳：不改代码，在该 review 下用同语种精简 comment 说明“不跟进原因”，然后直接 resolve。
4. 重复直到所有非人类 review 处理完成。

## Rules

- 需要代码改动时，强制一条 review 一次 commit，不合并多条 review 到同一 commit。
- 若多条 review 存在依赖，先处理根因再处理依赖项，但仍保持逐条 commit。
- comment 保持 1-2 句，必须有信息密度，不写空话或流程汇报腔。
- 语言跟随 review：英文 review 用英文回复，中文 review 用中文回复；混合时跟随主语言。
- AI review 可能不准确，必须进行独立判断；不采纳时给出简短工程理由后可直接 resolve。
- 无法完整处理时，用同语种简述阻塞原因，并暂不 resolve。
- 采纳项 comment 必须包含 `commit <short_sha>`（例如 `commit a1b2c3d`）和一条具体改动点。
- 禁用表述：`已按建议调整`、`已在本次独立提交完成`、`已处理` 这类不可追溯句式。

## Comment Pattern

- 采纳项（中文）：`已在 commit <short_sha> 调整：<具体改动点>。`
- 采纳项（英文）：`Fixed in commit <short_sha>: <specific change>.`
- 不采纳（中文）：`这条先不跟进：<工程原因>。`
- 不采纳（英文）：`Not applying this suggestion for now: <engineering reason>.`

## Done

- 非人类未解决 review 均已：
  - 已采纳项有对应 commit；
  - 未采纳项有同语种不跟进原因；
  - 有同语种精简 comment（采纳项含 short SHA）；
  - 已 resolve（或明确记录阻塞并保留未 resolve）。
