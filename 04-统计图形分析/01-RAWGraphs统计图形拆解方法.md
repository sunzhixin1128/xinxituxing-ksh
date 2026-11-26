# 01 RAWGraphs 统计图形拆解方法

以 RAWGraphs App 的 `48c3b2e` 为源码阅读基准。该版本的应用注册图形、读取数据、管理字段映射与预览和导出。锁文件中的图形模块为 `rawgraphs-charts 1.1.0`，解析和映射核心为 `rawgraphs-core 1.0.0`。本文的案例分析方法是基于这些实现整理的。 [应用图形注册表][registry] · [依赖锁文件][lock]

## 从数据表追到图上标记

| 顺序 | 源码中的观察点 | 分析案例时记录什么 |
| --- | --- | --- |
| 1. 数据导入与类型 | 数据加载环节调用核心解析器，识别并支持重新指定字段类型。 | 数据源、覆盖范围、单位和一行的粒度。每列对应一个字段，每格一个值。编号即使全是数字也可能是类别。注明空值、零、缺测与输入格式。 [加载逻辑][loader] · [解析器][parser] |
| 2. 图形与字段映射 | 图形的维度声明给出字段类型、必填项和多字段条件。预览检查字段缺失、最少数量和类型不匹配。 | 区分分组字段、数值字段、时序、层级、分面、类别色和标签。问清每个标记代表单条记录、汇总组、分箱还是关系。 [预览校验][preview] |
| 3. 分组与统计变换 | 核心支持分组和聚合。部分图形实现自定义 `mapData`，应用可通过 Worker 调用映射。 | 写清「分组键 → 聚合函数 → 输出粒度」。检查重复记录、唯一计数、加总、均值、分位数、分箱及比例分母。默认聚合要按各图形源码核对。 [聚合函数][aggregate] · [Worker][worker] |
| 4. 尺度与绘制 | 先得到映射后的数据，再绘制预览。图形模块的 `render.js` 决定坐标、长度、面积、连线等，也可能继续计算统计量。 | 记录「结果字段 → 标记 → 视觉通道 → 尺度」。区分视觉样式和数量编码。核对基线、范围、面积换算、色标、排序、标注与分面尺度。 [预览渲染][preview] |
| 5. 复核与导出 | 应用支持 SVG、PNG、JPG 及 `.rawgraphs` 项目文件。 | 用几条原始记录手算一组结果，对照图中标记和标签。保存源表、字段映射、聚合设置与项目版本，排版后复核图例、比例和单位。 [导出定义][export] |

复核时要特别关注比例。若每条记录含有分子与分母，组内总体比例通常由「分子之和 ÷ 分母之和」得到。以「逐条比例的简单平均」计算时，回答的是另一种等权问题。两种口径都应写清分母与纳入范围。

## 同样的分类字段可以回答不同问题

假设一行代表一笔订单，A 类有两笔，金额分别为 10、30 元。B 类有一笔，金额为 50 元。把类别映射到 RAWGraphs 条形图的 `Bars` 时，对 `Size` 的三种设置得到不同的图。

| Size 设置 | A 类 | B 类 | 回答的问题 |
| --- | ---: | ---: | --- |
| 留空，按组记录数 | 2 笔 | 1 笔 | 哪类订单更多？ |
| 金额，求和 | 40 元 | 50 元 | 哪类总金额更大？ |
| 金额，均值 | 20 元/笔 | 50 元/笔 | 哪类平均每笔金额更高？ |

图形可保持相似外观，但分组数量领先和总额领先分别属于 A、B 两类。案例报告中应以轴标题、统计口径和可回查的数据说明所比较的量。 [条形图维度][bardim] · [条形图映射][barmap]

## 案例记录模板

| 记录项 | 填写内容 |
| --- | --- |
| 原图与任务 | 图源、页码或链接、版本、模块位置，以及它要回答的一个问题 |
| 数据结构 | 原始观察单位、字段类型、时间与范围、缺失处理。必要时列两至三条样例 |
| 统计变换 | 筛选条件、分组键、聚合与排序规则、比例分母、输出单位 |
| 图形与编码 | 图形名称、一个标记所指的对象、字段到通道的映射、尺度和图例 |
| 证据与复核 | 区分图注直接给出的口径、可复算的数据、根据外观作出的推断及待核验内容 |
| 设计取舍 | 哪种读图任务更容易完成，哪里可能误读，有哪些可验证的改法 |

没有原始表时，先按图例、标签与注释建立可转录的部分。若圆面积、分箱边界或隐含分母缺少证据，把它们列为待核验项。可另做一版明确口径的重制图与原图比较。

[registry]: https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/charts.js
[lock]: https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/package-lock.json
[loader]: https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/hooks/useDataLoader.js
[preview]: https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/ChartPreview/ChartPreview.js
[worker]: https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/worker/worker.js
[export]: https://github.com/rawgraphs/rawgraphs-app/blob/48c3b2ee9ff412721a2c99c7bed38e734b3fa534/src/components/Exporter/Exporter.js
[parser]: https://github.com/rawgraphs/rawgraphs-core/blob/715cc49e2c98f2171aa818f5600817db9a4b3cd6/src/dataset.js
[aggregate]: https://github.com/rawgraphs/rawgraphs-core/blob/715cc49e2c98f2171aa818f5600817db9a4b3cd6/src/expressionRegister.js
[bardim]: https://github.com/rawgraphs/rawgraphs-charts/blob/265a80e0da305c17df06f6a175713b8dbcbbb2bc/src/barchart/dimensions.js
[barmap]: https://github.com/rawgraphs/rawgraphs-charts/blob/265a80e0da305c17df06f6a175713b8dbcbbb2bc/src/barchart/mapping.js
