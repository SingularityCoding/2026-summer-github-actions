# 0:35–1:05 AI 辅助测试与实现

同样先由老师演示一次，学员照做。这一段比写 Spec 更容易踩坑，重点讲"先红后绿"这个动作本身，不要图快跳过验证红灯这一步。

## 演示：给 AI 的 Prompt（照着输入）

```text
请先阅读 specs/001-filter-todos/spec.md 和现有项目代码。总结一下这次要改的地方以及可能涉及的文件；如果 Spec 有歧义，先提问，不要直接开始写代码。确认没有问题后，先根据 Success Criteria 补充或修改 tests/test_todos.py 里的测试，跑一下测试确认新测试真的会因为功能缺失而失败；然后再实现满足 Spec 的最小改动。不要为了实现去改 Spec，也不要删除或削弱已有的测试。做完之后运行 uv run pytest 和 uv run ruff check . 确认都通过，并说明每一条 Success Criteria 分别由哪个测试覆盖。
```

这段 prompt 就是 `AGENTS.md` 里"统一 Agent 指令"的口语版，本质是把 `AGENTS.md` 的 10 条规则用一句话触发。

## 讲解要点

- **先红后绿是硬性要求，不是形式主义**：AI 写完测试后必须先跑一次，看到真的失败（而且是"因为功能缺失"失败，不是因为测试本身写错了），再动手实现。如果 AI（或学员）图快直接把测试和实现一起写完再跑，等于跳过了"测试能不能抓到缺陷"这个验证——这是本阶段唯一必须现场展示的动作。
- **两个场景没法直接用 `/docs` 或 `curl` 展示**：Spec 里"没有匹配结果"（场景 4）和"筛选后顺序不变"（场景 6）需要专门构造一个完成状态单一/交替的仓库，而不是复用默认种子数据（默认数据一个 true 一个 false，凑不出"全部无匹配"或"4 条交替"的情况）。AI 应该在测试里新建一个专门的 `InMemoryTodoRepository` 实例，而不是硬套现有的 `client` fixture。学员如果自己写卡在这里，提示他们看 Spec 里这两条场景的具体数据构造。
- **无效布尔值具体是哪些取值** 容易被问到：FastAPI/Pydantic 对 `bool` 查询参数的标准强制转换只接受 `true`/`false`（以及等价写法如 `1`/`0`），像 `"maybe"`、`"2"`、`""` 都应该 422。这个不用死记，AI 会自己验证，但老师心里要有数，免得被问住。

## 预期产出

完整参考见 `ukeSJTU/2026-summer-github-actions-reference` 的两个 commit：`test: add completed filter acceptance cases`（先提交，跑一遍应该是 7 个新测试全红）、`feat: implement completed todo filtering`（再提交，14 个测试全绿）。

Success Criteria → 测试的对应关系（用来检查学员有没有漏场景）：

| Success Criteria | 测试函数 |
|---|---|
| 不传参数，全部+原顺序 | `test_list_todos_preserves_order_and_mixed_states`（已有，不用改） |
| `completed=true` | `test_filter_todos_completed_true_returns_only_completed` |
| `completed=false` | `test_filter_todos_completed_false_returns_only_pending` |
| 无匹配 → 空数组 | `test_filter_todos_with_no_matches_returns_empty_list` |
| 无效布尔值 → 422 | `test_filter_todos_rejects_invalid_boolean`（parametrize `"maybe"`/`"2"`/`""`） |
| 子集保序 | `test_filter_todos_preserves_original_order_within_subset` |
| ruff + pytest 全过 | 不对应单个测试，是最后跑 `uv run pytest` + `uv run ruff check .` 确认 |

实现改动只有 `src/todo_api/routers/todos.py` 一处：给 `list_todos` 加一个 `completed: Annotated[bool | None, Query()] = None` 参数，`None` 时原样返回，否则按 `todo.completed == completed` 过滤。

## 提交

```bash
git add tests/test_todos.py
git commit -m "test: add completed filter acceptance cases"
git add src/todo_api/routers/todos.py
git commit -m "feat: implement completed todo filtering"
```

先提交测试再提交实现，两个 commit 分开——这样任何人（包括学员自己回看历史）都能重放"先红后绿"的过程。

## 时间与节奏

参考实现里这一步大约 6 个工具调用（读 AGENTS.md/Spec → 读现有代码 → 写测试 → 跑红 → 写实现 → 跑绿），演示带讲解压缩在 15-20 分钟内可以做完。但这是"熟手 AI + 熟手老师"的用时——学员自己第一次做，容易在两个地方多花时间：构造专门的仓库 fixture（场景 4/6），以及验证"无效布尔值"具体包含哪些取值。按 30 分钟排（0:35–1:05），不要压缩到 20 分钟，卡在这两点上是正常现象，不是学员进度慢。
