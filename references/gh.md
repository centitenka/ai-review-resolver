# GitHub (gh) 命令参考

Review thread 的列取、回复、resolve 都必须走 GraphQL 或 REST，`gh pr view --comments` 拿不到 thread ID，无法用于本流程。

## 前置变量

```bash
OWNER=$(gh repo view --json owner --jq '.owner.login')
REPO=$(gh repo view --json name --jq '.name')
PR=$(gh pr view --json number --jq '.number')
```

## 1. 列出未解决的 review thread

```bash
gh api graphql -f owner="$OWNER" -f repo="$REPO" -F pr="$PR" -f query='
  query($owner: String!, $repo: String!, $pr: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 100) {
          nodes {
            id
            isResolved
            isOutdated
            path
            line
            comments(first: 50) {
              nodes {
                databaseId
                author { login __typename }
                body
                url
              }
            }
          }
        }
      }
    }
  }' --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

- `id`（形如 `PRRT_...`）用于 resolve。
- 首条 comment 的 `databaseId` 用于回复。
- `author.__typename == "Bot"` 或 login 匹配 known-bots 清单 → 判定为非人类 reviewer。
- thread 超过 100 条时需要用 `reviewThreads(first: 100, after: <cursor>)` 翻页。

## 2. 在 thread 内回复

回复目标是 thread 首条 comment 的 `databaseId`：

```bash
gh api "repos/$OWNER/$REPO/pulls/$PR/comments/<databaseId>/replies" \
  -f body='已在 commit a1b2c3d 调整：<具体改动点>。'
```

## 3. Resolve thread

```bash
gh api graphql -f threadId='<PRRT_xxx>' -f query='
  mutation($threadId: ID!) {
    resolveReviewThread(input: {threadId: $threadId}) {
      thread { id isResolved }
    }
  }'
```

## 4. Push 时机

回复中引用的 commit SHA 必须先存在于远端：

```bash
git push origin HEAD   # 先 push
# 然后再执行第 2、3 步的回复与 resolve
```
