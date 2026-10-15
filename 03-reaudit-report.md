> **公开发布说明（本块为脱敏发布信息，非原始证据内容）**
> - 本文件是私有仓库 `yem638769-eng/ai-security-platform` 证据分支 `audit/evidence-2026-10-15`（证据提交 `92ac44e`）中 `docs/audit-evidence/2026-10-15/03-reaudit-report.md` 的脱敏公开副本。
> - 原始文件 SHA-256：`76CEB703B0139DB6EE2E014AEB95394510B5514ABCB6A33A12F5CA2B61EBA6A7`（与原始审计报告 `docs/reports/FULL_PROJECT_REAUDIT_2026-10-15.md` 逐字节一致；本副本以字节级复制生成，另加本说明块与下述 1 处替换）。
> - 脱敏替换（1 处）：§1 环境表 gh CLI 行的本机代理地址（本地回环地址与端口）→ `[本地代理地址已脱敏]`。该地址属内部网络环境配置，与任何审计结论无关。
> - 生成时间：2026-10-15 17:30（Asia/Shanghai）。以下为原始证据正文（含上述 1 处替换）。

# AI Security Platform — 全项目重新审计报告（Phase 24-C）

> 审计日期：2026-10-15（Asia/Shanghai）
> 审计基线：分支 `main`，HEAD `faae13363684fb1551231f7bd1b49e0ed2606802`（`faae1336`，PR #1 合并提交）= `origin/main`
> 审计性质：基于当前代码的重新全面审计。所有实现结论来自本轮对当前代码的直接核查（路由注册、文件树、符号检索、源码读取）；所有测试数字来自本轮实际执行（本地全量套件 + 质量检查）或当前基线上的真实 CI 运行记录（gh 实测）。不沿用旧报告结论，不将历史结果冒充为本轮结果。
> 与旧报告的关系：`FULL_PROJECT_AUDIT_2026-10-10.md`（第一轮）与 `FULL_PROJECT_SECOND_AUDIT_2026-10-10.recovered.md`（第二轮恢复副本）仅作历史资料保留，本报告未修改、未覆盖任何旧文件。

---

## 1. 审计范围、时间与环境

| 项 | 值 |
| --- | --- |
| 审计时间 | 2026-10-15 16:40–17:10（本轮取证与测试执行窗口） |
| 审计范围 | 后端全部 4 层（api/application/domain/infrastructure）、前端、CI、迁移、测试套件、Docker 环境 |
| Python | 项目级 `.venv`，Python 3.11.9（一切工具经 `.\.venv\Scripts\python.exe -m <module>` 调用） |
| PostgreSQL | 容器 `ai-security-postgres`（pgvector/pgvector:pg16）Up (healthy) |
| Redis | 容器 `ai-security-redis`（redis:7）Up (healthy) |
| gh CLI | 已认证（经本机代理 `[本地代理地址已脱敏]` 按命令临时设置，未改 git config） |
| 本轮未执行 | 前端构建（无业务代码）、真实 LLM smoke（opt-in 未授权）、semgrep 容器包装器定向运行（CI 已覆盖） |

## 2. 当前 Git 基线与工作区状态

| 项 | 值（本轮实测） |
| --- | --- |
| 分支 | `main` |
| HEAD | `faae13363684fb1551231f7bd1b49e0ed2606802` |
| origin/main | `faae13363684fb1551231f7bd1b49e0ed2606802`（一致，同步） |
| 近 3 提交 | `faae133` Merge PR #1 / `d0e0018` fix(semgrep) / `b31c9de` fix(security) |
| PR #1 | state **MERGED**，mergedAt 2026-10-12T05:25:40Z（gh 实测） |
| 工作区 | 已跟踪文件**零修改、零暂存、零删除**；未跟踪 = 4 个 phase18 备份 + `docs/reports/`（第一报告 43,625 B、第二轮恢复副本 32,905 B、本报告） |
| 测试写库说明 | 本轮全量测试按项目现行 conftest 机制直写开发库 `ai_security`（见 §4 隔离核查） |

## 3. 功能实现验收矩阵（55 项，唯一编号 F-01…F-55）

> 状态口径（互斥五分类 + 环境注记）：
> **PASS** = 实现存在 + 本轮本地测试或当前基线 CI 真实通过证据；
> **IMPLEMENTED_UNVERIFIED** = 代码完整但本轮无验证证据；
> **PARTIAL** = 部分实现；
> **NOT_IMPLEMENTED** = 本轮核查确认不存在；
> **ENV 注记** = 本地环境阻断，作为 PASS 行的附加说明（不改变主状态，CI 证据为判定依据）。

### A. 用户、认证与权限（7 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-01 | 用户注册（Argon2id） | `routes/auth.py` L29；`core/security.py` L18-28（pwdlib Argon2） | 本轮全量套件通过 | **PASS** |
| F-02 | 登录/JWT（HS256） | `routes/auth.py` L46；`core/security.py` L1-28 | 本轮全量套件通过 | **PASS** |
| F-03 | 身份校验 me | `routes/auth.py` L65 | 本轮全量套件通过 | **PASS** |
| F-04 | 项目 CRUD | `routes/projects.py` L32/46/56/71/91（5 端点） | 本轮全量套件通过 | **PASS** |
| F-05 | 仓库 CRUD（纯元数据，无 provider 集成——设计决策） | `routes/repositories.py` L49/71/90/110/132（5 端点） | 本轮全量套件通过 | **PASS** |
| F-06 | 资源级授权/IDOR 同构 404 | `application/findings/service.py` L84-114（SQL 级 ownership join + FOR UPDATE） | 本轮全量套件（含 IDOR 拒绝用例）通过 | **PASS** |
| F-07 | 输入校验/错误响应 | Pydantic schemas + `core/exceptions.py` 类型化异常 | 本轮全量套件通过 | **PASS** |

### B. 扫描与异步执行（9 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-08 | 扫描创建/调度（201 即返 + Celery `.delay`） | `routes/scans.py` L99；`infrastructure/tasks/scan_tasks.py` | 本轮全量套件通过 | **PASS** |
| F-09 | 真实异步链 HTTP→broker→worker→PG | `infrastructure/tasks/{scan_tasks,celery_app}.py`；测试 `infrastructure/tasks/test_async_scan_integration.py` | 本轮全量套件内通过（真实 Redis/Celery/PG） | **PASS** |
| F-10 | 幂等/FOR UPDATE/失败处理 | `application/scans/service.py` L64（locked load FOR UPDATE） | 本轮全量套件通过 | **PASS** |
| F-11 | bandit CLI 适配器 | `infrastructure/scanners/bandit.py` | 本轮全量套件（解析器）+ CI Run 38207885778（真实 CLI）通过 | **PASS** |
| F-12 | semgrep CLI 适配器 | `infrastructure/scanners/semgrep.py` + `tools/semgrep-docker/`（容器包装器） | 本轮全量套件（解析器）+ CI Run 38207885778（Linux 真实 CLI）通过 | **PASS** |
| F-13 | gitleaks CLI 适配器 | `infrastructure/scanners/gitleaks.py` | 同上模式通过 | **PASS** |
| F-14 | pip-audit CLI 适配器 | `infrastructure/scanners/pip_audit.py` | 同上模式通过 | **PASS** |
| F-15 | trivy CLI 适配器 | `infrastructure/scanners/trivy.py` | CI Run 38207885778 通过（含 trivy 测试）；**本地 ENV 阻断**（GHCR 漏洞库下载被网络阻断，本轮本地 3 failed，见 §7） | **PASS**（CI 证据 + 本地环境注记） |
| F-16 | 扫描器失败路径（不存在/超时/错误退出） | `infrastructure/scanners/cli.py` + 各适配器 | 本轮全量套件失败路径用例通过 | **PASS** |

### C. Finding 生命周期（6 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-17 | Finding 持久化/字段校验 | `domain/models/finding.py` | 本轮全量套件通过 | **PASS** |
| F-18 | fingerprint 去重（同 scan 首见胜出） | `domain/models/finding.py` + `application/findings/service.py` | 本轮全量套件通过 | **PASS** |
| F-19 | 跨 scan 历史保留 | 同上（fingerprint 即身份，无独立聚合表——设计形态） | 本轮全量套件通过 | **PASS** |
| F-20 | 状态机/PATCH/同态 409 | `domain/models/finding.py`（ALLOWED_TRANSITIONS + @validates）；`routes/findings.py` L50 | 本轮全量套件通过 | **PASS** |
| F-21 | 查询/过滤/分页/CWE/OWASP | `routes/findings.py` L41 + service | 本轮全量套件通过 | **PASS** |
| F-22 | SCA 无行号契约（null 而非伪造） | `domain/models/finding.py`（迁移 `c9308a858c7a` 允许 null start_line） | 本轮全量套件通过 | **PASS** |

### D. Triage 与 LLM（5 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-23 | Triage 服务/CodeContext（唯一 target_path 消费者） | `application/triage/{service,code_context}.py` | 本轮全量套件通过 | **PASS** |
| F-24 | 结构化输出解析 fail-closed | `application/triage/structured_output.py` | 本轮全量套件（含非法输出路径）通过 | **PASS** |
| F-25 | AI 永不改 Finding.status / append-only | triage 服务 + AST 守卫测试 | 本轮全量套件通过 | **PASS** |
| F-26 | AI 分析查询 API | `routes/triage.py` L97 | 本轮全量套件通过 | **PASS** |
| F-27 | 真实 LLM 调用路径（Ollama/OpenAI-compatible） | `infrastructure/llm/{ollama,openai_compatible}.py`（httpx 无 SDK） | 仅有 opt-in smoke（`test_llm_smoke.py` L21-23 门控 `RUN_LLM_SMOKE_TESTS=1`），本轮未执行 | **IMPLEMENTED_UNVERIFIED** |

### E. LLM Gateway、Agent 与 Workflow（5 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-28 | Provider 抽象/供应商隔离 | `infrastructure/llm/{base,resolver}.py` | 本轮全量套件通过 | **PASS** |
| F-29 | Security Agent 编排（迭代/重试/失败处理） | `application/agent/{service,state,decision,models}.py` | 本轮全量套件（agent 套件）通过 | **PASS** |
| F-30 | Agent 工具授权（每次调用先 authorize） | `application/agent/` + AST 守卫测试 | 本轮全量套件通过 | **PASS** |
| F-31 | Workflow 编排 + 分级降级 | `application/workflow/service.py` | 本轮全量套件通过 | **PASS** |
| F-32 | Workflow API（同构 404/封闭信封） | `routes/workflows.py` L155 | 本轮全量套件（`test_workflows_api.py`）通过 | **PASS** |

### F. RAG 与向量检索（5 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-33 | pgvector 存储/相似检索 | `infrastructure/rag/vectorstore/pgvector.py` | 本轮全量套件（真实 pgvector）通过 | **PASS** |
| F-34 | 知识去重/原子替换 | `application/rag/service.py` | 本轮全量套件通过 | **PASS** |
| F-35 | 闭键引用/检索失败降级/UNTRUSTED 分区 | `application/rag/{citations,retriever}.py` | 本轮全量套件通过 | **PASS** |
| F-36 | RAG 摄取生产入口 | **不存在**：`main.py` 仅注册 8 组 router（L21-28，无 knowledge）；ingest 逻辑仅存在于服务层（`application/rag/service.py`），无 API/CLI 入口 | 路由注册核查 + 符号检索（本轮实测） | **NOT_IMPLEMENTED** |
| F-37 | Embedding 语义能力 | `infrastructure/rag/embeddings/local_hash.py`（blake2b 特征哈希 256 维，L4/L9/L61，非语义；抽象接口 `base.py` 在） | 本轮全量套件通过（哈希基线）；语义 provider 无 | **PARTIAL** |

### G. Tool Layer、MCP 与审计（8 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-38 | ToolRegistry/Executor（owner/路径/白名单/截断） | `application/tools/{registry,executor,base}.py` | 本轮全量套件通过 | **PASS** |
| F-39 | bandit/gitleaks/trivy 三工具 | `application/tools/{bandit,gitleaks,trivy}.py`；`registry.py` L14-18 恰 3 项注册 | 本轮全量套件通过 | **PASS** |
| F-40 | semgrep 工具注册 | **不存在**：`registry.py` 仅 bandit_scan/gitleaks_scan/trivy_scan（本轮全文读取核实） | 注册表核查 | **NOT_IMPLEMENTED** |
| F-41 | stdio MCP Server/协议交互 | `infrastructure/mcp/{server,adapter,context}.py` | 本轮全量套件通过 | **PASS** |
| F-42 | MCP 授权（capability 白名单） | `infrastructure/mcp/authorization.py` | 本轮全量套件通过 | **PASS** |
| F-43 | 审计事件生成/脱敏 | `infrastructure/mcp/audit.py`（L30 AuditEventType、L104 sanitize_audit_value、L173 AuditSink Protocol、L179/L192 InMemory/Logging sink） | 本轮全量套件通过（含脱敏正则用例） | **PASS** |
| F-44 | 审计持久化 | **不存在**：11 个迁移无审计表（本轮迁移清单全量核对）；audit.py 仅内存/日志 sink | 迁移链核查 | **NOT_IMPLEMENTED** |
| F-45 | 远程 MCP/OAuth | **不存在**：`oauth` 全后端零命中（本轮检索） | 符号检索 | **NOT_IMPLEMENTED** |

### H. Fix Agent 与补丁（1 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-46 | Fix Agent（补丁生成/差异模型/应用/回滚/隔离测试/审批/验证——作为 1 个能力域计 1 项） | **不存在**：`apply_patch|PatchModel|fix_agent|FixAgent` 全后端零命中（本轮检索） | 符号检索 | **NOT_IMPLEMENTED** |

### I. Web UI 与报告（3 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-47 | 前端业务页面 | `frontend/src/` 仅 3 文件（App.vue/main.ts/style.css，本轮文件树核实） | 文件树核查 | **NOT_IMPLEMENTED** |
| F-48 | 报告 summary/report（JSON） | `routes/reports.py` L64/L82（两端点） | 本轮全量套件（`test_reports_api.py` 等）通过 | **PASS** |
| F-49 | 报告导出（PDF/CSV 等） | **不存在**：`reports.py` 内 `csv|pdf|export|download` 零命中（本轮检索） | 符号检索 | **NOT_IMPLEMENTED** |

### J. GitHub 产品级集成（1 项）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-50 | GitHub 集成（OAuth/webhook/PR 回写/仓库读取/凭据管理——作为 1 个能力域计 1 项） | **不存在**：`webhook|pull_request|oauth` 全后端零命中（本轮检索） | 符号检索 | **NOT_IMPLEMENTED** |

### K. 工程质量（5 项，汇总类，计入总数并在此注明类别）

| 编号 | 功能 | 代码证据 | 验证证据 | 状态 |
| --- | --- | --- | --- | --- |
| F-51 | CI 门禁（lint + test 双 job） | `.github/workflows/ci.yml`（L29 lint / L52 test；pgvector+redis 服务容器；钉定 5 个扫描器 CLI） | main @ `faae1336` Run 38207885778（2026-10-12）：**双 job success**（本轮 gh 实测） | **PASS** |
| F-52 | 静态门禁（Ruff/MyPy） | `pyproject.toml` L57-83 配置 | 本轮实测：Ruff "All checks passed"（exit 0）；MyPy "Success: 126 source files"（exit 0） | **PASS** |
| F-53 | 数据库迁移完整性 | `backend/alembic/versions/` 11 个迁移单链 | 本轮实测：`alembic current` == `alembic heads` == `d7c4a9e2f1b8` | **PASS** |
| F-54 | 依赖固化/版本钉定 | `pyproject.toml` 全 `~=` 约束；CI 钉定 bandit 1.9.4 / pip-audit 2.10.1 / semgrep 1.120.0 / gitleaks 8.30.1 / trivy 0.75.0 | 配置核查（注：semgrep venv 未钉 setuptools，P3 级，见 §8） | **PASS** |
| F-55 | 测试数据库隔离 | **不存在**：`TEST_DATABASE_URL` 全仓库零命中（本轮检索）；`backend/tests/conftest.py` L3 自述 "Tests run against the Docker Compose PostgreSQL (ai_security database)"，L31 直连应用 SessionLocal | conftest 全文机制核查（见 §4 隔离核查） | **NOT_IMPLEMENTED** |

## 4. 完整测试命令与测试统计（本轮实际执行）

| 类别 | 命令 | 结果 | 耗时/说明 |
| --- | --- | --- | --- |
| Ruff | `.\.venv\Scripts\python.exe -m ruff check .` | **All checks passed**（exit 0） | 本轮实测 |
| MyPy | `.\.venv\Scripts\python.exe -m mypy backend/app` | **Success: no issues found in 126 source files**（exit 0） | 本轮实测 |
| Alembic | `-m alembic current` / `-m alembic heads` | 均 `d7c4a9e2f1b8 (head)` | 本轮实测 |
| **本地全量套件** | `.\.venv\Scripts\python.exe -m pytest -q` | **968 passed / 3 failed / 14 skipped**（985 收集，exit 1） | **139.88s**，2026-10-15 16:45:04–16:47:24 |
| CI @ 当前基线 | Run 38207885778（push main `faae1336`，2026-10-12） | lint job **success**（37s）；test job **success**：**978 passed / 0 failed / 7 skipped / 244 warnings**（96.51s） | 本轮 gh 实测（job 级 + 日志摘要） |

**本地 3 个失败明细（全部环境阻断类，与代码无关的判定依据：CI 同测试通过）**：

| 测试 | 错误摘要 | 分类 |
| --- | --- | --- |
| `application/tools/test_trivy_tool.py::test_real_trivy_tool_end_to_end` | `assert 'failed' == 'success'`（底层 trivy exit 1） | GHCR 漏洞库下载被网络阻断 |
| `infrastructure/scanners/test_cli_integration.py::test_trivy_cli_integration` | `ScannerError: trivy exited with code 1: FATAL ... failed to download vulnerability DB: OCI artifact error` | 同上 |
| `test_scans_api.py::test_trivy_scan_via_api_returns_lineless_findings` | 同上（经 API+worker 链路） | 同上 |

**本地/CI 数字调和**：本地 968+3+14=985 收集；CI 978+0+7=985（同一套测试，平台差异：本地 Windows 缺原生 CLI 触发 skip 门控，CI Linux 全装并验证 PATH 后运行）。两处收集数一致 ✓。

**测试写库隔离核查（任务书要求，先核查后执行）**：
- 机制（conftest.py 本轮核实）：L3 自述直连开发库 `ai_security`；L31 导入应用 `SessionLocal`；`db_session` fixture 每测试 rollback+close；`register_user_for_cleanup` / `register_knowledge_for_cleanup` 在 teardown 按注册清单显式 DELETE（FK 逆序）+ commit。
- 判定：测试**会写开发库**，但仅创建并回收自有测试实体；无清库/重置/破坏性操作；已知风险 = 测试中断可能残留孤儿行（F-55，P2 级）。本轮全量套件正常结束（exit 1 为断言失败非中断），清理机制全程运转。
- CI 侧：`ci.yml` L60-92 用独立 `ai_security_ci` 一次性容器库，与开发数据物理隔离。

## 5. 质量、安全、依赖与迁移检查

| 检查项 | 结果 | 本轮证据 |
| --- | --- | --- |
| 认证安全 | ✅ | Argon2id（pwdlib 推荐 hasher）+ JWT HS256（`core/security.py` L1-28） |
| 越权/租户隔离 | ✅ | SQL 级 ownership join（Finding→Scan→Repository→Project→owner，`findings/service.py` L84-114）；IDOR 拒绝用例在套件内通过 |
| SQL 注入 | ✅ | SQLAlchemy 参数化 + ORM，无拼接 SQL |
| 命令注入 | ✅ | CLI 适配器 `subprocess` argv 列表模式（`scanners/cli.py`） |
| Secret 泄露 | ✅ | gitleaks 仅存 RuleID/描述/行号；审计层字段级脱敏（`audit.py` L104 sanitize）；`.env` 未入库，`.env.example` 在位 |
| LLM 输出安全 | ✅ | fail-closed 结构化校验；target_path 不进 prompt；AI 不改 Finding.status（套件 + AST 守卫） |
| 迁移完整性 | ✅ | 11 迁移单链无分叉；`current == heads == d7c4a9e2f1b8` |
| 依赖固化 | ✅ | pyproject 全 `~=`；CI 钉定全部 5 个扫描器 CLI 版本 |
| 环境配置卫生 | ✅ | `docker-compose.yml`、`alembic.ini`、`.env.example` 在位；仓库根无 Dockerfile（部署仅开发栈，见 §8 P3-06） |
| 测试规模 | — | 84 个 `test_*.py` 文件，985 收集用例，分层（unit/application/infrastructure/API/AST 守卫） |

## 6. 未实现功能与开发缺口（9 项，对应矩阵 NOT_IMPLEMENTED）

| 编号 | 缺口 | 核查方式（本轮） | 影响 |
| --- | --- | --- | --- |
| F-36 | RAG 摄取生产入口 | `main.py` 8 组 router 注册核查；ingest 仅服务层 | RAG 检索测试全绿但生产无数据来源 |
| F-40 | semgrep 工具注册 | `registry.py` 全文读取（恰 3 工具） | Agent/Workflow/MCP 无法调用 semgrep |
| F-44 | 审计事件持久化 | 11 迁移清单核对（无审计表）；audit.py 仅内存/日志 sink | 平台自身操作无留存轨迹 |
| F-45 | 远程 MCP/OAuth | `oauth` 零命中 | 仅 stdio 形态 |
| F-46 | Fix Agent/补丁/验证 | `apply_patch|PatchModel|fix_agent|FixAgent` 零命中 | "修复→验证"后半链不存在 |
| F-47 | 前端业务页面 | `frontend/src/` 文件树（3 个脚手架文件） | 无 UI 展示 |
| F-49 | 报告文件导出 | `reports.py` 导出符号零命中 | 仅 JSON DTO |
| F-50 | GitHub 产品级集成 | `webhook|pull_request|oauth` 零命中 | 无交付闭环 |
| F-55 | 测试数据库隔离 | `TEST_DATABASE_URL` 零命中 + conftest L3/L31 | 本地测试直写开发库（本轮全量套件即在此机制下运行） |

另有一项部分实现：F-37 Embedding 语义能力（blake2b 哈希基线在，语义 provider 缺）。

## 7. 环境阻断及未完成验证（与代码缺陷严格分开）

| 项 | 性质 | 证据 | 处置 |
| --- | --- | --- | --- |
| 本地 Trivy 3 测试失败 | **环境阻断**（GHCR 下载被本机网络阻断），非代码缺陷 | 本轮本地 3 failed 错误均为 "failed to download vulnerability DB: OCI artifact error"；CI Run 38207885778 同测试通过（978p/0f） | 未伪装为 PASS；未归咎代码；修复方向 = 为 trivy 配置可达代理（P2-01） |
| 真实 LLM smoke | **未执行**（opt-in 门控 `RUN_LLM_SMOKE_TESTS=1`，未经授权不发起） | `test_llm_smoke.py` L21-23 | F-27 记 IMPLEMENTED_UNVERIFIED |
| 前端构建 | **未执行**（仅 3 个脚手架文件，无业务代码可构建） | 文件树核查 | F-47 记 NOT_IMPLEMENTED（与构建成败无关） |
| semgrep 容器包装器定向运行 | 本轮未重跑 | 历史本地实证（2026-10-10，1 passed）；CI Linux 每次真实运行（Run 38207885778 test job success） | F-12 判定采用本轮 CI 证据 |

## 8. 风险分级与修复建议

> 本轮无 P0、无 P1（旧报告的 P1-01 main CI 红已随 PR #1 合入解决——本轮实测 main Run 38207885778 双 job success）。

| 级别 | ID | 项 | 对应功能记录 | 修复方向 |
| --- | --- | --- | --- | --- |
| P2 | P2-01 | 本地 Trivy 网络阻断 | F-15 环境注记 | 为 trivy 配置可达代理；验收 = 本地全量 0 failed |
| P2 | P2-02 | 测试直写开发库 | F-55 | TEST_DATABASE_URL 契约 + conftest 导入前守卫（fail-closed）；CI 已隔离不受影响 |
| P2 | P2-03 | RAG 无摄取入口 | F-36 | 摄取 API（需产品决策：权限/限额/去重/异步解析） |
| P2 | P2-04 | 审计无持久化 | F-44 | 审计表 + 迁移 + 查询 API（延续 append-only + 脱敏契约） |
| P2 | P2-05 | semgrep 未注册为工具 | F-40 | 注册第 4 工具，复用扫描器 + ToolExecutor 契约 |
| P3 | P3-01 | 文档债：README L26/54/163/199 仍声称"无 CI、CI 见 TODO"（本轮实测仍在）；Phase 编号歧义 | — | 文档同步批次（低风险） |
| P3 | P3-02 | Workflow 同步长请求（设计决策；大仓库超时风险） | F-31 注记 | 需产品决策后改异步 |
| P3 | P3-03 | CI semgrep venv 未钉 setuptools（`ci.yml` L115-117） | F-54 注记 | 一行钉定，对齐 trivy Dockerfile 模式 |
| P3 | P3-04 | pytest warnings：CI 本轮实测 244 条 | — | warnings 清理批次 |
| P3 | P3-05 | 4 个 phase18 备份文件未跟踪留存（本轮完好保留，未修改） | — | 处置待用户决定 |
| P3 | P3-06 | 无生产部署物（仓库根无 Dockerfile，仅 docker-compose 开发栈） | — | 需要部署需求后补 |

## 9. 逐项证据索引（关键代码位置，均本轮核实）

| 主题 | 位置 |
| --- | --- |
| 路由注册（8 组，无 knowledge） | `backend/app/main.py` L5-12（import）、L21-28（include_router）、L31/L39（health 两端点） |
| 端点清单（24 个） | auth L29/46/65；projects L32/46/56/71/91；repositories L49/71/90/110/132；scans L99/136/157/168；findings L41/50；triage L74/97；workflows L155；reports L64/82 |
| 密码/JWT | `backend/app/core/security.py` L1-28 |
| owner 隔离 | `backend/app/application/findings/service.py` L84-114 |
| 扫描幂等 | `backend/app/application/scans/service.py` L64 |
| 工具注册表（3 工具） | `backend/app/application/tools/registry.py` L14-18 |
| 审计层结构 | `backend/app/infrastructure/mcp/audit.py` L30/L104/L109/L173/L179/L192 |
| Embedding 基线 | `backend/app/infrastructure/rag/embeddings/local_hash.py` L4/L9/L61 |
| LLM smoke 门控 | `backend/tests/infrastructure/llm/test_llm_smoke.py` L21-23 |
| 测试直写开发库 | `backend/tests/conftest.py` L3/L31/L36/L40/L57/L160 |
| CI 配置 | `.github/workflows/ci.yml`（L18-22 触发；L29-50 lint；L52-157 test：服务容器/隔离库/5 CLI 钉定/迁移/全量 pytest） |
| 迁移链 | `backend/alembic/versions/`（11 个：`0abffdd9…` → `d7c4a9e2f1b8` head） |
| semgrep 容器包装器 | `tools/semgrep-docker/{semgrep.cmd,Dockerfile,README.md}` |
| 零命中核查 | `apply_patch|PatchModel|fix_agent|FixAgent`、`webhook|pull_request|oauth`、`TEST_DATABASE_URL`、`reports.py` 内 `csv|pdf|export|download` —— 全部零命中 |

## 10. 统计口径、分类计算和一致性校验

**计数单位**：验收矩阵中的一个功能行 = 一条功能记录（唯一编号 F-01…F-55）。

**口径规则**：
1. 一行内列举的多个子能力（如 F-46 补丁全家桶、F-50 GitHub 全家桶）按 1 个能力域计 1 项，子能力在行内说明，不拆分计数；
2. K 组工程质量 5 项为汇总类，**计入总数**并在组标题注明类别；
3. 同一功能在其他章节（§6/§8）出现时仅引用编号，不重复计数；
4. 状态五分类互斥（PASS / IMPLEMENTED_UNVERIFIED / PARTIAL / NOT_IMPLEMENTED / 环境注记不占独立分类）；"无法确认"类本轮为 **0 项**（全部 55 项均有本轮可追溯证据）。

**一致性校验（逐组累加）**：

| 状态 | 计数 | 明细 |
| --- | --- | --- |
| PASS | **44** | A7 + B9 + C6 + D4 + E5 + F3 + G5 + I1 + K4 |
| IMPLEMENTED_UNVERIFIED | **1** | F-27 |
| PARTIAL | **1** | F-37 |
| NOT_IMPLEMENTED | **9** | F-36、F-40、F-44、F-45、F-46、F-47、F-49、F-50、F-55 |
| **合计** | **55** | 44 + 1 + 1 + 9 = 55 = 矩阵行数（A7+B9+C6+D5+E5+F5+G8+H1+I3+J1+K5 = 55）✓ |

**与旧报告口径的差异声明**：旧报告（第二轮）声明"44 项功能记录"，但其矩阵物理行数与声明无法对齐（该口径缺口在恢复副本复核 Phase 24-B-4 中已确认为无法确认）。本报告不沿用任何旧数字，以本轮 55 行唯一编号清单为准，校验等式成立。

## 11. 最终审计结论及其限制

**总体结论**：项目在当前基线 `faae1336`（main，PR #1 已合并）上处于**后端主链成熟、CI 门禁全绿**的状态：认证→项目/仓库→扫描（5 扫描器）→Finding→Triage→Agent/工具→Workflow→只读报告的完整后端链路有真实实现且经本轮本地全量（968 passed，3 个失败全部为已知网络环境阻断）与当前基线 CI（978 passed / 0 failed）双重验证；Ruff/MyPy 全绿；迁移链完整在 head。旧报告的最高风险项（P1-01 main CI 红）已解决。剩余缺口集中在：RAG 生产摄取、审计持久化、semgrep 工具面、Fix/GitHub/UI/导出四段后半链、以及测试数据库隔离机制（F-55，本轮全量测试仍直写开发库运行）。

**结论限制**：
1. 真实 LLM 调用路径（F-27）本轮未验证（opt-in 未授权）；
2. 前端构建未执行（无业务代码）；
3. 本地 Trivy 三测被网络环境阻断，其 PASS 判定依据为当前基线 CI 真实运行，非本地运行；
4. 本轮测试在开发库 `ai_security` 上执行（项目现行机制，含 teardown 清理），测试数据残留风险见 F-55/P2-02；
5. 第二轮审计报告原件已丢失，现存恢复副本与本报告为独立文档；本报告所有结论不依赖该原件。

— 完 —
