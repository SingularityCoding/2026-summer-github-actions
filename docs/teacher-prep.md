# 教师课前准备

## 模板仓库

本仓库（`SingularityCoding/2026-summer-github-actions`）本身即模板：在 GitHub 仓库设置中开启 Template Repository 开关后，学员通过 `gh repo create --template ...` 或网页上的 "Use this template" 创建自己的独立仓库（见 `docs/prerequisite.md`）。模板仓库包含：

- 可运行的 FastAPI Todo API
- 已有接口和测试
- `pyproject.toml`
- `uv.lock`
- `.python-version`
- Spec 目录和模板
- 空的或待补充的 `pipeline.yml`
- Pull Request 模板
- README 课堂指引
- `docs/homework.md` 中的可选拓展任务（非本次课程考核内容）

## 预埋教学故障

`docs/lesson/05-basic-ci.md` 第 3 步已经内置了一个真实可用的例子（`fastapi.HTTPException` 未使用导入，Ruff 的 `F401`），照着教案里的步骤做就行，不用另外准备。其余类别仍可按需要准备：

- 一个 Wheel 构建后入口命令配置错误的示例
- 一个 Smoke Test 无法连接时的日志示例

## 参考材料

完整参考实现在 <https://github.com/ukeSJTU/2026-summer-github-actions-reference>（公开仓库，`ukeSJTU` 个人账号下，完全按学生的方式操作：从模板创建、写 Spec、AI 辅助测试与实现、两个 PR 搭 CI、打 tag 发 Release）。这个仓库是公开的，不是私密的答案库——如果不希望学员提前搜到，注意不要在课堂上主动提起仓库名。仓库里能直接对照的内容：

- `specs/001-filter-todos/spec.md` — 完整 Spec 参考答案
- PR #1（`spec: define completed todo filtering` → `test: add completed filter acceptance cases` → `feat: implement completed todo filtering` → `ci: add lint and test workflow` → `fix: remove unused import caught by ruff`）— 完整测试和实现参考答案，含一次真实的 Ruff 失败和修复记录
- PR #2（`ci: add build, smoke-test, and release jobs`）— 完整 Workflow 参考答案
- Release [`v0.1.0`](https://github.com/ukeSJTU/2026-summer-github-actions-reference/releases/tag/v0.1.0) — 完整发布产出，含 wheel、sdist 两个 Asset

## 课前演练

已经跑过一次完整流程（上面的参考仓库就是这次演练的产出，模板创建 → Spec → AI 实现 → 两个 PR 搭建 lint/test/build/smoke-test/release → 打 tag → 检查 Release，全部通过）。开课前建议教师自己再手动过一遍，找到属于自己的讲解节奏，尤其是 `docs/lesson/05-basic-ci.md` 到 `07-release.md` 这三段——照着教案的逐字稿操作一次，确认自己讲的时候不会卡壳。
