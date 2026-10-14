# 分发包验收

执行日期：2026-10-14。全部输入使用随包合成示例或合成测试夹具，未读取真实病例库。

| 检查 | 实际结果 |
| --- | --- |
| 标准库单元测试 | 168 项通过，0 失败 |
| 无历史库、无天气的 CLI 分析 | 成功生成 HTML、JSON、CSV；历史状态 `not_connected`，明确未知 |
| 无预置气象档案的自动联网分析 | Open-Meteo 请求成功，ERA5 状态 `complete_reanalysis` |
| ERA5 历史窗口 | 示例 5 个日期锚点、2 个公开背景点的 14 日及 84 日窗口完整 |
| IFS 前 14 日 | 两个背景点的温度、湿度、降雨、风速和风向均覆盖 336/336 小时 |
| 当前天气 | 示例评估日期为过去时点，当前记录缺失/过期；程序按未知保留，未用今日值冒充历史当前值 |
| 官方气象接口 | `not_connected`，没有声称接通官方观测 |
| 冻结重放 | 禁用 socket 网络访问后运行成功；全部结果文件与首次运行逐字节一致 |
| 无训练模型的分数 | 所有行的概率及 1—10 档均为 `null` |

自动联网测试的评估时点为合成示例的 `2026-10-03T12:00:00+08:00`，起病锚点为 2026-09-28、09-29、09-30、10-01。该测试验证日期选择和自动抓取，不表示实际疫情，也不保证未来服务可用性。天气原始响应、临时输出、冻结锁和本地绝对路径均未纳入发布包。

## 重现基本验证

```bash
python3 -m unittest discover -s baoan-vector-control/tests -v
python3 baoan-vector-control/scripts/vector_control.py analyze \
  --input baoan-vector-control/assets/example_input.json \
  --output demo-offline --weather none
python3 baoan-vector-control/scripts/vector_control.py analyze \
  --input baoan-vector-control/assets/example_input.json \
  --output demo-auto --weather auto --weather-cache weather-cache
```

发布清单仅收录源码、规则、schema、合成示例、测试、无私人资源的配置、说明和流程图。原算法及测试代码保持不变；部署说明及本机配置已在分发副本中调整。
