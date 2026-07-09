# GitLab (glab) 命令参考

GitLab 中 review 对话对应 MR 的 discussion（可 resolve 的 thread）。`glab api` 的 `:id` 占位符会自动替换为当前项目。

## 前置变量

```bash
MR_IID=$(glab mr view --output json | jq -r '.iid')
```

## 1. 列出未解决的 discussion

```bash
glab api "projects/:id/merge_requests/$MR_IID/discussions" --paginate \
  | jq '[.[] | select(.notes[0].resolvable == true and .notes[0].resolved == false)]'
```

每个 discussion 关注：

- `.id` — discussion ID，用于回复和 resolve。
- `.notes[0].author.username` — 匹配 known-bots 清单判定是否为非人类 reviewer（GitLab 的 bot 账号 username 常含 `_bot`，如 `project_123_bot_abc`；自建工具账号需按清单维护）。
- `.notes[0].body` / `.notes[0].position` — review 内容与代码位置。

## 2. 在 discussion 内回复

```bash
glab api "projects/:id/merge_requests/$MR_IID/discussions/<discussion_id>/notes" \
  -X POST -f body='已在 commit a1b2c3d 调整：<具体改动点>。'
```

## 3. Resolve discussion

```bash
glab api "projects/:id/merge_requests/$MR_IID/discussions/<discussion_id>" \
  -X PUT -F resolved=true
```

## 4. Push 时机

回复中引用的 commit SHA 必须先存在于远端：

```bash
git push origin HEAD   # 先 push
# 然后再执行第 2、3 步的回复与 resolve
```
