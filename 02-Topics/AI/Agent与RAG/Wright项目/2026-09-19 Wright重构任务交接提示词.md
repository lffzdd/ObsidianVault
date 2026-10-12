# Wright 重构任务交接提示词

给执行型 AI 使用。每个 Prompt 对应**一个独立 PR**,互不依赖,建议按顺序做。

---

## 使用前须知(给你自己看的,不要粘给执行者)

1. **先处理未提交改动。** 当前仓库有 14 个已修改文件 + 1 个未跟踪文件(`src/wright/tests/autonomy/test_process_recovery.py`)。开工前先 commit 或 stash,否则执行者的 diff 会和你的改动缠在一起,无法审查。

2. **难度分级:**
   - Prompt 1、2、3 = 低风险,机械性,放心交给便宜模型。
   - Prompt 4 = 中风险,已改成纯物理拆分(mixin),零行为变更。
   - Prompt 5 = 高风险大 diff,建议最后做,或你自己做。

3. **我已经替执行者验证过的事实**(所以提示词里写的是确定值,不是猜测):
   - `ruff check src` 当前**零错误**,接入 CI 无需修任何代码。
   - `ruff format` 会重排 **119 个文件** → 提示词里已明令禁止。
   - `mypy` 宽松模式 38 个错误,其中大部分是**刻意的 `hasattr` duck-typing 测试接缝**,不是 bug → 提示词里已明令不要"修干净"。
   - `npm run build` 产物**完全可复现**(hash 一致,构建后 git 干净)→ 所以漂移检查可行。
   - `AutonomyStore._lock` 是 `RLock`(可重入),但 `_write()` 内含 `with self._conn:`(SQLite 事务),**嵌套会破坏原子性** → 这是 Prompt 4 的核心约束。

---

# Prompt 1 — 接上已有的质量关卡(最高优先级)

```markdown
你在 /Users/williamlao/Project/wright 这个 Python 项目里工作。这是一个终端 coding agent,
用 uv 管理依赖,源码在 src/wright/,前端在 web/。

## 背景

这个项目已经配好了 lint 规则、写好了前端测试,但 CI 里一个都没跑。你的任务是把这些
已经存在的东西接进 CI。你不需要新写任何业务代码。

## 任务

### 1. 把 ruff 加进开发依赖

pyproject.toml 里 [tool.ruff] 有完整的 lint 配置,但 [dependency-groups] 的 dev 组
只有 pytest。把 ruff 加进去:

    [dependency-groups]
    dev = ["pytest>=8", "ruff>=0.15"]

然后运行 `uv sync --group dev`。

### 2. 在 CI 里加 lint 步骤

编辑 .github/workflows/ci.yml,在现有的 "Pytest" 步骤**之前**插入一步:

    - name: Lint
      run: uv run ruff check src

### 3. 在 CI 里加前端测试与构建

web/ 目录已经有 vitest 测试和 vite 构建配置。在 ci.yml 里新增一个独立 job:

    web:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - uses: actions/setup-node@v4
          with:
            node-version: "22"
        - name: Install
          run: npm ci
          working-directory: web
        - name: Unit tests
          run: npm run test
          working-directory: web
        - name: Build
          run: npm run build
          working-directory: web
        - name: Verify committed assets are up to date
          run: git diff --exit-code -- src/wright/web/static

最后一步的意义:src/wright/web/static/ 里提交了预构建产物,这一步确保它和 web/src/
的源码没有漂移。我已经验证过这个构建是可复现的,所以这一步现在会通过。

## 绝对不要做的事

- **不要运行 `ruff format` 或 `ruff check --fix`。** 这个项目没有使用 formatter,
  `ruff format` 会重排 119 个文件,产生一个无法审查的巨大 diff 并毁掉 git blame。
  只运行 `ruff check`(不带 --fix)。
- **不要修改任何 .py 源文件。** `ruff check src` 当前已经是零错误。如果你看到错误,
  说明你装的 ruff 版本比配置预期的新,这时降低 ruff 版本约束,不要改源码。
- 不要引入 pre-commit、tox、nox 或任何新的工具框架。
- 不要改动 pyproject.toml 里 [tool.ruff] 的任何规则(select / ignore 列表都别动)。
- 不要升级任何现有依赖的版本。

## 验收标准(必须逐条实际运行并粘贴输出)

1. `uv run ruff check src` → 输出 `All checks passed!`
2. `uv run pytest -q` → 全部通过,0 failed(当前基线是 498 passed, 2 skipped;
   数字可能因未提交改动略有不同,关键是 0 failed)
3. `cd web && npm run test` → 17 passed
4. `cd web && npm run build && cd .. && git diff --exit-code -- src/wright/web/static`
   → 退出码 0,无输出
5. `git diff --stat` → 只应包含 pyproject.toml、uv.lock、.github/workflows/ci.yml
   这三个文件。如果出现任何 .py 或 .tsx 文件,你做错了,回退重做。

最后报告:你改了哪几个文件、每条验收命令的实际输出。
```

---

# Prompt 2 — 加一个导入环回归测试

````markdown
你在 /Users/williamlao/Project/wright 这个 Python 项目里工作。源码在 src/wright/,
测试在 src/wright/tests/,用 pytest。

## 背景

这个项目的模块依赖图有一个重要但脆弱的性质:src/wright/ 下有 11 个模块在"逻辑上"
构成一个循环依赖(interaction、permission、permission.config、permission.interactive、
permission.resolver、renderer、session、tool_capabilities、tools.base、ui_events、util),
但所有的"回边"都被刻意放在 `if TYPE_CHECKING:` 块里,所以**真实的运行时导入图是无环的**。

这个性质完全靠人工纪律维持,没有任何测试保护它。如果有人不小心把一个 TYPE_CHECKING
导入提到模块顶层,就会引入真实的循环导入,而现在没有任何东西会报警。

你的任务:加一个测试把这个性质钉住。

## 任务

新建文件 src/wright/tests/test_import_graph.py,内容如下(直接使用,不要改写逻辑):

```python
"""Guard the runtime import graph against cycles.

src/wright 有若干模块在类型层面互相引用（TYPE_CHECKING 回边），但运行时导入图
必须保持无环。这个测试用 AST 静态分析只统计真正会在运行时执行的 import。
"""

from __future__ import annotations

import ast
from pathlib import Path

PACKAGE_ROOT = Path(__file__).resolve().parents[1]

def _module_name(path: Path) -> str:
    rel = path.relative_to(PACKAGE_ROOT)
    parts = list(rel.parts)
    if parts[-1] == "__init__.py":
        parts.pop()
    else:
        parts[-1] = parts[-1].removesuffix(".py")
    return ".".join(parts)

def _collect_modules() -> dict[str, Path]:
    modules: dict[str, Path] = {}
    for path in PACKAGE_ROOT.rglob("*.py"):
        if "__pycache__" in path.parts or "tests" in path.parts:
            continue
        modules[_module_name(path)] = path
    return modules

def _type_checking_nodes(tree: ast.Module) -> set[int]:
    """id() of every node nested under an `if TYPE_CHECKING:` block."""
    guarded: set[int] = set()
    for node in ast.walk(tree):
        if isinstance(node, ast.If) and "TYPE_CHECKING" in ast.dump(node.test):
            for child in ast.walk(node):
                guarded.add(id(child))
    return guarded

def _runtime_edges(module: str, path: Path, known: set[str]) -> set[str]:
    tree = ast.parse(path.read_text(encoding="utf-8"))
    guarded = _type_checking_nodes(tree)
    is_package = path.name == "__init__.py"
    base = module if is_package else module.rpartition(".")[0]
    edges: set[str] = set()
    for node in ast.walk(tree):
        if id(node) in guarded:
            continue
        targets: list[str] = []
        if isinstance(node, ast.ImportFrom):
            if node.level:
                parts = base.split(".") if base else []
                for _ in range(node.level - 1):
                    if parts:
                        parts.pop()
                if node.module:
                    parts = [*parts, node.module]
                targets.append(".".join(parts))
            elif (node.module or "").startswith("wright"):
                targets.append(node.module[len("wright") :].lstrip("."))
        elif isinstance(node, ast.Import):
            for alias in node.names:
                if alias.name.startswith("wright"):
                    targets.append(alias.name[len("wright") :].lstrip("."))
        for target in targets:
            if target and target in known and target != module:
                edges.add(target)
    return edges

def _find_cycle(graph: dict[str, set[str]]) -> list[str] | None:
    WHITE, GREY, BLACK = 0, 1, 2
    color = dict.fromkeys(graph, WHITE)
    stack: list[str] = []

    def visit(node: str) -> list[str] | None:
        color[node] = GREY
        stack.append(node)
        for neighbour in sorted(graph.get(node, ())):
            if color.get(neighbour, BLACK) == GREY:
                return [*stack[stack.index(neighbour) :], neighbour]
            if color.get(neighbour, BLACK) == WHITE:
                found = visit(neighbour)
                if found is not None:
                    return found
        stack.pop()
        color[node] = BLACK
        return None

    for node in sorted(graph):
        if color[node] == WHITE:
            found = visit(node)
            if found is not None:
                return found
    return None

def test_runtime_import_graph_is_acyclic() -> None:
    modules = _collect_modules()
    known = set(modules)
    graph = {
        name: _runtime_edges(name, path, known) for name, path in modules.items()
    }
    cycle = _find_cycle(graph)
    assert cycle is None, (
        "运行时导入出现循环依赖："
        + " -> ".join(cycle or [])
        + "\n如果这条边只是为了类型标注，请把它移到 `if TYPE_CHECKING:` 块里。"
    )
```

## 绝对不要做的事

- **不要为了让测试通过而修改 src/wright/ 下的任何源文件。** 这个测试在当前代码上
  就应该是通过的。如果它失败了,说明你把测试脚本抄错了,去修测试脚本,不要动源码。
- 不要把任何现有的 TYPE_CHECKING 导入提到模块顶层。
- 不要安装任何新依赖(这个测试只用标准库的 ast / pathlib)。
- 不要修改其他任何测试文件。

## 验收标准(必须实际运行并粘贴输出)

1. `uv run pytest src/wright/tests/test_import_graph.py -v` → 1 passed
2. `uv run pytest -q` → 0 failed,总数比之前多 1
3. 反向验证测试真的有效:临时把 src/wright/tool_capabilities.py 里
   `if TYPE_CHECKING:` 下的 `from .session import ...` 提到文件顶层,
   再跑 `uv run pytest src/wright/tests/test_import_graph.py`,
   确认它**失败**并打印出环路。然后**用 `git checkout` 完整还原这个文件**。
   在报告里粘贴这次失败的输出,证明测试不是永真的。
4. `git status --short` → 只有一个新增文件 src/wright/tests/test_import_graph.py

最后报告:第 3 步的失败输出(证明测试有效)、以及最终 git status。
````

---

# Prompt 3 — 清理误提交的产物

```markdown
你在 /Users/williamlao/Project/wright 这个仓库工作。这是一个终端 coding agent 项目。

## 背景

仓库里提交了一些和项目无关的产物,需要清掉。

## 任务

### 1. 删除 novel/ 目录

这个目录里是 3 章中文小说加一个 README,是用这个 agent 做测试时生成的产物,
和项目本身无关。用 `git rm -r novel` 删除。

### 2. 把 .design-qa/ 从版本控制里移除

这里面是 3 张不同尺寸的 Web 界面截图(desktop / tablet / mobile)。
截图不适合长期放在仓库里。执行:

    git rm -r --cached .design-qa

然后在 .gitignore 里加一行 `.design-qa/`。注意用 `--cached`,
保留本地文件,只是不再跟踪。

### 3. 检查 design-qa.md

根目录有一个 design-qa.md(6.2KB 的设计走查记录)。**先读它的内容**,然后判断:
- 如果它是一份仍然有参考价值的设计规范/检查清单 → 保留,但把它移到 docs/ 下。
- 如果它只是某次一次性走查的结论记录,且里面引用了 .design-qa/ 的截图 → 也移到 docs/。

无论哪种情况都**不要删除它**,只移动位置,并修正里面指向 .design-qa/ 的相对路径
(如果有的话)。在报告里说明你的判断依据。

## 绝对不要做的事

- 不要删除或移动 docs/ 下任何现有文件。
- 不要删除 examples/、scripts/、.github/ 下任何东西。
- 不要碰 src/ 或 web/ 下任何文件。
- 不要修改 .gitignore 里任何现有行,只追加 `.design-qa/`。
- 不要 commit,只把改动留在工作区。

## 验收标准

1. `uv run pytest -q` → 0 failed(应该和之前完全一样,这些文件不参与测试)
2. `git status --short` → 只应看到:novel/ 下文件被删除、.design-qa/ 下文件被取消跟踪、
   .gitignore 被修改、design-qa.md 被移动。不应有任何 src/ 或 web/ 下的改动。
3. `ls novel` → 报告目录不存在
4. `ls .design-qa` → 本地文件仍然存在(因为用了 --cached)

最后报告:你对 design-qa.md 的判断和处理方式、完整的 git status 输出。
```

---

# Prompt 4 — 拆分 AutonomyStore(纯物理拆分,零行为变更)

```markdown
你在 /Users/williamlao/Project/wright 这个 Python 项目里工作。

## 背景

src/wright/autonomy/store.py 有 1808 行,里面是一个叫 AutonomyStore 的类,
承担了 7 组互不相关的职责,有 44 个公开方法、88 条 SQL、6 个 schema 迁移。
它是整个项目最难读的文件。

你的任务是把它**按物理位置拆开**,让每个文件都在 400 行以内。

## 极其重要的约束:这是纯物理拆分,不是重新设计

你必须做到 **零行为变更**。具体来说:

- **不要修改任何方法体内的任何一行代码。** 你只是把方法从一个文件搬到另一个文件。
- **不要改任何方法的签名、名字或顺序。**
- **不要引入新的类来"持有"数据库连接。** 连接、锁、迁移全部留在基类里。
- **不要把任何方法拆成更小的方法。**
- **不要修改任何 SQL 语句。**

### 为什么这么严格:一个会导致数据损坏的雷

AutonomyStore 用 `self._lock = threading.RLock()`(可重入),配合两个上下文管理器
`self._read()` 和 `self._write()`。`_write()` 的实现是:

    @contextmanager
    def _write(self):
        with self._lock:
            self._ensure_open()
            with self._conn:      # <-- 这是一个 SQLite 事务
                yield

`with self._conn:` 是 SQLite 的事务上下文。**它不能安全嵌套**:如果你让一个
`_write()` 方法去调用另一个 `_write()` 方法,内层退出时就会提交事务,
外层剩下的操作将不在同一个原子单元里,出错时无法回滚 → 数据不一致。

现有代码用 `_xxx_locked` 命名约定规避了这个问题(这些方法假定调用者已经持有事务,
所以它们内部不再调 `_write()`)。

**所以:如果你为了拆分而引入任何新的跨对象方法调用,就有可能制造出嵌套事务。
用继承(mixin)而不是组合/委托,就完全不会有这个问题,因为所有方法仍然是同一个
对象上的方法,`self._write()` 的调用关系一个字都不变。**

### 一个实证例子,说明为什么必须用 mixin

`materialize_due`(L993,123 行)在**一个** `self._write()` 事务里横跨调用了
分属不同职责组的 6 个辅助方法:

    self._has_live_run_locked(...)          # 我分给 _automations
    self._note_pending_event_locked(...)    # 我分给 _automations
    self._flush_pending_events_locked(...)  # 我分给 _automations
    self._automation_from_row(...)          # 我分给 _automations
    self._advance_interval(...)             # 我分给 _automations
    self._file_snapshot(...)                # 我分给 _automations
    self._insert_run_locked(...)            # 我分给 _runs

如果改成仓储对象互相持有引用,这些调用就会变成跨对象调用,任何一个不小心
自己开了 `_write()`,就会在外层事务中间提交。用 mixin,它们全都还是 `self.` 上的
普通方法调用,**一个字都不用改**。

**因此:看到这种跨组调用是完全正常的,不要试图"消除"它,也不要为了让分组
更"纯粹"而移动这些 `_locked` 方法。照归属表放就行。**

## 具体做法:mixin 拆分

在 src/wright/autonomy/ 下新建一个 store/ 包(把原来的 store.py 变成 store/ 目录),
或者保留 store.py 并新建同级模块文件 —— 你选前者,结构更清晰:

    src/wright/autonomy/store/
        __init__.py        # 只负责 re-export,保证外部导入路径不变
        _base.py           # 连接、锁、_read/_write/_ensure_open/close/closed、异常类
        _migrations.py     # _initialize_schema + _migrate_v2..v6
        _automations.py    # automation 生命周期
        _commands.py       # 交互命令幂等
        _interactions.py   # 权限 / ask-user 交互记录
        _tool_executions.py# 工具意图 / 结果 / 恢复
        _history.py        # durable run 历史事实流
        _runs.py           # run 调度、认领、完成、恢复
        _scheduling.py     # materialize_due + web probe
        _helpers.py        # 模块级纯函数

### 方法归属表(严格按这个分配,行号是原 store.py 的行号)

**_base.py** — 类名 `_StoreBase`,同时放 `AutonomyStoreError`(L20)、`AutonomyNotFoundError`(L24)
  `__init__`(L36)、`close`(L62)、`closed`(L70)、`_ensure_open`(L73)、`_read`(L78)、`_write`(L84)

**_migrations.py** — 类名 `_MigrationsMixin`
  `_initialize_schema`(L90)、`_migrate_v2`(L164)、`_migrate_v3`(L242)、
  `_migrate_v4`(L270)、`_migrate_v5`(L289)、`_migrate_v6`(L322)

**_automations.py** — 类名 `_AutomationsMixin`
  `create_automation`(L342)、`get_automation`(L401)、`list_automations`(L411)、
  `pause_automation`(L420)、`resume_automation`(L439)、`cancel_automation`(L466)、
  `emit_event`(L495)、`_normalize_trigger`(L1499)、`_initial_next_run`(L1514)、
  `_advance_interval`(L1532)、`_resume_next_run`(L1540)、`_automation_from_row`(L1664)、
  `_has_live_run_locked`(L1562)、`_note_pending_event_locked`(L1570)、
  `_flush_pending_events_locked`(L1586)、`_file_snapshot`(L1549)

**_commands.py** — 类名 `_CommandsMixin`
  `accept_command`(L518)、`_command_from_row`(L565)、`queue_command`(L581)、
  `recover_commands`(L592)、`claim_command`(L620)、`start_command`(L639)、
  `cancel_command`(L652)、`complete_command`(L668)、`get_command`(L693)

**_interactions.py** — 类名 `_InteractionsMixin`
  `record_interaction`(L705)、`resolve_interaction`(L721)、
  `terminate_pending_interactions`(L736)、`list_interactions`(L749)

**_tool_executions.py** — 类名 `_ToolExecutionsMixin`
  `record_tool_intent`(L766)、`mark_tool_started`(L834)、`record_tool_result`(L847)、
  `recover_tool_executions`(L869)、`list_tool_executions`(L882)

**_history.py** — 类名 `_HistoryMixin`
  `record_run_event`(L906)、`list_run_history`(L950)、`run_history`(L971)

**_scheduling.py** — 类名 `_SchedulingMixin`
  `materialize_due`(L993)、`list_due_web_probes`(L1116)、
  `record_web_probe`(L1132)、`defer_web_probe`(L1183)

**_runs.py** — 类名 `_RunsMixin`
  `get_run`(L1212)、`list_runs`(L1219)、`claim_next_run`(L1231)、
  `count_active_runs`(L1279)、`start_run`(L1290)、`set_run_root_turn`(L1312)、
  `cancel_run`(L1321)、`is_cancel_requested`(L1343)、`finish_run`(L1349)、
  `recover_interrupted`(L1418)、`_run_query`(L1654)、`_run_from_row`(L1688)、
  `_insert_run_locked`(L1622)

**_helpers.py** — 模块级函数,不属于任何类
  `_bounded`(L1719)、`_dump`(L1730)、`_load_object`(L1737)、
  `_bounded_history_payload`(L1747)、`_hash_payload`(L1769)、
  `_safe_run_config`(L1773)、`_redact_arguments`(L1803)

### __init__.py 里的组装

    class AutonomyStore(
        _MigrationsMixin,
        _AutomationsMixin,
        _CommandsMixin,
        _InteractionsMixin,
        _ToolExecutionsMixin,
        _HistoryMixin,
        _SchedulingMixin,
        _RunsMixin,
        _StoreBase,
    ):
        """SQLite-backed store for durable automations, runs, and commands."""

注意 `_StoreBase` 必须放**最后**(MRO 最末),这样它的 `__init__` 是最终实现。
`__init__.py` 还要 re-export `AutonomyStore`、`AutonomyStoreError`、`AutonomyNotFoundError`,
并定义 `__all__`,保证 `from .store import AutonomyStore` 这类现有导入完全不变。

### mixin 的类型标注

每个 mixin 都会用到 `self._conn`、`self._lock`、`self._read()`、`self._write()`、
`self.session_id` 等基类成员。为了让类型检查和 IDE 不报错,在每个 mixin 文件里
让 mixin 继承 `_StoreBase`(而不是用裸 mixin):

    from ._base import _StoreBase
    
    class _RunsMixin(_StoreBase):
        ...

这样 MRO 仍然正确(所有 mixin 共享同一个基类),且不需要写 Protocol。

## 绝对不要做的事

- 不要改动任何方法体的内容。搬运时用复制粘贴,不要"顺手优化"。
- 不要改 src/wright/autonomy/ 下其他文件(models.py、runner.py、scheduler.py)。
- 不要修改任何测试文件。测试是你唯一的安全网,改了就失去意义。
- 不要改变 `from .store import ...` 或 `from wright.autonomy.store import ...`
  这类外部导入路径的可用性。
- 不要引入组合/委托/仓储模式。用 mixin 继承,理由见上面的事务嵌套说明。
- 不要新增或删除任何 import 之外的代码。

## 验收标准(必须逐条实际运行并粘贴输出)

1. `uv run pytest -q` → 0 failed。**这是最重要的一条**,
   其中 src/wright/tests/autonomy/test_store.py 有 589 行专门测这个类。
2. `uv run ruff check src` → All checks passed!
3. 验证公开 API 完全没变,运行这段脚本并粘贴输出:

       uv run python -c "
       from wright.autonomy.store import AutonomyStore, AutonomyStoreError, AutonomyNotFoundError
       names = sorted(n for n in dir(AutonomyStore) if not n.startswith('__'))
       print(len(names)); print('\n'.join(names))
       "

   把这个输出和你**开工前**在原始代码上运行同一脚本的输出做 diff,
   必须**完全一致**。在报告里贴出 diff 结果(应为空)。
   注意:开工前就先跑一次存下来。
4. `wc -l src/wright/autonomy/store/*.py` → 每个文件都 < 450 行
5. `git diff --stat` → 不应包含 src/wright/tests/ 下任何文件

最后报告:第 3 步的 API diff(必须为空)、每个新文件的行数、pytest 输出。
```

---

# Prompt 5 — core 分层(高风险,最后做)

> 这个改动会碰几乎所有文件的 import,是个 merge conflict 磁铁。
> 建议等前面 4 个 PR 都合并后再单独做,且做的时候不要并行开其他分支。

```markdown
你在 /Users/williamlao/Project/wright 这个 Python 项目里工作。

## 背景

src/wright/ 根目录平铺了 38 个 .py 模块,共约 10000 行,占生产代码的 38%,
没有任何内部分层。更糟的是命名严重重叠 —— "session"、"runtime"、"host"
这几个词各自指代好几个不同的东西:

- session:Session(可检查点的领域对象)、SessionService(应用服务)、
  session_host(事件队列消费者)、session_models、session_service
- runtime:WrightRuntime(服务定位器)、RuntimeServices(活句柄集合)、
  runtime.py(装配 + CLI 解析)、web/runtime_manager
- host:session_host(每个事件做什么)、application_host(持久自动化所有者)

你的任务:把根目录的模块按职责分进子包。**这是纯粹的文件移动 + import 更新,
不改任何逻辑。**

## 目标结构

在 src/wright/ 下新建三个子包,把现有模块移进去:

    src/wright/domain/      # 可检查点的领域状态,不依赖 engine/app
        session.py          (原 session.py)
        session_models.py
        conversation.py
        runs.py
        coordination.py
        checkpoint.py
    
    src/wright/engine/      # 模型循环与工具执行
        agent.py
        executor.py
        context.py
        looping.py
        verifier.py
        subagent.py
        agent_background.py
    
    src/wright/app/         # 应用层:装配与事件分发
        runtime.py          (原 runtime.py)
        services.py
        session_service.py
        event_dispatch.py   (原 session_host.py —— 顺便改名消解 "host" 歧义)
        application_host.py
        lifecycle.py

其余模块(llm.py、model.py、model_adapters.py、prompt.py、protocol.py、
renderer.py、ui_events.py、interaction.py、repl.py、headless.py、terminal.py、
processes.py、execution.py、paths.py、project.py、artifacts.py、attachments.py、
capabilities.py、tool_capabilities.py、events.py、logger.py、util.py、
worktrees.py、main.py)**全部留在原地不动**。

## 做法

1. 用 `git mv` 移动文件(保留 history),不要用 cp + rm。
2. 每个新子包加 `__init__.py`。**`__init__.py` 保持为空**,不要在里面写
   re-export —— 那会制造新的循环导入风险。
3. 更新所有 import。全部用显式相对导入,例如原来在 agent.py 里的
   `from .session import Session` 改成 `from ..domain.session import Session`。
4. `session_host.py` → `app/event_dispatch.py` 是唯一的改名。
   模块内的函数名(`process_session_event` 等)**不要改**。
5. 测试文件里的 import 也要跟着更新,但**不要改任何测试逻辑或断言**。

## 处理循环导入

这个项目目前的运行时导入图是**无环**的,靠的是把类型专用的导入放在
`if TYPE_CHECKING:` 里。移动文件时:

- 如果新的分层暴露出一个真实的循环导入(运行时 ImportError),
  **不要用"把 import 挪到函数内部"来绕过**。停下来,在报告里说明是哪两个模块、
  哪个方向,然后把该模块留在原地不移动。宁可少移一个文件,也不要制造隐蔽的
  延迟导入。
- 如果只是类型标注需要,才可以放进 `if TYPE_CHECKING:`。

如果仓库里已经有 src/wright/tests/test_import_graph.py,它会自动帮你验证这一点。

## 绝对不要做的事

- 不要修改任何函数、方法、类的实现。这次只动 import 语句和文件位置。
- 不要改任何公开类名或函数名(唯一例外是模块文件名 session_host → event_dispatch)。
- 不要合并、拆分或删除任何模块。
- 不要在 __init__.py 里写 re-export。
- 不要用 `sys.path` 技巧或延迟导入绕过循环依赖。
- 不要改 pyproject.toml(hatch 打包的是整个 src/wright 包,不需要改)。
- 不要动 web/ 或 .github/。
- 不要运行 ruff format。

## 验收标准(必须逐条实际运行并粘贴输出)

1. `uv run pytest -q` → 0 failed,总数和开工前完全一致
2. `uv run ruff check src` → All checks passed!
3. `uv run python scripts/smoke_agent.py` → 成功
4. `uv run python scripts/smoke_subagent.py` → 成功
5. `uv run python -c "import wright.main"` → 无输出无报错
6. `uv run wright --help` → 正常打印帮助
7. `git diff --stat -M` → 用 -M 让 git 识别重命名。检查输出里移动的文件
   显示为 rename 而不是 delete+add(证明用了 git mv)
8. 确认没有任何文件的实现被改动:
   `git diff -M --stat | tail -1` 报告的改动行数应该主要来自 import 行。
   在报告里说明:如果某个文件的 diff 超过 10 行(纯 import 更新之外),
   逐个解释原因。

最后报告:完整的 git diff --stat -M、所有验收命令的输出、
以及任何你决定"不移动"的模块及其循环依赖原因。
```

---

## 附:如果执行者卡住了

常见失败模式和对应处理:

| 症状 | 原因 | 对策 |
|---|---|---|
| 执行者跑了 `ruff format`,diff 119 个文件 | 没读禁止项 | 直接 `git checkout .` 回退,重新强调 Prompt 1 的禁止段 |
| 执行者开始"顺手修" mypy 报的类型错 | 越界 | 那 38 个错误大部分是刻意的 `hasattr` 测试接缝(`repl.py` 的 `service.put`、`session_service.py` 的 `state.user_goal` 都是死分支),不要碰 |
| Prompt 4 后 test_store.py 失败 | 方法搬错组 或 改了方法体 | 回退,对照归属表逐个核对;重点查 `_xxx_locked` 方法有没有被搬到别的 mixin |
| Prompt 5 出现 ImportError | 分层暴露了真实环 | 让它把那个模块留在原地,不要用延迟导入绕 |
