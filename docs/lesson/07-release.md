# 2:30–2:50 持续交付与 Release（逐字稿）

最后一个 Job——但 YAML 已经在上一段（06 第 4 步）写好并合并进 `main` 了，这一段不用再编辑 `pipeline.yml`，直接从理解这个 Job 开始。学员现在应该在 `main` 分支，工作区干净（06 结束时已经合并删了分支）。

## 第 1 步：讲解已经加进去的 `release` Job（约 8 分钟）

**老师做：** 打开 `.github/workflows/pipeline.yml`，指着上一段已经写好的这一块：

```yaml
  release:
    name: Release
    needs: smoke-test
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: todo-api-package
          path: dist/
      - uses: softprops/action-gh-release@v2
        with:
          files: dist/*
          generate_release_notes: true
```

**老师说：**

> "`needs: smoke-test` 好理解——没通过冒烟测试的东西不该发布。关键是这一行 `if: startsWith(github.ref, 'refs/tags/v')`：前面 lint/test/build/smoke-test 四个 Job，每次 PR、每次 push 到 main 都会跑；但 release 这个 Job 只有在触发条件是"push 了一个 `v` 开头的 tag"时才会真的执行，其他时候这个 Job 会显示"skipped"，不是失败，是被跳过。"
>
> "然后是权限。整个 Workflow 默认给的 GITHUB_TOKEN 是只读权限——这是刚才建 lint/test/build/smoke-test 的时候都没提过的一个默认设定，因为那几个 Job 只是读代码、跑命令，不需要写权限。但创建 GitHub Release 需要往仓库里写东西，所以只有这一个 Job 单独声明 `permissions: contents: write`，问 GitHub 要额外的写权限——不是整个 Workflow 都开了写权限，是精确到这一个 Job。这是权限最小化原则：能不给的权限就不给，只在真正需要的地方开一个口子。"
>
> "最后 `softprops/action-gh-release` 这个 Action 负责真正创建 Release：把刚才下载下来的 `dist/*`（wheel 和 sdist 两个文件）挂到 Release 的 Assets 上，`generate_release_notes: true` 会自动根据这段时间合并的 PR 生成一份更新说明，不用手写。"

## 第 2 步：更新版本号（如果需要）（约 2 分钟）

**老师说：**

> "`pyproject.toml` 里的 `version = "0.1.0"` 和待会儿要打的 tag `v0.1.0` 应该对应上——这次版本号已经是 `0.1.0`，不用改；如果是后续迭代要发新版本，这里要先手动改版本号再打 tag。"

## 第 3 步：打 tag、推送（约 3 分钟，本段核心动作）

**老师做：**

```bash
git switch main
git pull
git tag v0.1.0
git push origin v0.1.0
```

**老师说：**

> "注意打 tag 和推 tag 是两个动作：`git tag` 只在本地建了这个标记，`git push origin v0.1.0` 才会把它推到 GitHub，触发 push 事件——GitHub Actions 是在收到"有一个 `v0.1.0` tag 被推上来了"这个事件之后才开始跑,不是你在本地建 tag 那一刻。"

## 第 4 步：观察五个 Job 全部跑完（约 5 分钟）

**老师做：** 打开 Actions 页面，指出这次触发的是同一个 `pipeline.yml`，但因为事件是"push tag"，五个 Job（lint/test/build/smoke-test/release）会全部执行，包括之前在 PR 和 push main 时被跳过的 `release`。

**老师说：**

> "这就是为什么整个流水线只写一份 `pipeline.yml`，而不是给"平时"和"发布"分别写两份——同一份定义，靠 `if` 条件决定这次触发要不要跑最后那个 Job。"

## 第 5 步：检查 GitHub Release（约 2 分钟）

**老师做：** 打开仓库的 Releases 页面，点开 `v0.1.0`，展示：
- 标题/tag 是 `v0.1.0`
- Assets 里有 `todo_api-0.1.0-py3-none-any.whl` 和 `todo_api-0.1.0.tar.gz`
- 自动生成的 Release Notes 里列出了合并过的 PR

**老师说：**

> "到这里，Artifact 和 Release 的区别可以点一下：Workflow Artifact（比如刚才那个 `todo-api-package`）是流水线内部传递东西用的，有效期有限、面向的是"这次运行"；GitHub Release 是面向所有人的正式发布记录，长期存在，这才是"我们发布了 0.1.0 版本"这件事真正对外的凭证。"

## 学员现在应该有的东西

- 完整的 `.github/workflows/pipeline.yml`（lint/test/build/smoke-test/release 五个 Job）
- `v0.1.0` Git Tag
- 一个带 `.whl` 和 `.tar.gz` 两个 Asset 的 GitHub Release

## 常见卡点

- Release Job 显示 skipped：先检查触发方式是不是真的推了 tag（`git push origin v0.1.0`），而不是只在本地 `git tag` 或者 push 了别的分支。
- Release 创建失败、报权限错误：检查 `permissions: contents: write` 是不是写在 `release` 这个 Job 底下（不是写在整个 Workflow 顶层，虽然写顶层也生效，但这里刻意示范"精确到 Job"的写法）。
- 学员把版本号和 tag 弄错（比如 `pyproject.toml` 是 `0.1.0` 但打了 `v0.2.0`）：这次课程不强制校验一致性（这是附录里的挑战任务之一），但可以顺嘴提一句"生产环境通常会加一步校验"。
