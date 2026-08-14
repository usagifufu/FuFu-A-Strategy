# HTML 报告版式约定（WorkBuddy 适配版，替代原 pptd-layout.md）

原版用 kimi-slides 渲染 `.pptd`。本版改为 `scripts/build_html_report.py` 读取 `content.json`
装配**单文件自包含 HTML**（图表以 base64 内嵌，离线可看）。本文件定义主题配色、
分节结构与 `content.json` 字段约定。

## 主题（MORANDI 暖棕，与原 pptd 主题一致）

```json
{
  "theme": {
    "primary":   "#7A5C3E",
    "primaryDark":"#54401F",
    "tint1":     "#C4B7A6",
    "tint2":     "#E4DCCC",
    "tint3":     "#F5F1E8",
    "ink":       "#2B2620",
    "gray":      "#8A8378",
    "grayLight": "#B9B3A8",
    "accent":    "#A05238",
    "positive":  "#5B7A5E",
    "negative":  "#A8503C",
    "line":      "#D8D2C6"
  }
}
```

红涨绿跌（A股惯例）：涨/利多用 `positive`（绿），跌/利空用 `negative`（红）。

## content.json 顶层结构

```json
{
  "title": "报告全名（人类可读，作为 HTML <title> 与封面标题）",
  "theme": { ... 上面的主题对象 ... },
  "meta":  { "ticker": "002463", "name": "沪电股份", "updated": "2026-08-10",
             "rating": "增持", "weight": "8-15%" },
  "pages": [ ... 下面各节对象 ... ]
}
```

## pages 分节类型与字段

按报告结构约 18 节。每节通用字段：`type`、`kicker`（小标签）、`title`、`footer`
（来源，如 `"来源: a-stock-data/tdx — 数据集, 截至2026-08-10；公司公告"`）、`accent`（bool，
风险警示类评级封面置 true 用 accent 色条）。

| type | 用途 | 关键字段 |
|---|---|---|
| `cover` | 封面 | `thesis`（一句话结论）、`cards`（4个KPI卡：`[{label,value}]`）、`date`、`rating`、`weight`、`accent` |
| `summary` | 摘要（评级/加权期望/三情景速览） | `rows`（表格行）、`verdict`（裁定条文本） |
| `section` | 六维各页/估值/情景 的标题+正文 | `body`（段落或要点数组）、`cards`（可选） |
| `kpi` | KPI 四卡（bignum+说明） | `kpis`：`[{label, value, sub}]` |
| `chart` | 图表页 | `img`（PNG 路径，脚本 base64 内嵌）、`caption`、`side`（右侧文本数组） |
| `table` | 表格页（F红线/操作手册/风险日历/同行对比） | `columns`（表头）、`rows`（二维数组）、`note`（可选） |
| `verdict` | 裁定页（F红线/操作手册/风险日历三联） | `blocks`：`[{title, type, table:{columns,rows}}]` |
| `text` | 纯文本/要点页 | `body` |

视觉化规范（与原版一致）：
- 小图标字符：◆（要点）⚠（红旗）✕（否定/纪律）✓（验证通过）①②③（步骤）。
- 图表：先 `scripts/morandi_charts.py` 生成 PNG（figsize 8.4×5.33 dpi170），HTML 内嵌 base64。
- 每页 footer 标注来源：`来源: a-stock-data/tdx — 数据集, 截至YYYY-MM-DD；公司公告`。
- 交付：HTML_REF 指向 .html，严禁再转 pptx（除非用户另行要求）。

## 质检流程（render 后必做）

```bash
python3 scripts/build_html_report.py content.json -o report.html
# 在浏览器/预览中逐节目检：表格溢出、文本截断、注释压标题、footer 渲染
# 同时脱敏检查：不得出现其他标的的评级/期望（同行对比纯倍数表允许）
```

## 与原 pptd 版式陷阱的对应处理

原 kimi-slides 的 YAML 陷阱（flow mapping 冒号、`<` 截断、`&` 截断）在 HTML 中天然不存在
（HTML 由脚本转义渲染）。本版唯一需注意：图表 PNG 尺寸过大时内嵌体积膨胀——
单图控制在 ≤300KB，7 图合计 ≤2MB 为宜。
