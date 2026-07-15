# 1:15–1:30 CI/CD 与 GitHub Actions 核心模型

放在这里讲（而不是开场前 15 分钟），是因为这时候学员手上已经有一个写完、测试通过的功能分支——概念一讲完立刻要用，记忆点更实。不是逐字稿，但按下面的顺序讲，不要打乱，因为后面几段（建立 CI、构建制品、Release）是按这个顺序把概念对应到具体 Job 上的。

## 讲解顺序

1. **手工流程的问题**：现在这份分支只在你自己电脑上跑过 ruff/pytest，没有任何人/任何机器验证过它在"干净环境"里也是这样。合并前谁来保证？——这是引出 CI 的钩子。
2. **CI / 持续交付 / 持续部署 三者的区别**（一句话版本，不用展开定义）：
   - CI：每次改动自动跑检查（lint/test/build），保证"可重复"。
   - 持续交付：验证通过后自动打包好、随时能发布，但发布动作（打 tag）还是人决定。
   - 持续部署：发布后自动上线到长期运行环境——本课程做到"持续交付"为止，不做真正部署到服务器，讲清楚这条边界，别让学员以为课程漏了一步。
3. **GitHub Actions 的定位**：GitHub 自带的自动化平台，触发条件是仓库里的事件（PR、push、打 tag 等）。
4. **五个核心概念，按包含关系讲**（Event 触发 Workflow，Workflow 里有多个 Job，Job 里有多个 Step，Step 可能调用一个 Action，Job 在 Runner 上执行）：
   - Event：PR 创建/更新、push 到 main、push `v*` tag、手动触发
   - Workflow：一个 `.yml` 文件，定义"发生这些 Event 时要做什么"
   - Job：Workflow 里的一个执行单元，可以并行也可以用 `needs` 声明依赖顺序
   - Step：Job 里的一步，可以是一行 shell 命令，也可以是调用一个 Action
   - Runner：真正执行 Job 的虚拟机（`ubuntu-latest`）
   - Action：别人（或 GitHub 官方）打包好的可复用 Step，比如 `actions/checkout`
5. **接下来要建的完整流水线**（画出来或口头过一遍，对应 `docs/lesson/05-basic-ci.md` 到 `07-release.md`）：

   ```text
   lint ─────┐
             ├→ build → smoke-test → release（仅版本 Tag 触发）
   test ─────┘
   ```

   讲清楚为什么是这个形状：lint 和 test 互不依赖，可以并行，越早发现越省时间；build 必须等两者都过了才做，没意义在有已知问题的代码上浪费时间构建；smoke-test 必须在 build 产出的 wheel 上验证，而不是验证源码目录；release 只在打版本 tag 时才跑，而且必须建立在 smoke-test 通过的基础上。

## 检查点

学员能口头说出接下来要经历哪几个 Job、大致顺序，不需要能默写 YAML。
