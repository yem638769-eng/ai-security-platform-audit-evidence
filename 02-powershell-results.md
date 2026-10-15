> **公开发布说明（本块为脱敏发布信息，非原始证据内容）**
> - 本文件是私有仓库 `yem638769-eng/ai-security-platform` 证据分支 `audit/evidence-2026-10-15`（证据提交 `92ac44e`）中 `docs/audit-evidence/2026-10-15/02-powershell-results.md` 的脱敏公开副本。
> - 原始文件 SHA-256：`E25435C7ED1E6D278C2B228A84581A7EDAE8D7CE1F250A48CB450EA0607F5D89`（本副本以字节级复制生成，另加本说明块）。
> - 脱敏替换：无——公开前扫描未发现凭据、个人路径或内部地址，正文未做任何替换。
> - 生成时间：2026-10-15 17:30（Asia/Shanghai）。以下为原始证据正文。

# Phase 24-C 只读验收结果

- 采集时间：2026-10-15 17:09:44 +08:00
- 仓库：yem638769-eng/ai-security-platform
- 当前分支：main
- HEAD SHA：faae13363684fb1551231f7bd1b49e0ed2606802
- 本地 origin/main SHA：faae13363684fb1551231f7bd1b49e0ed2606802
- HEAD 与 origin/main 一致：True

## Git 状态（归档前实测）
```text
?? .github/workflows/ci.yml.phase18a-backup
?? .github/workflows/ci.yml.phase18b-backup
?? backend/app/application/triage/code_context.py.phase18-path-backup
?? backend/tests/infrastructure/scanners/test_cli_integration.py.phase18b-diagnostic-backup
?? docs/reports/
```

## 历史报告与本轮报告
| 文件 | 存在 | 大小（字节） | 最后修改时间 |
|---|---:|---:|---|
| FULL_PROJECT_AUDIT_2026-10-10.md | 是 | 43625 | 2026-10-15 14:41:55 |
| FULL_PROJECT_SECOND_AUDIT_2026-10-10.recovered.md | 是 | 32905 | 2026-10-14 15:06:13 |
| FULL_PROJECT_SECOND_AUDIT_2026-10-10.md | 否 | — | — |
| FULL_PROJECT_REAUDIT_2026-10-15.md | 是 | 26072 | 2026-10-15 16:51:37 |

## 四个备份文件（只读检查）
- 存在：.github\workflows\ci.yml.phase18a-backup；大小 5035 字节；修改时间 2026-10-09 18:46:56
- 存在：.github\workflows\ci.yml.phase18b-backup；大小 5189 字节；修改时间 2026-10-09 19:10:48
- 存在：backend\app\application\triage\code_context.py.phase18-path-backup；大小 11453 字节；修改时间 2026-10-09 22:19:47
- 存在：backend\tests\infrastructure\scanners\test_cli_integration.py.phase18b-diagnostic-backup；大小 5286 字节；修改时间 2026-10-06 10:16:54

## 本轮审计报告 SHA-256
- 路径：docs/reports/FULL_PROJECT_REAUDIT_2026-10-15.md
- 大小：26072 字节
- SHA-256：76CEB703B0139DB6EE2E014AEB95394510B5514ABCB6A33A12F5CA2B61EBA6A7

## 报告关键项定位（仅用于导航，完整内容见独立报告副本）
34: ## 3. 功能实现验收矩阵（55 项，唯一编号 F-01…F-55）
41: > **ENV 注记** = 本地环境阻断，作为 PASS 行的附加说明（不改变主状态，CI 证据为判定依据）。
107: | F-36 | RAG 摄取生产入口 | **不存在**：`main.py` 仅注册 8 组 router（L21-28，无 knowledge）；ingest 逻辑仅存在于服务层（`application/rag/service.py`），无 API/CLI 入口 | 路由注册核查 + 符号检索（本轮实测） | **NOT_IMPLEMENTED** |
116: | F-40 | semgrep 工具注册 | **不存在**：`registry.py` 仅 bandit_scan/gitleaks_scan/trivy_scan（本轮全文读取核实） | 注册表核查 | **NOT_IMPLEMENTED** |
120: | F-44 | 审计持久化 | **不存在**：11 个迁移无审计表（本轮迁移清单全量核对）；audit.py 仅内存/日志 sink | 迁移链核查 | **NOT_IMPLEMENTED** |
127: | F-46 | Fix Agent（补丁生成/差异模型/应用/回滚/隔离测试/审批/验证——作为 1 个能力域计 1 项） | **不存在**：`apply_patch|PatchModel|fix_agent|FixAgent` 全后端零命中（本轮检索） | 符号检索 | **NOT_IMPLEMENTED** |
133: | F-47 | 前端业务页面 | `frontend/src/` 仅 3 文件（App.vue/main.ts/style.css，本轮文件树核实） | 文件树核查 | **NOT_IMPLEMENTED** |
135: | F-49 | 报告导出（PDF/CSV 等） | **不存在**：`reports.py` 内 `csv|pdf|export|download` 零命中（本轮检索） | 符号检索 | **NOT_IMPLEMENTED** |
141: | F-50 | GitHub 集成（OAuth/webhook/PR 回写/仓库读取/凭据管理——作为 1 个能力域计 1 项） | **不存在**：`webhook|pull_request|oauth` 全后端零命中（本轮检索） | 符号检索 | **NOT_IMPLEMENTED** |
151: | F-55 | 测试数据库隔离 | **不存在**：`TEST_DATABASE_URL` 全仓库零命中（本轮检索）；`backend/tests/conftest.py` L3 自述 "Tests run against the Docker Compose PostgreSQL (ai_security database)"，L31 直连应用 SessionLocal | conftest 全文机制核查（见 §4 隔离核查） | **NOT_IMPLEMENTED** |
160: | **本地全量套件** | `.\.venv\Scripts\python.exe -m pytest -q` | **968 passed / 3 failed / 14 skipped**（985 收集，exit 1） | **139.88s**，2026-10-15 16:45:04–16:47:24 |
161: | CI @ 当前基线 | Run 38207885778（push main `faae1336`，2026-10-12） | lint job **success**（37s）；test job **success**：**978 passed / 0 failed / 7 skipped / 244 warnings**（96.51s） | 本轮 gh 实测（job 级 + 日志摘要） |
163: **本地 3 个失败明细（全部环境阻断类，与代码无关的判定依据：CI 同测试通过）**：
175: - 判定：测试**会写开发库**，但仅创建并回收自有测试实体；无清库/重置/破坏性操作；已知风险 = 测试中断可能残留孤儿行（F-55，P2 级）。本轮全量套件正常结束（exit 1 为断言失败非中断），清理机制全程运转。
193: ## 6. 未实现功能与开发缺口（9 项，对应矩阵 NOT_IMPLEMENTED）
197: | F-36 | RAG 摄取生产入口 | `main.py` 8 组 router 注册核查；ingest 仅服务层 | RAG 检索测试全绿但生产无数据来源 |
198: | F-40 | semgrep 工具注册 | `registry.py` 全文读取（恰 3 工具） | Agent/Workflow/MCP 无法调用 semgrep |
199: | F-44 | 审计事件持久化 | 11 迁移清单核对（无审计表）；audit.py 仅内存/日志 sink | 平台自身操作无留存轨迹 |
201: | F-46 | Fix Agent/补丁/验证 | `apply_patch|PatchModel|fix_agent|FixAgent` 零命中 | "修复→验证"后半链不存在 |
202: | F-47 | 前端业务页面 | `frontend/src/` 文件树（3 个脚手架文件） | 无 UI 展示 |
203: | F-49 | 报告文件导出 | `reports.py` 导出符号零命中 | 仅 JSON DTO |
204: | F-50 | GitHub 产品级集成 | `webhook|pull_request|oauth` 零命中 | 无交付闭环 |
205: | F-55 | 测试数据库隔离 | `TEST_DATABASE_URL` 零命中 + conftest L3/L31 | 本地测试直写开发库（本轮全量套件即在此机制下运行） |
209: ## 7. 环境阻断及未完成验证（与代码缺陷严格分开）
213: | 本地 Trivy 3 测试失败 | **环境阻断**（GHCR 下载被本机网络阻断），非代码缺陷 | 本轮本地 3 failed 错误均为 "failed to download vulnerability DB: OCI artifact error"；CI Run 38207885778 同测试通过（978p/0f） | 未伪装为 PASS；未归咎代码；修复方向 = 为 trivy 配置可达代理（P2-01） |
215: | 前端构建 | **未执行**（仅 3 个脚手架文件，无业务代码可构建） | 文件树核查 | F-47 记 NOT_IMPLEMENTED（与构建成败无关） |
225: | P2 | P2-02 | 测试直写开发库 | F-55 | TEST_DATABASE_URL 契约 + conftest 导入前守卫（fail-closed）；CI 已隔离不受影响 |
226: | P2 | P2-03 | RAG 无摄取入口 | F-36 | 摄取 API（需产品决策：权限/限额/去重/异步解析） |
227: | P2 | P2-04 | 审计无持久化 | F-44 | 审计表 + 迁移 + 查询 API（延续 append-only + 脱敏契约） |
228: | P2 | P2-05 | semgrep 未注册为工具 | F-40 | 注册第 4 工具，复用扫描器 + ToolExecutor 契约 |
253: | 零命中核查 | `apply_patch|PatchModel|fix_agent|FixAgent`、`webhook|pull_request|oauth`、`TEST_DATABASE_URL`、`reports.py` 内 `csv|pdf|export|download` —— 全部零命中 |
257: **计数单位**：验收矩阵中的一个功能行 = 一条功能记录（唯一编号 F-01…F-55）。
260: 1. 一行内列举的多个子能力（如 F-46 补丁全家桶、F-50 GitHub 全家桶）按 1 个能力域计 1 项，子能力在行内说明，不拆分计数；
272: | NOT_IMPLEMENTED | **9** | F-36、F-40、F-44、F-45、F-46、F-47、F-49、F-50、F-55 |
279: **总体结论**：项目在当前基线 `faae1336`（main，PR #1 已合并）上处于**后端主链成熟、CI 门禁全绿**的状态：认证→项目/仓库→扫描（5 扫描器）→Finding→Triage→Agent/工具→Workflow→只读报告的完整后端链路有真实实现且经本轮本地全量（968 passed，3 个失败全部为已知网络环境阻断）与当前基线 CI（978 passed / 0 failed）双重验证；Ruff/MyPy 全绿；迁移链完整在 head。旧报告的最高风险项（P1-01 main CI 红）已解决。剩余缺口集中在：RAG 生产摄取、审计持久化、semgrep 工具面、Fix/GitHub/UI/导出四段后半链、以及测试数据库隔离机制（F-55，本轮全量测试仍直写开发库运行）。
284: 3. 本地 Trivy 三测被网络环境阻断，其 PASS 判定依据为当前基线 CI 真实运行，非本地运行；
285: 4. 本轮测试在开发库 `ai_security` 上执行（项目现行机制，含 teardown 清理），测试数据残留风险见 F-55/P2-02；

## 本次操作边界
- 本记录保存归档前的只读检查结果。
- 本脚本不运行测试、不连接数据库、不修改业务代码。
- 本脚本只将本记录与本轮审计报告副本加入独立审计分支。
- 不提交旧报告、恢复副本或四个备份文件。
- 原始项目报告保留在原位置，不覆盖、不移动。
