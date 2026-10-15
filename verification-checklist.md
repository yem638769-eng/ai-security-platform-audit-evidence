# 公开证据独立核对清单（Verification Checklist）

> 核对时间：2026-10-15 17:30（Asia/Shanghai）
> 核对人：AI 审查助手（Trae Code / GLM 模型）
> 核对对象：本仓库 `02-powershell-results.md` 与 `03-reaudit-report.md`（公开发布版）

## 1. 审查范围

- 对私有仓库 `yem638769-eng/ai-security-platform` 证据分支 `audit/evidence-2026-10-15`（证据提交 `92ac44e`）归档的 Phase 24-C 审计证据进行独立核对；
- 核对维度：证据文件完整性、基线提交、测试统计、失败原因、功能矩阵锚点、已知缺口、统计口径；
- 定位为抽样复核关键声称的可复核性，不重复执行完整审计。

## 2. 证据来源

| 来源 | 说明 |
| --- | --- |
| 证据提交 `92ac44e` | 私有仓库 `audit/evidence-2026-10-15` 分支 HEAD，树内恰含两份证据文件 |
| 原始审计报告 | 私有仓库 `docs/reports/FULL_PROJECT_REAUDIT_2026-10-15.md` |
| 本地全量测试日志 | 2026-10-15 本地 pytest 运行输出（exit 1，3 failed 均为 Trivy） |
| CI 记录 | GitHub Actions Run 38207885778（`gh` CLI 实测 + 日志摘要行） |
| 当前代码 | 私有仓库 `main` 分支 `faae1336`（与证据基线一致） |

## 3. 逐项核对结果

| # | 证据声称 | 核对方法（独立执行） | 结果 |
| --- | --- | --- | --- |
| V-01 | 基线 main = `faae13363684fb1551231f7bd1b49e0ed2606802` = origin/main | `git rev-parse main origin/main` 实测 | ✅ 一致 |
| V-02 | 证据提交 `92ac44e` 存在且含两份证据文件 | `git cat-file -t` + `git ls-tree -r` + 工作区全文读取（82 行 / 288 行，无截断） | ✅ 存在，完整可读 |
| V-03 | `03-reaudit-report.md` 为原始报告副本 | SHA-256 对比：证据副本与原始报告均为 `76CEB703…BA6A7`，且与 02 文件中预先记录的哈希一致 | ✅ 逐字节一致 |
| V-04 | 本地全量 968 passed / 3 failed / 14 skipped（985 收集，139.88s） | 本地测试日志末行复核："3 failed, 968 passed, 14 skipped in 139.88s" | ✅ 一致 |
| V-05 | 3 个失败全部为 Trivy GHCR 网络阻断 | 日志失败明细：3 个测试均为 "failed to download vulnerability DB: OCI artifact error" | ✅ 一致 |
| V-06 | CI Run 38207885778 @ `faae1336` 双 job success | `gh run view`：completed / success，Lint 与 Test 两 job 均 success，headSha `faae1336` | ✅ 一致 |
| V-07 | CI test job 978 passed / 0 failed / 7 skipped / 244 warnings（96.51s） | CI 日志摘要行："978 passed, 7 skipped, 244 warnings in 96.51s"；job success ⟹ 0 failed | ✅ 一致 |
| V-08 | `main.py` 恰 8 组 router，无 knowledge 路由（F-36） | 当前代码全文读取 | ✅ 一致 |
| V-09 | `registry.py` 恰 3 工具 bandit/gitleaks/trivy（F-40） | 当前代码全文读取 | ✅ 一致 |
| V-10 | `apply_patch\|PatchModel\|fix_agent\|FixAgent` 后端零命中（F-46） | 当前代码符号检索 | ✅ 零命中 |
| V-11 | `webhook\|pull_request\|oauth` 后端零命中（F-45/F-50） | 当前代码符号检索（不区分大小写） | ✅ 零命中 |
| V-12 | `TEST_DATABASE_URL` 代码/配置零命中（F-55） | 当前代码检索（仅审计报告文档自身提及该词） | ✅ 零命中 |
| V-13 | 11 个迁移、无审计表（F-44/F-53） | `backend/alembic/versions/` 清点 = 11 个文件；迁移内容检索 "audit" 仅现于注释文字，无审计表创建 | ✅ 一致 |
| V-14 | `frontend/src/` 仅 3 文件（F-47） | 文件树清点 | ✅ 恰 3 文件（App.vue / main.ts / style.css） |
| V-15 | 统计口径 44 PASS + 1 IMPLEMENTED_UNVERIFIED + 1 PARTIAL + 9 NOT_IMPLEMENTED = 55 | 报告 §10 互斥校验 + 矩阵编号 F-01…F-55 连续性核查 | ✅ 等式成立 |

## 4. 声称分类

- **有证据支持通过**：V-01～V-15 覆盖的基线、本地/CI 测试统计、失败原因、矩阵锚点、统计口径声称；
- **报告声称、核对窗口未独立重跑**：本地 Ruff / MyPy / Alembic 结果（有前序执行窗口实测记录，且 CI lint job success 佐证 Ruff/MyPy）；
- **已实现但未验证**：F-27 真实 LLM 调用路径（报告自记 IMPLEMENTED_UNVERIFIED）；
- **部分实现**：F-37 Embedding 语义能力（哈希基线在，语义 provider 缺）；
- **尚未实现**：F-36、F-40、F-44、F-45、F-46、F-47、F-49、F-50、F-55（报告自记 NOT_IMPLEMENTED，其中 8 项锚点经本清单复验）。

## 5. 未验证事项

1. 真实 LLM smoke 测试（opt-in 门控 `RUN_LLM_SMOKE_TESTS=1`，未经授权不发起）；
2. 前端构建（仅 3 个脚手架文件，无业务代码可构建）；
3. 本地 Trivy 三个测试的通过性（本机网络阻断 GHCR 漏洞库下载；报告对其 PASS 判定依据为 CI 真实运行，非本地运行）；
4. 本地 Ruff / MyPy / Alembic 未在核对窗口重跑（以证据文件记录 + CI 记录为准）；
5. 44 项 PASS 未逐项定向重测（以全量套件日志 + CI 记录为整体证据）；
6. 旧报告（2026-10-10 两轮）内容未纳入本次核对范围（历史资料，非本次发布证据）。

## 6. 脱敏记录

| 文件 | 位置 | 原始内容 | 处置 |
| --- | --- | --- | --- |
| `03-reaudit-report.md` | §1 环境表 gh CLI 行 | 本机代理地址（本地回环地址与端口） | 替换为 `[本地代理地址已脱敏]`；属内部网络配置，不影响审计结论 |
| `02-powershell-results.md` | 全文 | — | 未发现凭据、个人路径或内部地址，正文未替换 |

失败记录保留声明：本地 3 个 Trivy 失败的错误类型与脱敏后摘要（"failed to download vulnerability DB: OCI artifact error"）在公开副本中完整保留，未因脱敏删除任何影响审计结论的失败记录。
