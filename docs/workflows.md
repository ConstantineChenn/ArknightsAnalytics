# 分析复现与数据更新

工作台默认读取已提交快照。以下命令用于主动重算或新增数据，**不是首次启动的前置步骤**。涉及重建时，先保留需要对照的历史批次；生成脚本可能覆盖对应的派生文件。

示例从仓库根目录执行`python`，请先激活虚拟环境，或替换为`.\.venv\Scripts\python.exe`（macOS / Linux为`.venv/bin/python`）。

## 离线分析

以下脚本读取已有文件，不需要重新访问数据来源网站：

| 分析任务 | 命令 | 主要输出 |
| --- | --- | --- |
| 选品决策证据 | `python scripts/build_decision_evidence.py` | `reports/generated/decision_evidence/` |
| 角色生命周期 | `python scripts/build_lifecycle.py` | `reports/generated/lifecycle/` |
| GMV与营销情景 | `python scripts/build_gmv_drivers.py` | `reports/generated/gmv/` |
| 渠道与SKU配置 | `python scripts/build_channel_strategy.py` | `reports/generated/channel_strategy/` |
| 商业数据决策 | `python scripts/build_business_decisions.py` | `reports/generated/business_decisions/` |
| SQL分析 | `python scripts/run_sql_analysis.py` | `reports/generated/sql_analysis_report.md` |
| 公开扩充层回放 | `python scripts/build_public_expansion.py --run-date 2026-09-12` | `reports/generated/public_expansion/2026-09-12/` |

需要重建基础分析、模拟ERP和商品维护产物时，依次执行：

```powershell
python scripts/run_pipeline.py
python scripts/build_erp_operations.py
python scripts/build_product_catalog.py
python scripts/build_operational_analytics.py
```

该流程会改写相应分析产物，包括模拟经营快照，建议在独立工作副本执行。`run_pipeline.py --use-fixture`使用测试输入，同样会生成派生结果；不要把fixture产物误作新增真实采集，也不要为打开已有界面而重跑管道。

### 单独复核退款修正

```powershell
python scripts/verify_erp_refund_fix.py --output reports/generated/erp_refund_verification
```

脚本读取历史快照，在内存中用相同输入与随机种子重建并比较结果，向指定目录写出证据。它不替换原始CSV，也不刷新默认看板。详细口径见[ERP核对与修正](erp_checks_workflow.md)。

## 可选采集与人工导入

采集依赖来源网站的公开可用性、登录状态或访问限制。按需选择来源，先查看脚本`--help`和来源协议；遇到访问限制时保留失败记录，不将空响应当作真实的零值。

| 入口 | 用途 |
| --- | --- |
| `scripts/collect_multiplatform.py` | 原有多平台公开快照采集入口 |
| `scripts/collect_bilibili_archive.py` | B站官号历史内容 |
| `scripts/collect_skland_strategy.py` | 森空岛攻略站公开搜索观察 |
| `scripts/collect_public_expansion.py` | 独立批次的商城、内容与元数据扩充 |
| `scripts/import_taobao_snapshot.py` | 从人工整理的CSV导入商品快照 |
| `scripts/export_sku_recapture_queue.py` | 导出固定商品的补采任务 |

淘宝导入字段、输入格式和固定商品复采步骤见[固定SKU追踪协议](fixed_sku_tracking_protocol.md)。采集与导入后应检查来源、去重、币种和角色关联，再重建对应分析层。详细数据规模见[数据来源](data_sources.md)。

### 问卷导入

通用结构化问卷与已整理的文本问卷使用不同入口，选择与实际格式相符的一项：

```powershell
# 将示例路径替换为自己的导出文件
python scripts/import_survey_responses.py path/to/survey_export.csv
python scripts/import_questionnaire_text.py path/to/questionnaire.txt
```

格式与字段要求见[用户调研协议](user_research_protocol.md)。`scripts/simulate_survey_responses.py`仅供模拟演示，不能覆盖真实回收批次的来源属性。

## 商业试点门户

试点问卷门户是独立服务，可与工作台同时运行：

```powershell
python scripts/run_pilot_portal.py --port 8767
```

- 本地入口：`http://127.0.0.1:8767/`；本地管理页：`/admin`。
- 渠道来源可通过`?source=bilibili`等参数记录，来源参数不等于平台已验证的流量归因。
- 运行数据写入`data/manual/commercial_pilot/pilot_capture.db`，该私有运行文件不进入Git。
- 导出与备份使用`scripts/export_pilot_capture.py`和`scripts/backup_pilot_capture.py`，参数及执行步骤见[试点执行手册](commercial_pilot_runbook.md)。

该门户目前面向本地试点准备。公开部署前需独立处理身份、访问控制、数据告知及运行验收；候选、意向、付款和履约应分别记录。

## 主要输出

| 内容 | 文件 |
| --- | --- |
| 基础需求与商业信号 | [综合分析报告](../reports/generated/analysis_report.md)、[B站历史内容](../reports/generated/bilibili_archive_report.md)、[淘宝商品观察](../reports/generated/taobao_commerce_report.md) |
| 用户研究与选品 | [用户调研报告](../reports/generated/user_research_report.md)、[选品案例](../reports/generated/selection_case_study.md)、[决策证据](../reports/generated/decision_evidence/report.md) |
| 商品维护 | [商品资料与维护清单](../reports/generated/product_catalog_maintenance_report.md) |
| ERP与经营分析 | [ERP报告](../reports/generated/erp_operations_report.md)、[SQL分析](../reports/generated/sql_analysis_report.md)、[经营分析](../reports/generated/operational_analytics_report.md) |
| 专项决策 | [生命周期](../reports/generated/lifecycle/report.md)、[GMV](../reports/generated/gmv/report.md)、[渠道](../reports/generated/channel_strategy/report.md)、[商业数据决策](../reports/generated/business_decisions/report.md) |
| 工作簿与数据库 | [运营工作簿](../reports/generated/operations_dashboard.xlsx)、[经营分析工作簿](../reports/generated/operational_analytics.xlsx)、[分析数据库](../reports/generated/operations.db) |
| 独立批次与试点 | [公开扩充报告](../reports/generated/public_expansion/2026-09-12/report.md)、[商业试点报告](../reports/generated/commercial_pilot_report.md) |

报告说明当前输入下的分析结论。来源更新后，应明确重跑范围并保留版本；生成成功不代表数据已被人工核验或建议已执行。
