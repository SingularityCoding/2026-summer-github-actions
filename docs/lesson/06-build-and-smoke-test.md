# 2:05–2:30 构建和验证制品（逐字稿）

上一段结束时已经把 `feature/completed-filter` 的 PR 合并了（05 第 4 步）。这一段是**新的一个 PR**，不是接着改刚才那个已经合并、已经删掉的分支——开场先建分支、开 PR，再继续编辑 `pipeline.yml`，不要跳过这一步。

## 第 0 步：新建分支、开新 PR（约 2 分钟）

**老师做：**

```bash
git switch main
git pull
git switch -c ci/build-smoke-test-release
```

**老师说：**

> "刚才那个功能分支已经合并删掉了，现在从最新的 `main` 开一个新分支，专门给这一段的改动用。等下这段写完、验证过，会开一个新的 PR，跟刚才写 Spec/测试/实现的那个 PR 是两个独立的 PR。"

## 第 1 步：讲清楚为什么要单独一个 build 步骤（约 3 分钟）

**老师说：**

> "lint 和 test 只验证了源码本身。但学员/用户实际安装、使用的是打包出来的 Wheel，不是这份源码目录——源码能跑测试，不代表打包出来的东西一定装得上、跑得起来。所以我们要单独构建一次，并且拿构建出来的东西去验证，而不是偷懒继续测源码。"

## 第 2 步：加 `build` Job（约 8 分钟）

**老师做：** 在 `test` Job 后面追加：

```yaml
  build:
    name: Build
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
        with:
          enable-cache: true
      - run: uv build
      - uses: actions/upload-artifact@v4
        with:
          name: todo-api-package
          path: dist/
```

**老师说：**

> "`needs: [lint, test]` 是刚才没用过的依赖声明——build 必须等 lint 和 test 都成功才开始，没必要在一份已知有问题的代码上浪费时间构建。待会儿 smoke-test 和 release 还会各自用一次 `needs`，这是本课程唯一用来控制 Job 顺序的写法。`uv build` 就是你们本地跑过的那条命令，会在 `dist/` 下生成 `.whl` 和 `.tar.gz`。最后这个 `actions/upload-artifact` 步骤，是把 `dist/` 整个目录打包成一个叫 `todo-api-package` 的 Workflow Artifact——这个名字待会儿 smoke-test 和 release 两个 Job 都要用同一个名字去下载，记好了，写错名字下载不到东西。"

## 第 3 步：加 `smoke-test` Job（约 12 分钟，本段核心）

**老师做：**

```yaml
  smoke-test:
    name: Smoke test
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: todo-api-package
          path: dist/
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - name: Install only the built wheel into a clean venv
        run: |
          python -m venv /tmp/smoke-venv
          /tmp/smoke-venv/bin/pip install dist/*.whl
      - name: Start the installed CLI and check /health
        run: |
          /tmp/smoke-venv/bin/todo-api &
          for i in $(seq 1 20); do
            if curl -sf http://127.0.0.1:8000/health; then
              exit 0
            fi
            sleep 0.5
          done
          echo "todo-api did not become healthy in time" >&2
          exit 1
```

**老师说（边写边讲，这是全课程最容易讲飞的地方，按下面几句话的顺序讲，不要跳）：**

> "第一步 `download-artifact`，把刚才 build 产出的东西下载下来——注意这是一个全新的 Job，全新的虚拟机，**不会**继承 build Job 里已经装好的东西，包括源码本身都没有，这就是为什么第一步必须专门下载。"
>
> "第二步装一个干净的 Python 3.12，跟这个仓库的源码环境完全没关系。"
>
> "第三步是全场重点：新建一个全新的虚拟环境，**只装刚下载下来的 `.whl` 文件**，不装 `-e .`，不装源码目录，不装 dev 依赖。这样如果测试用的代码路径依赖了什么源码目录里有、但没打进 Wheel 里的东西，这一步就会装不上或者装上了缺东西，能提前暴露"打包漏了文件"这类问题。"
>
> "第四步启动服务：装完之后应该有一个 `todo-api` 命令可以直接用——这是 `pyproject.toml` 里 `[project.scripts]` 那一行配置出来的入口命令。这里没有用固定的 `sleep 2` 就去 curl，而是写了个循环，最多等 10 秒，每 0.5 秒 curl 一次 `/health`，因为不同 Runner 启动服务的速度不完全一样，写死等待时间早晚会在某次运行里偶然失败。请求成功就退出成功；等满还没成功就打印错误退出失败。"

## 第 4 步：顺手把 `release` Job 也加上（约 3 分钟，概念留到下一段讲）

**老师做：** 在 `smoke-test` 后面追加：

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

> "这个 Job 现在先加上，具体怎么用、`if` 那行什么意思、为什么单独给了写权限，放到下一段（持续交付与 Release）再讲——现在只要知道：这次推送和这次 PR 都不会真的触发它，Actions 页面上会显示这个 Job 是 skipped，先不用管。"

## 第 5 步：提交、推送、开 PR、观察运行结果（约 5 分钟）

**老师做：**

```bash
git add .github/workflows/pipeline.yml
git commit -m "ci: add build, smoke-test, and release jobs"
git push -u origin ci/build-smoke-test-release
gh pr create --title "Add build, smoke-test, and release jobs" --fill
```

打开这个新 PR 的 Checks，看 `build` → `smoke-test` 按顺序跑完（`release` 这个 Job 会显示 skipped——上一步已经讲过，正常）。点开 `smoke-test` 的日志，指给学员看 `{"status":"ok"}` 这行输出。全部通过后合并这个 PR：

```bash
gh pr merge --merge --delete-branch
git switch main
git pull
```

**老师说：**

> "现在验证的已经不是源码能不能跑测试，而是——一个陌生人，拿到你发布出去的这个包，装上、敲一个命令、能不能真的用。这才是"能发布"这件事真正要验证的东西。"

## 学员现在应该有的东西

- `.github/workflows/pipeline.yml` 增加了 `build`、`smoke-test`、`release` 三个 Job（`release` 目前还没真的跑过，下一段会触发它）
- 一个叫 `todo-api-package` 的 Workflow Artifact（在 Actions 运行详情页可以下载）
- 一次成功的 Smoke Test 记录（日志里能看到 `/health` 返回 `{"status":"ok"}`）
- 这个 PR 已合并到 `main`——下一段（07）不再需要新开分支/PR，直接在 `main` 上打 tag

## 常见卡点

- `dist/*.whl` 展开失败/装错文件：确认 `download-artifact` 那步的 `name:` 和 `build` Job 里 `upload-artifact` 的 `name:` 完全一致（`todo-api-package`）。
- `todo-api: command not found`：多半是装到了错误的虚拟环境（比如又不小心用了系统 Python 的 pip，而不是 `/tmp/smoke-venv/bin/pip`），检查装/跑用的是不是同一个 venv 路径。
- Smoke Test 偶尔超时：先看是不是把固定 `sleep` 换成了轮询 `curl`，这是这一段最容易被学员简化掉、然后在某次运行里随机失败的地方。
