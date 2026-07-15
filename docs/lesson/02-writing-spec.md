# 0:15–0:35 编写功能 Spec

这一段需要先由老师现场演示一次 AI 辅助写 Spec，学员看完演示后在自己仓库里重复同样的操作。不是逐字稿，但演示用的 prompt 要固定，照着念/照着打字，不要临场发挥措辞。

## 开场（1 分钟）

口头交代这次的需求（不用打字，学员待会自己会在 Spec 里读到细节）：

> "我们要给 `GET /todos` 加一个 `completed` 查询参数，用来按完成状态筛选。不传参数返回全部；`completed=true`/`completed=false` 分别只返回对应状态；没有匹配项返回空数组；顺序保持不变；传了无效值要返回 422。分页、认证、数据库持久化、Todo 结构调整、前端这些都不在这次范围内。"

## 演示：给 AI 的 Prompt（照着输入）

```text
我们要给现有的 Todo API 加一个新功能：GET /todos 支持 completed 查询参数，用来按完成状态筛选。请用 spec-driven-development 这个 skill，在 specs/001-filter-todos/spec.md 里写一份 Spec。这是在现有项目里加一个小功能，不是新建项目，Tech Stack、Commands、Project Structure、Testing Strategy 这些项目级字段直接引用现有的 pyproject.toml、README.md 和 AGENTS.md，不用重新写。重点说清楚这些行为：不传 completed 时返回什么、completed=true 和 completed=false 分别返回什么、没有匹配结果时返回什么、传了无效的布尔值怎么处理、返回顺序要不要变。同时明确这次不做什么（比如分页、认证、数据库持久化、Todo 数据结构改动、前端）。最后给出可以直接验证的验收场景。
```

## 讲解要点（AI 生成过程中顺带讲）

- 这个 skill 默认会先列出假设、有歧义就先问，不直接写。这次需求已经写得比较完整，AI 大概率不会停下来提问——**如果 AI 真的停下来问了（比如问"无效布尔值具体指哪些取值"），这是好事，正好现场展示"Spec 要澄清歧义"这条原则，不要觉得是意外，顺势回答就行**：有效值只有 `true`/`false`（以及 FastAPI/Pydantic 对 bool 类型的标准等价写法），其余一律 422。
- AI 会先读 `AGENTS.md` 和 skill 说明，再读现有代码（`models.py`/`routers/todos.py`/`repository.py`/`tests/test_todos.py`），最后才写文件——这是"先理解现状再写 Spec"，值得点一下，别让学员以为 AI 是凭空编的。
- Spec 模板是 skill 自带的九段式（Objective/Tech Stack/Commands/Project Structure/Code Style/Testing Strategy/Boundaries/Success Criteria/Open Questions）。讲清楚 `AGENTS.md` 里的约定：Tech Stack/Commands/Project Structure/Testing Strategy 这四段因为项目已经存在，应该是"指向现有文件"的一两句话，不是重新展开；真正要写满的是 Objective、Boundaries、Success Criteria。

## 预期产出（跑完这个 prompt 应该长这样）

完整参考见 `ukeSJTU/2026-summer-github-actions-reference` 仓库的 `specs/001-filter-todos/spec.md`（对应 commit `spec: define completed todo filtering`）。关键是 Success Criteria 要包含（可以照这个数目对一下，学员写少了就是漏了场景）：

1. 不传 `completed` → 全部，顺序不变
2. `completed=true` → 只有已完成
3. `completed=false` → 只有未完成
4. 没有匹配结果 → 空数组（不是 404/null）
5. 无效布尔值 → 422
6. 筛选后子集顺序不变
7. 质量门槛：ruff + pytest 全过

## 提交

```bash
git switch -c feature/completed-filter
git add specs/001-filter-todos/spec.md
git commit -m "spec: define completed todo filtering"
```

学员提交后举手/发消息，老师快速扫一眼 Success Criteria 是不是覆盖了上面 7 条，覆盖了就示意进入下一阶段，不用逐字逐句 review。

## 时间与节奏

演示本身大约 1-2 分钟（AI 一次性生成，没有自然的中断点），加上开场和收尾，15-20 分钟应该够全员写完并提交自己的 Spec。真正会拖时间的是学员对着模板发呆不知道怎么起笔——如果卡住超过 3-5 分钟，直接把上面这段 prompt 甩给他们抄。
