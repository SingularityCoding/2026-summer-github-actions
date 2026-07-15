# 1:30–2:05 建立基础 CI（逐字稿）

这是"重头戏"的第一段，按下面的顺序逐步做，不要跳步、不要合并步骤。每个"老师说"是可以直接照读的讲解词，"老师做"是要敲的命令/要点开的页面。

## 第 1 步：创建 `pipeline.yml`（约 5 分钟）

**老师说：**

> "我们现在把刚才讲的 lint、test 两个 Job 写成一个真正的 Workflow 文件。GitHub Actions 的 Workflow 文件必须放在 `.github/workflows/` 目录下，文件名不重要，但这个仓库约定叫 `pipeline.yml`。"

**老师做：** 打开（或替换）`.github/workflows/pipeline.yml`，逐块讲解着写：

```yaml
name: Pipeline

on:
  pull_request:
  push:
    branches:
      - main
    tags:
      - "v*"
  workflow_dispatch:
```

**老师说：**

> "`on` 这一块就是刚才讲的 Event：PR 创建或更新会触发一次；push 到 `main` 会触发一次；push 一个 `v` 开头的 tag 会触发一次——这个待会儿发 Release 要用；`workflow_dispatch` 是允许在 Actions 页面手动点一个按钮触发，方便调试。"

**老师做：** 接着写两个 Job：

```yaml
jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
        with:
          enable-cache: true
      - run: uv sync --locked
      - run: uv run ruff check .

  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
        with:
          enable-cache: true
      - run: uv sync --locked
      - run: uv run pytest --cov=todo_api
```

**老师说：**

> "两个 Job 目前互相不认识对方，天然并行——这就是刚才说的 lint 和 test 为什么能并行的原因，没有 `needs`，GitHub 会尽量同时跑。`runs-on: ubuntu-latest` 就是刚才说的 Runner。每个 Job 里的每一项都是一个 Step；`uses:` 开头的是调用别人写好的 Action（`actions/checkout` 负责把代码拉下来，`astral-sh/setup-uv` 负责把 `uv` 装到这台干净的虚拟机上），`run:` 开头的是直接跑一条 shell 命令——你们本地跑过的 `uv sync --locked`、`uv run ruff check .`、`uv run pytest --cov=todo_api`，原封不动搬过来。"

## 第 2 步：提交、推送、开 PR（约 5 分钟）

**老师做：**

```bash
git add .github/workflows/pipeline.yml
git commit -m "ci: add lint and test workflow"
git push -u origin feature/completed-filter
gh pr create --title "Add completed filter for GET /todos" --fill
```

**老师说：**

> "推送之后打开 GitHub 上的 Pull Request 页面，往下翻能看到 Checks 这一块——这就是刚才配置的 Workflow 真的跑起来了。"

## 第 3 步：观察一次失败（约 10 分钟，这是本段的核心）

老师这里**不需要额外制造 bug**——如果按上面的步骤走到这里，多半 Ruff 或 Pytest 已经在真实环境里第一次跑，最容易暴露的是"本地环境和 Runner 环境不完全一致"或者一个没注意到的 lint 问题。如果这次运气好全绿，就直接现场引入一个真实的小问题来补这一课（不要跳过这一步，"看一次失败"是本段的检查点）：

**老师做：** 在 `src/todo_api/routers/todos.py` 顶部的 import 里随手加一个没用到的名字，比如把

```python
from fastapi import APIRouter, Depends, Query, Request, status
```

改成

```python
from fastapi import APIRouter, Depends, HTTPException, Query, Request, status
```

不用 `HTTPException`，提交推送。

**老师说（等 Actions 转圈的时候）：**

> "现在故意留了一个没用到的 import。本地大家养成习惯是先跑 `uv run ruff check .` 再推送，但假设这次忘了——CI 就是这道最后防线：不管你有没有在本地跑过检查，PR 上的 Checks 会替你跑一遍，跑不过就不该合并。"

**老师做：** 等 Checks 出现红叉，点进失败的 Lint Job，翻到日志：

```text
F401 [*] `fastapi.HTTPException` imported but unused
 --> src/todo_api/routers/todos.py:5:41
  |
3 | from typing import Annotated, cast
4 |
5 | from fastapi import APIRouter, Depends, HTTPException, Query, Request, status
  |                                         ^^^^^^^^^^^^^
6 |
7 | from todo_api.models import Todo, TodoCreate
  |
help: Remove unused import: `fastapi.HTTPException`

Found 1 error.
[*] 1 fixable with the `--fix` option.
##[error]Process completed with exit code 1.
```

**老师说：**

> "这段日志和你们本地跑 `ruff check` 看到的一模一样——CI 失败不是什么新的、更可怕的东西，就是同一条命令换了台机器跑。日志会直接告诉你错误代码（`F401`）、文件、行号、甚至给出 `help` 提示。"

**老师做：** 改回来（去掉这行 import），提交，推送：

```bash
git add src/todo_api/routers/todos.py
git commit -m "fix: remove unused import caught by ruff"
git push
```

**老师说：**

> "推上去之后 CI 会自动重新跑这个 PR 的 Checks，不需要手动重新触发。等两个都变绿，这个 PR 就具备合并条件了。"

## 第 4 步：合并（约 2 分钟）

**老师做：** 确认 Lint、Test 都是绿的，`gh pr merge --merge --delete-branch` 或者在网页上点 Merge。

**老师说：**

> "合并到 `main` 也会再触发一次同样的 Workflow——这是为了保证 `main` 分支本身随时是绿的，不依赖"PR 上测过就一定没问题"这个假设。"

## 学员现在应该有的东西

- `.github/workflows/pipeline.yml`，包含 `lint` 和 `test` 两个 Job
- 一个已合并的 PR
- 至少一次真实的失败记录 + 修复记录（可以在 PR 的 Checks 历史里点开看到）

## 常见卡点

- Runner 上 `uv run ruff check .` 报错但本地是干净的：多半是本地忘了保存文件就跑了检查，或者本地虚拟环境里装了本地才有的包。提示学员对照日志逐行看，不要瞎猜。
- `gh pr create --fill` 报错找不到默认分支/远程：确认 `git push -u` 那一步真的执行过，分支已经推到远程。
- 学员误以为要手动去 Actions 页面点"重新运行"：强调 push 新 commit 会自动重跑对应 PR 的 Checks，不需要手动点。
