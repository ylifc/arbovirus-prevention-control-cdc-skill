# Arbovirus Prevention and Control CDC Skill

虫媒病毒防控 CDC Skill · v0.3.0

结合流调、历史病例及气象数据，对照小区、职业和活动时段，区分输入与本地情景，依据病例和现场证据研判传播风险。

本仓库提供独立程序压缩包。下载 `arbovirus-prevention-control-cdc-skill-0.3.0.zip` 并解压，即可得到完整源码、规则、合成示例、测试和说明。可用同目录的 `SHA256SUMS.txt` 核对下载文件；包内 `MANIFEST.sha256` 可核对每个文件。

## 包含的模块

| 目录 | 用途 |
| --- | --- |
| `baoan-vector-control/` | 深圳宝安区 DENV / CHIKV 流调辅助、历史对照、自动气象、证据研判与防控任务报告 |
| `global-vector-investigation-planner/` | 全球城市虫媒调查规划与本地化迁移的 Skill、模板和参考资料 |
| `docs/` | 工作流程图，含 PNG、SVG、Mermaid 和可编辑 Excalidraw 文件 |

仅需 **Python 3.11 或更高版本**，运行与测试使用标准库。无需安装 Biomni、Conda 或第三方 Python 包。

## 快速运行

在解压后的根目录执行：

```bash
python3 baoan-vector-control/scripts/vector_control.py --version
python3 baoan-vector-control/scripts/vector_control.py validate \
  --input baoan-vector-control/assets/example_input.json
python3 baoan-vector-control/scripts/vector_control.py analyze \
  --input baoan-vector-control/assets/example_input.json \
  --output demo-offline \
  --weather none
```

打开 `demo-offline/result/report.html`。示例完全虚构，不是实际病例或疫情；输出目录必须为尚不存在的新目录。

将自动天气打开：

```bash
python3 baoan-vector-control/scripts/vector_control.py analyze \
  --input baoan-vector-control/assets/example_input.json \
  --output demo-auto \
  --weather auto \
  --weather-cache weather-cache
```

`auto` 根据输入中的起病日和评估日抓取对应时段。合成示例的日期固定，演示的是这些日期的资料；实际任务应填写经核实的真实日期。程序保存原始天气响应、来源、散列和缺测情况。网络失败仍会明确报告缺口，成功生成报告不代表天气完整。

## 分析新的流调

1. 提取本地材料：`python3 baoan-vector-control/scripts/vector_control.py extract /path/to/report.docx --output extracted.json`。支持 TXT、MD、CSV、JSON、DOCX、XLSX；XLS 需要预先登记且散列一致的本地转换文件。
2. 由使用者或代理对照原文，将事实整理为输入 schema；保留证据引用，核验地点、病种、日期、来源层级和活动时段。提取结果不能直接当作结构化流调输入。
3. 使用 `validate` 校验，再运行 `analyze --input /path/to/input.json --output /path/to/new-result --weather auto`。
4. 审阅 HTML 与 JSON/CSV 台账中的历史对比、证据等级、缺项、防控任务和天气覆盖率。详细字段见 `baoan-vector-control/references/data-contract.md`。

当前规则校验支持的评估日期为 **2026-04-10 至 2027-01-01（含端点）**；这属于本程序规则验证范围，不是官方规范失效日期。范围外任务须先维护并验证适用规则。

## 接回本机授权历史数据

发布包不包含真实病例、流调原件、私人病例统计、六年气象档案或旧分析结果。默认无历史库时明确显示“未接入/未知”，不会把未知写成零例。

```bash
python3 baoan-vector-control/scripts/vector_control.py analyze \
  --input /path/to/input.json \
  --output /path/to/new-result \
  --weather auto \
  --history /path/to/authorized-history/registry.json \
  --previous /path/to/previous/assessment.json \
  --weather-cache /path/to/weather-cache
```

历史库必须满足本程序的 `READY` 注册、文件散列及 schema 契约；普通 Excel 不能直接作为 `--history`。构建和升级方法见 `baoan-vector-control/references/history.md`。默认资源可在本地登记到 `profiles/history-resource.json`；该私有指针不应提交仓库。`--previous` 可多次提供。

需要复用授权气象档案时，在本地 `profiles/weather-resource.json` 设置 `archive_manifest`；本发布副本该值及 `cache_root` 均为空。已有 `READY` 资源只读复用，不自动重建或替换。

## 自动气象与判断边界

- 自动获取固定公开背景坐标的 Open-Meteo ERA5 历史再分析及 IFS 模型天气。ERA5 按起病日、评估日前 84 日取数，形成 7/14/28/56/84 日窗口；IFS 提供独立的近期与可用当前背景。实际时间覆盖以输出质控为准。
- 仅向天气服务发送公开背景坐标、日期、模型与变量；不发送病例姓名、住址、职业或流调轨迹。深圳官方授权站点接口尚未接通，背景格点不代表小区实测。
- 输入后续发、本地扩展、来源未定分开处理；历史比较按病种、街道、小区、职业、年龄及可比气候窗口展开。病例登记数不能自动改称暴发数或人群风险率。
- 暂无完整随访队列与经验证的概率模型，传播概率、置信区间及 1—10 档保持 `null`。P0—P3 是处置优先级，不能解释为传播概率或官方响应等级。
- 全球城市模块用于调查规划，不是已经实现的全球自动评分引擎。全部输出用于调查辅助和专业复核。

## 冻结与离线重放

`analyze` 依次执行联网准备和冻结后离线重放；也可分别使用 `prepare` 与 `run`：

```bash
python3 baoan-vector-control/scripts/vector_control.py prepare \
  --input /path/to/input.json --output /path/to/new-bundle --weather auto
python3 baoan-vector-control/scripts/vector_control.py run \
  --lock /path/to/new-bundle/lock.json --output /path/to/new-output
```

冻结后校验解释器、代码和资源散列，生产重放不联网。锁与路径绑定；迁移程序后应在新环境重新准备，不能直接搬用旧锁。冻结包会包含原始输入与证据快照，应按病例材料管理，勿上传到此仓库。

## 在支持 Skill 的代理中使用

向代理指定对应的 `SKILL.md` 即可使用。也可将两个 Skill 目录复制到宿主的技能目录，例如 `~/.agents/skills/`；如已有同名技能，先检查版本并保留原配置。安装不是 CLI 运行的前提。

## 验证与文件完整性

```bash
python3 -m unittest discover -s baoan-vector-control/tests -v
sha256sum -c MANIFEST.sha256
```

本次发布的实际验收结果记录在 `RELEASE_VALIDATION.md`。运行数据、缓存、输出和本地资源指针应保存在受控本地目录；仅提交程序变更。

父项目的 Apache-2.0 许可原文保留在 `LICENSE`。天气数据的来源、署名及服务使用条件见 `baoan-vector-control/references/weather.md`；程序许可不替代数据服务条款。
