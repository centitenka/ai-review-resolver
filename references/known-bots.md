# 常见 AI Reviewer 账号清单

判定顺序：

1. **平台标记**：GitHub GraphQL `author.__typename == "Bot"`，或 login 以 `[bot]` 结尾。
2. **账号清单**：命中下表即判定为 AI reviewer。
3. **默认人类**：未命中以上两条的账号一律视为人类 reviewer，不处理。人类账号手动运行 AI 工具发出的 review 也按人类处理，除非用户明确指示。

## GitHub

| 账号 | 工具 |
|------|------|
| `copilot-pull-request-reviewer[bot]`（显示名 Copilot） | GitHub Copilot code review |
| `gemini-code-assist[bot]` | Google Gemini Code Assist |
| `coderabbitai[bot]` | CodeRabbit |
| `qodo-merge-pro[bot]` / `codiumai-pr-agent-pro[bot]` | Qodo Merge（前 Codium PR-Agent） |
| `cursor[bot]` | Cursor Bugbot |
| `sourcery-ai[bot]` | Sourcery |
| `greptile-apps[bot]` | Greptile |
| `ellipsis-dev[bot]` | Ellipsis |
| `sweep-ai[bot]` | Sweep |
| `devin-ai-integration[bot]` | Devin |
| `graphite-app[bot]` | Graphite Diamond |
| `sonarqubecloud[bot]` | SonarQube Cloud（静态扫描类） |
| `deepsource[bot]` | DeepSource（静态扫描类） |

## GitLab

| 账号 | 工具 |
|------|------|
| `GitLabDuo` / `gitlab_duo` | GitLab Duo Code Review |
| username 含 `_bot_`（如 `project_123_bot_abc`） | 项目/群组 service account，多为自建自动化 |

企业自建 GitLab 上的自动化账号形态各异，遇到未收录的稳定出现的 AI reviewer 账号时，把它加进本清单。
