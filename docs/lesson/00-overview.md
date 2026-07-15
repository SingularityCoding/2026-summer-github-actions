# 教案总览

这是教师上课用的分阶段教案，对应 `docs/course-design.md` 第七节的时间表。每个阶段一个文件，越靠后的阶段（尤其是 GitHub Actions 部分）写得越详细——前面的阶段假设老师本来就熟，后面的阶段是这门课的重头戏，按逐字稿写，避免讲解顺序出错或漏讲。

参考实现（完整走过一遍学生流程得到的真实产出，包括实际能跑通的 `pipeline.yml`、真实的 CI 失败日志、真实的 Release）在：<https://github.com/ukeSJTU/2026-summer-github-actions-reference>。每个阶段文件末尾都标了对应的 commit/PR，卡住了可以直接去这个仓库对照。

| 时间 | 阶段 | 教案 | 详细程度 |
|---|---|---|---|
| 0:00–0:15 | 项目基线 | [01-project-baseline.md](./01-project-baseline.md) | 提纲 |
| 0:15–0:35 | 编写功能 Spec | [02-writing-spec.md](./02-writing-spec.md) | 含固定演示 Prompt |
| 0:35–1:05 | AI 辅助测试与实现 | [03-ai-implementation.md](./03-ai-implementation.md) | 含固定演示 Prompt |
| 1:05–1:15 | 休息 | — | — |
| 1:15–1:30 | CI/CD 与 GitHub Actions 核心模型 | [04-ci-cd-concepts.md](./04-ci-cd-concepts.md) | 讲解顺序 |
| 1:30–2:05 | 建立基础 CI | [05-basic-ci.md](./05-basic-ci.md) | 逐字稿 |
| 2:05–2:30 | 构建和验证制品 | [06-build-and-smoke-test.md](./06-build-and-smoke-test.md) | 逐字稿 |
| 2:30–2:50 | 持续交付与 Release | [07-release.md](./07-release.md) | 逐字稿 |
| 2:50–3:00 | 复盘与验收 | [08-wrap-up.md](./08-wrap-up.md) | 问题清单 |

学员任务清单和验收标准见 `docs/homework.md`，不在这份教案里重复。
