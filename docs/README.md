# 企业内部办公 Agent · 项目结论

> 状态：**已收敛** · 最后更新：本次评估后
> 本目录只保留结论性文档。历史设计稿见 `archive/`，**均已失效，不得引用**。

---

## 一、结论

**产品已经做完了。** 载体是 `<workspace>\AI-DeskTop`（企业 AI 工作台 v0.4.0）。

| 项 | 状态 |
|---|---|
| 代码 | 约 **75,000 行**（桌面端 32,162 / 服务端 17,810 / 测试 17,180 / 管理后台 7,411） |
| 测试 | **289 项 pytest + 79 项 vitest 全绿**，ruff / mypy 通过 |
| 数据库 | MySQL，**32 个 Alembic 迁移**已到 head |
| 部署 | **已上线**：`ai.example.com`（生产）/ `dev-ai.example.com`（测试），健康检查 200 |
| 真实链路 | DeepSeek + Moonshot 已配置，SSE 对话 + usage 计费已验证；企微会议返回真实数据 |
| 设计文档 | `AI-DeskTop/docs/design/01~18` + 实现状态表 + ADR-001~014 |

**范围已确认**：
- ❌ 不做员工 Web 端（原架构即 Python 桌面端，Web 无必要）
- ❌ 不做移动端 / 企微 H5（原系统未规划，维持）
- ⏸ SQLServer 只读连接器 —— **暂缓，后续使用时再做**

**因此当前无待开发项。**

---

## 二、剩余事项（均为外部条件，不写代码）

来自 `AI-DeskTop/docs/PROJECT_HANDOFF.md` §9：

| # | 事项 | 阻塞点 |
|---|---|---|
| 1 | 企微通讯录目录同步 | 回调未配置，**HTTPS 域名待处理** |
| 2 | 真实企微登录联调 | 需生产域名 + 回调 |
| 3 | MaxKB 建库 / 上传 / 索引管理 | 需公司**精确版本 Swagger + 系统 API Key** |
| 4 | 邮件 Graph OAuth / 企微应用邮箱 | 需应用注册与测试租户 |
| 5 | SMTP 发送现场验证 | 需测试邮箱 |
| 6 | 外网 HTTPS、会议预定实测 | 运维 / 验证 |
| 7 | FR-SYS-002 源系统权限对齐（行级权限） | 需设计：用户委托 Token / 行级策略 |

**唯一需要设计思考的是第 7 项**——要保证 Agent 查到的数据不超过该用户在源系统里本就能看到的范围。

---

## 三、暂缓项的备忘（SQLServer 连接器）

启动时的落点，避免重新调研：

- 文件：`platform_api/connectors.py`
- 现状：只支持 `http/https` 与 `mysql+pymysql`，后者**硬编码**
- 需改动：L521 驱动白名单、L568 凭据校验、L706 **`SELECT /*+ MAX_EXECUTION_TIME() */`（MySQL 专有提示，需按方言分支）**、L669 行数限制、L787 标识符引用
- 管理端：`admin-web/src/pages/ConnectorsPage.tsx` 增加类型选项
- 估算：**1~1.5 人周**
- 建议：抽出 `SqlDialect` 协议而非加 `if` 分支，便于后续扩展达梦 / 人大金仓等国产库
- **红线**：只生成已批准视图/字段/等值筛选/行数上限的 `SELECT`，禁止写入，客户端不直连生产库，复用现有一次性授权令牌与审计链路

---

## 四、文档说明

| 目录 | 内容 | 可用性 |
|---|---|---|
| `12-AI-DeskTop复用评估.md` | 本次评估的完整过程与依据 | ✅ 有效 |
| `archive/v1-自主设计稿/` | 11 份自主设计文档（01~11） | ❌ 已被现实推翻 |
| `archive/v0-讨论稿/` | 16 份早期讨论稿 | ❌ 已作废 |

**为什么归档 `01`~`11`**：它们写于「不知道 AI-DeskTop 存在」的前提下，与 `AI-DeskTop/docs/design/` 重复，且部分结论（PostgreSQL、Electron、自建 Web 端）已被现实推翻。保留会造成两套文档打架。

**若将来需要做手机端**：`archive/v1-自主设计稿/08-客户端规划.md` §4 的企微内置浏览器限制清单与移动端设计原则，仍可参考。

---

## 五、本项目结论

> 这轮工作的最大价值不是「省了多少代码」，而是**避免了重复造一个更差的轮子**。
>
> 从「17.5 人周开发一个新系统」收敛到「**无待开发项**」，只需要先确认一件事：**要做的这个东西是不是已经存在了。**

---

## 六、开发方式（已定）

**在 `<workspace>\AI-DeskTop` 原仓库继续开发。不复制、不开新仓库。**

| 事实 | 值 |
|---|---|
| Git 工作区 | **干净**（无未提交变更） |
| 远程 | `http://203.0.113.10/user/ai-desktop.git`（公司内网 GitLab） |
| 跟踪文件 | 360 个，顶层结构清晰 |
| 最新提交 | `6cc8959` 2026-09-11 |
| Tag | `worktree-snapshot-20260827` |

**为什么不在 `super-agent` 复制一份**：
1. 会分裂 Git 历史与协作（远程已在用）
2. 部署脚本、镜像标签、1Panel 配置、文档中的绝对路径均指向原仓库
3. 289 pytest + 79 vitest 的测试基线在本地已跑通，换目录要重建
4. 两套仓库必然漂移

**如确需换目录：用 `git clone`，不要复制文件系统**（保留历史、远程与 tag）。

### 该先做的三件事

| # | 事项 | 说明 |
|---|---|---|
| 1 | **清理本地垃圾（约 4~5 GB）** | 307 个 `tmp*`、61 个 `.pytest-tmp*`（约 2.2GB）、`dist/` 647MB、`build/` 83MB、`.mypy_cache/` 113MB。**已验证全部未被 git 跟踪，删除安全**。⚠️ 例外：`runtime-data/platform.db` 是本地开发库，删前确认 |
| 2 | **打版本 tag** | 当前只有 `worktree-snapshot-20260827`。建议打 `v0.4.0` 推 GitLab，与部署镜像标签对应，作为试点前可回滚基线 |
| 3 | **`super-agent` 的去向** | 与 AI-DeskTop 是两套东西。建议把本文 §三、§六 的有效结论合并进 `AI-DeskTop/docs/`，之后 `super-agent` 仅作立项记录保留或删除。**不要试图把两者合成一个仓库** |

### 已执行：清理本地垃圾（释放约 2.6 GB）

**已删除 378 项**，`git status` 仍干净、跟踪文件数仍为 360（源码零影响）：

| 项 | 数量 | 大小 |
|---|---|---|
| `tmpXXXXXXXX` | 307 | 157 MB |
| `.pytest-tmp*` / `.pytest-agent-*` | 66 | 1,626 MB |
| `.pytest_cache` / `.mypy_cache` / `.ruff_cache` | 3 | 113 MB |
| `build/` | 1 | 83 MB |
| `dist/` | 1 | 647 MB |

> ⚠️ **`dist/EnterpriseAIWorkbench`（已打包的桌面客户端）已一并删除**，分发前需重新打包。
> 保留未动：`runtime-data/`（含本地开发库 `platform.db`）、`.venv/`、`admin-web/node_modules/`。

### ⏳ 待办：堵住泄漏源（否则下周又长回来）

**根因**（已定位）：

| 泄漏源 | 位置 | 问题 |
|---|---|---|
| `TemporaryDirectory(dir=".")` × 6 | `tests/test_ui_smoke.py:1264,1333,1417,1451,1497,1614` | `dir="."` = 仓库根目录；Windows 上清理 `client.db-shm/-wal` 失败时只发 `ResourceWarning`，目录留在原地 |
| `mkdtemp(dir=".")` × 1 | `tests/test_knowledge_page_ui.py:111` | **`mkdtemp` 从不自动清理**，用一次漏一个 |
| `--basetemp` 每次换名字 | `README.md:346` 推荐 `--basetemp .pytest-tmp`，实际用了 `iter2`…`iter113`、`full-00xx`、`e2e2/3/4` 等 | 每换一次名字就永久留一份 → 66 个残留 |

**为什么一直没人发现**：`tmp*` 目录本身没被 `.gitignore` 忽略，但目录里唯一的文件（`*.db` / `*.db-*`）全被忽略，而 **git 不跟踪空目录** → `git status` 永远干净。

**修复动作**：
1. 7 处 `dir="."` → 改用 pytest 的 `tmp_path` fixture 或 `TemporaryDirectory(prefix="aiw-")`（走系统临时目录）
2. Windows 清理失败兼容：`ignore_cleanup_errors=True` + 用 fixture 确保先关 SQLite 连接
3. `--basetemp` 固定为一个名字，每次跑前清空
4. **CI 加一道检查**：仓库根目录出现 `tmp*` / `.pytest-tmp-*` 即失败

---

## 七、数据库决策：继续用 MySQL

**结论：维持 MySQL，不换 PostgreSQL。**

### 7.1 换 PG 的原始动机已经不存在

我们早期倾向 PostgreSQL 的**唯一决定性理由是 pgvector**（自建 RAG 向量检索）。

但 AI-DeskTop 的 ADR-009 已定：**正文 / 分段 / 向量由 MaxKB 管理，本项目不复制其 RAG 引擎。**

> **理由消失 → 换库失去依据。**

### 7.2 三条工程事实

| # | 事实 | 位置 |
|---|---|---|
| 1 | 生产环境**代码里硬校验** MySQL：必须 `mysql+pymysql://` 且带 `charset=utf8mb4` | `platform_api/config.py`（`is_production` 分支） |
| 2 | **31 个 Alembic 迁移**已在 MySQL 上跑通到 head | `migrations/versions/` |
| 3 | staging / production **双环境已部署**，production 有真实数据 | 见 §一 |

**「之前用了而已」不成立**——它不是历史遗留，是写死在启动校验里的约束。

### 7.3 换库的真实成本

| 项 | 位置 | 影响 |
|---|---|---|
| 生产 URL 校验 | `config.py`（`mysql+pymysql://` + `charset=utf8mb4`） | 必改 |
| 连接池参数 | `database.py` L29-35（MySQL 专属 `pool_recycle` / `isolation_level`） | 必改 |
| 企业连接器白名单 | `connectors.py` L521/L568、`schemas.py` L1608 | 必改 |
| **31 个迁移** | 12 个用 `server_default`、8 个涉及 `TEXT/LONGTEXT`、11 个用 `batch_alter_table` | 需逐个复核方言 |
| 测试基线 | 289 项 pytest | 全量重跑 |
| 已上线数据 | production MySQL | 迁移风险 |
| **收益** | — | **0** |

**一个好消息**：迁移已用 `batch_alter_table` 做了 SQLite/MySQL 双兼容（11 个文件），说明**不是 MySQL 深度绑定**，而是「已支持两种方言」——再加第三种要花力气，但可行。

**结论**：换库 = 花钱买零收益 + 引入迁移风险。**不做。**

### 7.4 顺带该解决的真实痛点

你们自己的风险表 **R-014** 已写明：

> 「SQLite 开发掩盖 MySQL 问题 | 概率 **高** | 影响 **高** | 缓解：dev/test 早期 MySQL CI」

且已踩到具体坑（`PROJECT_HANDOFF.md` §10）：
- 「MySQL TEXT 列不能 server_default」
- 「development 环境 create_schema 与迁移混用时，新表迁移必须幂等（`_table_exists` 检查）」

**建议落实 R-014 的缓解措施**：本地 dev/test 也用 MySQL（容器），或至少 **CI 用 MySQL 跑全量 pytest**。

> 这比纠结换不换库有价值得多。

### 7.5 顺手做：把方言收敛到一处

现在 MySQL 相关逻辑散在三个文件：`config.py`（URL 校验）、`database.py`（池参数）、`connectors.py`（`MAX_EXECUTION_TIME` 提示）。

建议抽 `platform_api/db/dialect.py`，集中管理：

- 生产 URL 校验与 charset 要求
- 连接池参数
- 分页语法（`LIMIT` vs `TOP`）
- 查询超时写法
- 标识符引用规则（反引号 vs `[]`）

**两个好处，都不需要现在换库**：
1. 将来加 **SQLServer 连接器**时只在此扩展（§三 已备忘）
2. 万一哪天真要换库，改动面从「全项目」缩到「一个文件」

### 7.6 「MySQL 不是必须」的正确用法

需求方的原意应是**不被它绑住**。正确落法：

| | |
|---|---|
| ✅ | **同一平台内不引入第二个业务数据库**（跨库事务/同步是无底洞） |
| ✅ | **新模块可自由选型**——如审计/遥测数据量大时，单独用分析型库，不与主库混用 |
| ✅ | **不让方言渗透业务代码**（靠 ORM + 方言层隔离） |
| ❌ | 不是「现在应该换成 PG」 |

### 7.7 什么时候才真的该换（触发条件）

目前**一条都不成立**：

| 触发条件 | 现状 |
|---|---|
| 需要 PG 原生 **RLS** 做行级权限（FR-SYS-002） | 当前是应用层判定，够用 |
| 需要 **pgvector** 做邮件/本地语义检索 | 更该复用 MaxKB（已有） |
| 需要 JSONB 高级索引 | MySQL 8 的 JSON + 函数索引够用 |
| 公司信创要求国产库 | MySQL 系亦有替代（TiDB / OceanBase / GoldenDB） |

