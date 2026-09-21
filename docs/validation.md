# Validation Baseline

验证日期：**2026-09-21**。代码基线：**`d83465d`**（公开`main`的应用代码）。本次README整理未修改应用逻辑，验证在该基线的独立工作目录中完成，未混入本地尚未发布的功能。

| 检查 | 结果与范围 |
| --- | --- |
| 环境 | Windows、Python 3.12.14、pnpm 11.19.0 |
| Python测试 | `262 passed, 722 warnings in 85.55s` |
| 语句覆盖率 | 3,637 / 4,443 = **81.86%**；未开启分支覆盖率统计 |
| 前端安装 | `pnpm install --frozen-lockfile`通过 |
| 前端构建 | `pnpm build`通过 |
| 页面冒烟检查 | 使用Playwright与本机Edge加载运营总览、玩家调研；页面标题可见，未捕获`pageerror` |
| 截图 | 首页中的两张图片来自该代码版本的本地工作台 |

722条warning保留在pytest输出中，不计作测试失败，也不应描述为“零警告”。覆盖率仅表示本次测试执行到的Python语句比例，不衡量数据代表性或经营策略有效性。

## 复现命令

在仓库根目录、已安装依赖后执行：

```powershell
.\.venv\Scripts\python.exe -m pytest --cov=arknights_merch_analytics --cov-report=term
pnpm --dir frontend install --frozen-lockfile
pnpm --dir frontend build
```

测试覆盖范围包括分析计算、数据质量规则、商品维护、任务处理、API与试点契约；具体用例见[`tests/`](../tests)。[GitHub Actions](https://github.com/ConstantineChenn/ArknightsAnalytics/actions/workflows/ci.yml)持续运行Python测试、前端构建及fixture管道。

## 未包含在本轮验证中的事项

- 完整Docker镜像构建和容器联调；现有指南中的Compose配置校验不能替代它。
- 全页面、全设备的交互回归；两页冒烟检查不等于端到端测试集。
- 全量在线重新采集、真实付款与履约、线上营销实验。
- 生产级认证、权限隔离或多人审批。

历史文档中的149、165、186项测试以及79.82%覆盖率属于较早版本，不与本轮数字累计。
