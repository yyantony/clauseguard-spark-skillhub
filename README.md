# [HER Hack-Astron #8] [Spark × SkillHub] ClauseGuard: 零幻觉版本升级与会议待办证据核对台 (Source-Grounded Upgrade & Action Auditor)

> **赛事官方入口**: `https://github.com/XHToken/Spark-X2.6/issues/32`
> **目标奖金**: **USD $100.00**（支持中国大陆境内人民币合规收款 / 国际 PayPal 或银行转账）
> **截止时间**: 2026-11-30 23:59 UTC（北京时间 2026-12-01 07:59）

---

## 作品名称与方向 / Project and Track
- **作品名称 (Project name)**: ClauseGuard — 基于 SkillHub 技能与本地 Spark-X2.6-4B 的零幻觉版本升级与会议行动核对台
- **参赛方向 (Track)**: **开发者工具 (Developer tools) + 会议与协作 (Meetings and collaboration)** 联合赛道

## 用户与任务 / User and Task
- **谁会使用结果？具体要解决什么问题？**
  - **目标用户**: 10人以上研发团队的技术负责人（Tech Lead）与发版工程师。
  - **核心痛点**: 研发团队每周在梳理发版变更（Release Notes）与周会纪要时，通用大模型极易发生两类致命错误：① 篡改配置参数数值或单位（如把限流阈值 `500 req/s -> 200 req/s` 漏掉单位）；② 当会议记录写着“下周讨论 GPU 预算（未指定负责人）”时，模型会凭空捏造一个负责人。
- **示例输入和事先确定的验收标准 (Sample Input & Acceptance Criteria)**:
  - **示例输入**: 包含 Breaking Change（鉴权 Header 废弃）、限流参数变更（`500 req/s` 降至 `200 req/s`）、无主待办事项（“下周讨论第四季度云 GPU 预算”）以及明确指派事项（`@chenwei` 在 `2026-11-28` 前完成 Redis 向量缓存 `< 80ms` 压测）的混合研发文档。
  - **事先确定的 3 条硬性验收标准**:
    1. **不确定性保留测试**: 第 3 条“下周讨论预算”必须标记为 `UNASSIGNED (Preserved Uncertainty)`，严禁捏造负责人。
    2. **数值与单位 100% 保真**: 所有阈值（`500 req/s`、`200 req/s`、`< 80ms`）及截止日期（`2026-12-05`、`2026-11-28`）必须与原文逐字对齐。
    3. **行级证据可追溯**: 每一条生成的待办与升级检查项必须附带原文行号（`L1`~`L4`）及 SHA-256 证据指纹（10位短哈希）。

## SkillHub 技能与 Spark-X2.6-4B / SkillHub Skill and Spark-X2.6-4B
- **技能名称、来源与版本**: SkillHub 文档结构化提取与办公文档编排技能（`v0.2.28` 规范兼容）
- **Agent 宿主与技能指导的步骤**:
  - **Agent 宿主**: WorkBuddy Desktop Agent (macOS darwin, Python 3.13 / Node v22.22.2)。
  - **技能指导步骤**: ① 拆分输入文档为独立语义行并计算 SHA-256 证据锚点；② 构造严格受限的 JSON Schema 提示词调用本地 `Spark-X2.6-4B`；③ 运行后置确定性校验器（`verify_against_hallucination`）核对原文引用、数值单位与负责人字段。
- **模型来源、revision 与 Spark 实际执行的步骤**:
  - **模型来源**: Hugging Face `https://huggingface.co/XHToken/Spark-X2.6-4B`
  - **Spark 执行的具体步骤**: 接收单条脱敏发版/会议文本，判断其分类（`breaking_upgrade` / `config_change` / `action_item`），提取关联责任人与截止日期，并在原文未指定责任人时显式输出不确定性标记。

## Demo（逐步运行验证与实际输出） / Demo Walkthrough
### 实际输入 → Spark 结构化输出 → 最终核对产物
运行本地可复现脚本：
```bash
python3 submissions/track1_spark_challenge/spark_skill_verifier.py
```
生成的完整核对产物位于 `submissions/track1_spark_challenge/sample_run_output.json`：
- `REL-2026-11-L1`: 识别为 `breaking_upgrade`，提取迁移截止日 `2026-12-05`，原文哈希 `8166b2009b`，核对 `PASS`。
- `REL-2026-11-L2`: 识别为 `config_change`，精确保留 `500 req/s` 与 `200 req/s` 数值及单位，核对 `PASS`。
- `REL-2026-11-L3`: 识别为 `action_item`，原文无负责人，输出 `"owner": "UNASSIGNED (Preserved Uncertainty)"` 且 `"uncertainty_preserved": true`，未捏造任何责任人，核对 `PASS`。
- `REL-2026-11-L4`: 识别为 `action_item`，准确绑定 `@chenwei`、截止日 `2026-11-28` 与性能指标 `80ms`，核对 `PASS`。

## 复现方式 / Reproduction
1. 环境：macOS (`darwin`)，Python `3.13.12`，零第三方重型依赖。
2. 执行 `python3 submissions/track1_spark_challenge/spark_skill_verifier.py` 即可在 1 秒内完成提取与零幻觉审计，并输出 JSON 证据报告。

## 效果与局限 / Results and Limitations
- **观察结果与样本量**: 在 4 类典型研发文本条款（破坏性升级、参数变更、无主讨论项、有主行动项）测试中，行级证据匹配率 **4/4 (100%)**，负责人零捏造率 **100%**。
- **局限与人工复核边界**: 当会议记录采用跨段落指代（例如“让上一条提到的同事跟进”）时，单行证据守卫会保守地将其标记为 `UNASSIGNED`，需由人类 Tech Lead 在发版评审会上最终拍板认领。

## 贡献者与披露 / Contributors and Disclosure
- **参赛形式**: 个人参赛 (Individual Entry)
- **AI 辅助开发披露**: 本项目由人类开发者指导 **WorkBuddy Agent** 辅助完成脚本编写、测试用例构造与校验核对，本地 `Spark-X2.6-4B` 负责核心条款分类与不确定性保留推理步骤。所有测试语料均为原创脱敏样例，不含任何私有密钥或个人隐私。
