# 2:50–3:00 复盘与验收

不需要逐字稿，这是开放问答，但要按顺序问下面这些问题（对应整堂课讲过的内容，按讲课顺序问，方便学员顺着回忆）：

1. Spec、测试和实现之间的关系是什么？（对应 `docs/lesson/02-writing-spec.md`、`03-ai-implementation.md`）
2. 为什么本地 `ruff`/`pytest` 都过了，还需要 CI 再跑一遍？（对应 `05-basic-ci.md`）
3. `lint` 和 `test` 为什么可以并行，`build` 为什么要等它们都过？（对应 `04-ci-cd-concepts.md`、`05`、`06`）
4. 为什么要验证打包出来的 Wheel，而不是只验证源码？（对应 `06-build-and-smoke-test.md`）
5. Workflow Artifact 和 GitHub Release 有什么区别？（对应 `07-release.md`）
6. Pull Request、`main`、版本 Tag 这三种事件分别触发流水线的哪些部分？（对应 `05`、`07` 里 `if` 条件那部分）
7. 本课程做到了 CI/CD 里的哪一部分，哪一部分是讲了但没做的？（对应 `04-ci-cd-concepts.md` 里 CI/持续交付/持续部署的边界）

## 检查

不需要每个人都答完全部 7 条，抽 2-3 个学员各答一两条即可，重点是听他们能不能用自己的话说清楚，而不是背出定义。

## 收尾

提醒学员：仓库里的 `docs/homework.md` 是这次课程的任务清单和验收标准，`docs/teacher-prep.md` 附录（如果之后开放）里有可选拓展任务，不计入本次考核。
