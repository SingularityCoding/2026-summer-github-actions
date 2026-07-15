# 0:00–0:15 项目基线

不需要逐字稿——对 FastAPI 足够熟悉，这一段是带学员confirm环境，不是讲解新概念。

## 要做的事

- 一句话说明今天要做什么："我们要给这个 Todo API 加一个筛选功能，全程走 Spec → AI 实现 → CI → Release 这条链路。"
- 带大家跑：
  ```bash
  uv sync --locked
  uv run ruff check .
  uv run pytest
  uv run todo-api
  ```
- 打开 `/docs`，调用 `/health`，调用已有的 `GET /todos`、`POST /todos`。
- 扫一眼项目结构：`src/todo_api/`（`main.py`/`models.py`/`repository.py`/`routers/`）、`tests/`。不用逐文件讲解，学员用得到的时候自己会去看。

## 检查点

每位学员本地项目跑起来，`/health` 能访问。卡住的（比如 `uv` 没装好）现在处理，不要拖到后面。
