# 工作空间 Skill 汇总

> 由 WorkBuddy 自动生成 · 2026-07-22
> 来源工作空间：`D:\素材\wodkbuddy\1\存储`
> 归档仓库：`3028074425q-afk/-rog`

本仓库用于归档当前工作空间内存储的 WorkBuddy skill 及其说明。当前共收录 **1** 个 skill。

## 目录

1. [cn-script-breakdown（中文剧本拆解）](#1-cn-script-breakdown中文剧本拆解)

---

## 1. cn-script-breakdown（中文剧本拆解）

### 1.1 是什么

专用于**中文剧本拆解（Script Breakdown / 制作细分）**的 skill。它将中文剧本（自由文本 / 分镜体 / Fountain / FDX 格式）转换为结构化的影视制作资产清单——**人物、场景、道具**三类，并标注同场关系；对高重要度资产进一步生成**属性细分卡**（角色圣经 / 场景概念 / 道具档案）。输出可直接交付制片、美术、服化道部门使用。

可选能力：对高重要度实体调用宿主图像生成能力，产出**概念参考图**（角色概念图 / 场景概念图 / 道具设计图）。

### 1.2 关键特性

- **中文优先**：面向中文小说体 / 分镜体自由文本，走 LLM 结构化抽取；Fountain / FDX 仅作为可选增强底座。
- **零 Key 抽取**：Phase 0–5 的抽取完全在 WorkBuddy 本地上下文完成，基于 `references/extraction-schema.md` 定义的 JSON Schema，**不依赖任何外部 API Key**，结果结构可控、可审计。
- **按需出图**：仅 Phase 6（参考图生成）调用图像模型，按张计费，执行前须告知用户预估张数与 credits 消耗。
- **可校对回路**：所有抽取结果默认可编辑、可回写；报告中显式标注「需人工校对」项（如自动推断的关系）。

### 1.3 适用场景

- 用户提供中文剧本并要求「拆解 / 整理 / 分析人物场景道具 / 做细分表」。
- 用户提供剧本文件（`.txt` `.md` `.fountain` `.fdx`）并要求提取资产。
- 用户要求对角色做「角色圣经 / 人物小传」、对场景做概念设定、对道具做资产档案。
- 用户要求为重要人物 / 场景 / 道具生成概念参考图 / 角色立绘 / 场景概念图 / 道具设计图。

### 1.4 工作流（Phase 0 – 6）

| 阶段 | 名称 | 产出 |
| --- | --- | --- |
| Phase 0 | 接收与识别输入 | 判定格式（Fountain / FDX 或中文自由文本） |
| Phase 1 | 分段与预处理 | 按「幕 / 集 / 场」切分、清洗；长本分块（单块 ≤ 约 6000 中文字，以「场」为最小单位） |
| Phase 2 | 实体抽取（核心） | 按 Schema 输出 `characters` / `scenes` / `props` 及同场关系 |
| Phase 3 | 消歧与合并 | 同名合并、别名归一、场景命名统一、回填同场关系 |
| Phase 4 | 重要资产细分 | 对 `high` 实体生成角色圣经 / 场景概念 / 道具档案，并产出 `visual_prompt` |
| Phase 5 | 输出与导出 | 写 `breakdown.json`，调用脚本生成报告与 CSV |
| Phase 6 | 参考图生成（可选） | 生成概念图并回填 `subdivision.concept_image` 路径 |

### 1.5 数据模型（核心字段）

**人物 Character**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `c{n}`，如 `c1` |
| `name` | string | 标准姓名 / 称谓 |
| `aliases` | string[] | 别名、绰号、尊称 |
| `role_type` | enum | 主角 / 配角 / 群演 / 旁白 / 未知 |
| `gender` | enum | 男 / 女 / 未知 |
| `age_range` | string | 年龄区间，如「20-30」 |
| `importance` | enum | high / medium / low（按出场频次 + 情节权重 + 跨场景跨度） |
| `scenes` | string[] | 出场场景 `id`（升序） |
| `subdivision` | object | high 时必填：appearance / personality / goal / relationships[] / wardrobe[] / first_scene / visual_prompt / concept_image |

**场景 Scene**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `s{n}` |
| `scene_no` | string | 场次编号，如「第一场」「S1」 |
| `title` | string | 场景简述 |
| `int_ext` | enum | 内景 / 外景 / 内外结合 / 未知 |
| `time_of_day` | enum | 日 / 夜 / 晨 / 黄昏 / 未知 |
| `location` | string | 地点名 |
| `characters` | string[] | 出场人物 `id` |
| `props` | string[] | 出现道具 `id` |
| `importance` | enum | high / medium / low |
| `subdivision` | object | high 时必填：mood / visual_tone / function / set_dressing[] / visual_prompt / concept_image |

**道具 Prop**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | `p{n}` |
| `name` | string | 道具名 |
| `category` | enum | 武器 / 服饰 / 家具 / 电子设备 / 交通工具 / 食物 / 文书 / 其他 |
| `importance` | enum | high / medium / low |
| `reusable` | boolean | 是否可跨场复用的关键资产 |
| `scenes` | string[] | 出现场景 `id` |
| `characters` | string[] | 主要持有 / 使用者 `id` |
| `subdivision` | object | high 时必填：visual_desc / function / reference_note / visual_prompt / concept_image |

### 1.6 输出与导出

- `breakdown.json`：完整结构化结果（顶层含 `meta` / `characters` / `scenes` / `props`）。
- `breakdown_report.md`：概览 + 三类资产表格 + 重要资产细分卡 + 概念图一览。
- `characters.csv` / `scenes.csv` / `props.csv`：可入库表格（UTF-8-SIG）。
- 导出脚本：`scripts/export_breakdown.py`（纯标准库，可隔离环境运行）

  ```bash
  python export_breakdown.py --input breakdown.json [--outdir ./out]
  ```

### 1.7 skill 目录结构（本体）

```
cn-script-breakdown/
├── SKILL.md                      # 技能定义与工作流（权威文档）
├── references/
│   └── extraction-schema.md      # 完整 JSON Schema、字段枚举、重要性评分细则、视觉提示词模板
└── scripts/
    └── export_breakdown.py       # JSON → Markdown 报告 + CSV 导出
```

### 1.8 同工作空间配套

- `剧本资产整理工具_调研与方案.md`：开源竞品调研（StoryWorld / Moyin Creator / BigBanana AI Director / screenplay-tools / story-shot-agent / CineView-AI）与自建实现方案。
- `script-breakdown-app/`：基于 FastAPI + 前端的剧本拆解 Web 应用原型（`backend/` 抽取与图像生成模块、`frontend/`、`.env.example`、`requirements.txt`），可作为该 skill 的产品化落地参考。
- 示例产物：`sample_script.txt`、`breakdown.json`、`breakdown_report.md`、`characters.csv`、`scenes.csv`、`props.csv`、`assets/concept/*.png`（概念参考图）。

### 1.9 来源与许可

- 由 WorkBuddy 在本工作空间内生成（`agent_created: true`）。
- 设计参照开源调研结论，详见 `剧本资产整理工具_调研与方案.md`：开源中无现成可免费商用的「中文剧本 → 人物/场景/道具 + 重要资产细分」底座，故采用自建方案；StoryWorld 提供最透明的架构范式，BigBanana 提供最贴合的功能范式，screenplay-tools 提供结构化输入底座。

---

*本汇总由 WorkBuddy 依据工作空间内 `cn-script-breakdown.zip` 及其配套文件自动整理。如需将完整 skill 源码（SKILL.md / references / scripts）一并推送到本仓库，可另行告知。*