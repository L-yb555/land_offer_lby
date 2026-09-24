# Source Files

Use these files as the authoritative starting materials for resume generation in this workspace.

## Core baselines

- `../../baseline.md`
  Best for fuller, balanced resumes. Use when the user wants a general version or a base draft.


## LaTeX template

- `../../template/main.tex`
  Current editable LaTeX resume source and the default PDF template. After generating the role-specific Markdown, transfer its selected content into this template when the user asks for PDF output, template updates, or `main.pdf`.

- `../../template/main.pdf`
  Build artifact generated from `main.tex`. Compile with `xelatex -interaction=nonstopmode main.tex` from `../../template`, then copy/rename to the role-specific filename when requested.

### Role variants

All variants share the same framework (no footer band; contact = 邮箱 + 电话 in the 个人信息 tabular; entry titles `\small`; blue tagline under each entry; `\hl` flashpoints; all lists `itemize` — never `enumerate`; `专业技能` as a flat tool list; unified three-paper 成果 section). Pick by target employer:

- `main.tex` — 自动驾驶感知工程师（智驾主线）
- `main_perception.tex` — 自动驾驶感知工程师（Caterpillar R0000390959 定向版：前置 Orin/TensorRT 加分项，LiDAR/BEV/跟踪列入方向储备）
- `main_dataengine.tex` — 自动驾驶数据引擎工程师（Caterpillar R0000390959 定向版：前置自动标注工具链 / 数据质量体系 / 训练基础设施）
- `main_algo.tex` — 算法研发（视觉方向）
- `main_pm.tex` — AI 产品经理 / 产品导向
- `main_ant.tex` — 具身智能 / 机器人算法
- `main_scconsult.tex` — 工业 AI / 咨询导向
- `main_general.tex` — 通用平衡版
- `main_guoqi.tex` — 国企 / 央企 / 院所导向：政治面貌与校园组织经历前置，含制造业一线实习，信息表保留年龄与籍贯

Compile any variant twice (`xelatex` two passes) — the header/watermark tikz overlays use `remember picture` and misplace on a single pass.

## Images

- `../../figure_normal.jpg`
  Default profile image for markdown/doc exports.

- `../../figure_formal.jpg`
  More formal alternate profile image if the user wants a more conservative tone.

## Legacy files

- `../../江南大学_27届硕士实习_AI产品.pdf`
  Historical visual reference. Use only as content/layout inspiration if needed.

- `../../main.tex`
  Legacy LaTeX artifact outside the current template folder. Do not use for new PDF generation unless the user explicitly asks.

## Practical selection rules

- Prefer the baseline that already matches the target role.
- Only merge content across baselines when it improves fit and does not introduce contradictions.
- If the user asks for "update all resumes", apply the same factual update across all three markdown baselines.



