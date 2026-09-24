---
name: resume-role-tailor
description: Tailor resumes for a specific role, industry, company, or JD using the Resume workspace. Use when the user wants JD-based resume Markdown/PDF output, role-specific rewriting, unified file naming, or automatic routing between direct generation and interactive refinement without inventing facts. Also use whenever the user asks to review, critique, or rewrite a resume for 大厂算法岗 / 视觉算法 / 多模态 / VLM / 自动驾驶数据算法 / 机器人感知 positions (phrases like "评审这份简历", "按大厂标准改简历", "简历太满了"), applying references/algo-review-standards.md.
---

# Resume Role Tailor

## Overview

Generate role-specific resume versions from the source files in `Resume/`.
Select the closest baseline, preserve factual accuracy, and rewrite the summary, project bullets, internship bullets, skills, and ordering to match the target role.

For this workspace, the default deliverables are both a role-specific Markdown resume and a compiled LaTeX PDF based on `template/main.tex`.
Treat `template/main.tex` as a fixed layout contract: generate or rewrite content inside the existing section framework, but do not change the resume format, visual layout, or overall structure unless the user explicitly requests a format/layout change.

Prefer asking the user to fill the template in [`references/input-template.md`](input-template.md) when the request is under-specified and several core fields are missing; otherwise ask one focused question.

## Processing Modes

All three modes use the same source files, baseline-selection rules, JD-to-resume mapping, rewrite rules, factual guardrails, and output conventions. The difference is whether the resume is produced directly or after minimal interaction.

- **自动模式（默认）**。先判断用户是否已提供 `岗位名称 + 公司名 + JD/关键要求`，且这些信息是否足够具体、无明显歧义。若足够具体，直接按现有流程选择 baseline、重写简历并生成输出；若信息缺失、笼统或存在关键歧义，就切到交互式补问，只追问最少的一个关键点。
- **选择一：直接处理（显式覆盖）**。用户明确要求一次性直接生成时使用。不要主动展开多轮确认；只有在缺少必需信息、且无法合理补全时，才提出最小化补充。
- **选择二：交互处理（显式覆盖）**。用户明确要求多轮确认时使用。继续按同一套流程处理，但在 baseline、岗位侧重点、经历取舍、技能强调、输出形式等关键决策点逐步确认，每次只问一个聚焦问题。

## Workflow

1. Read the target input first.

Determine the processing mode in this order:

1. If the user explicitly specifies **选择一** or **选择二**, obey that override.
2. Otherwise use **自动模式**.
3. In **自动模式**, if `岗位名称 + 公司名 + JD/关键要求` are present and specific enough, generate directly.
4. Otherwise switch to the minimum focused question needed and stay interactive until the missing decision is resolved.
5. If the user asks for tradeoffs, baseline comparisons, or multiple options, treat that as an interactive request even when the JD is otherwise complete.

If the user in **选择一** provides incomplete requirements, ask them to fill the template in [`references/input-template.md`](input-template.md). If the user is in **选择二**, collect missing information through one-question-at-a-time interaction instead of forcing a full template fill.

Extract the role title, company type, industry, seniority, JD keywords, and whether the user wants:
- a broad baseline version
- an AI PM / product-oriented version
- a screening / ATS version
- a custom company-specific version

For **选择二**, keep the same initial role/company/JD intake, then pause at the most important unresolved decision and ask one focused question at a time. Do not finalize or write the role-specific outputs until the user has finished the requested interaction or explicitly confirms the current direction.

If the user provides a full JD, treat that JD as the source of truth and do not dilute it with generic role assumptions. If the JD is incomplete and role research is needed, use Google/web search directly for public JD examples and role skill requirements. Do not use `opencli boss search` or BOSS direct search unless the user explicitly asks for it.

2. Read the source files in [`references/source-files.md`](source-files.md).

Use the closest existing baseline instead of starting from scratch unless the user explicitly wants a full rewrite.

3. Choose the starting baseline.

- Use the AI PM baseline for AI PM, product, intelligent interaction, solution, or platform roles.
- Use the screening baseline for concise HR screening, ATS, or keyword-first output.
- Use the general baseline for balanced, fuller, general-purpose output.
- Resolve the exact file paths through [`references/source-files.md`](source-files.md).

3.5. Gap analysis and web-search supplement.

Before rewriting, do a full gap analysis between what the JD requires and what `baseline.md` directly covers:

- List every major JD requirement or keyword.
- For each one, check whether `baseline.md` has a matching skill, bullet, or experience.
- Mark each as: **covered** (direct evidence exists), **transferable** (adjacent evidence, needs conservative phrasing), or **gap** (no direct evidence in baseline).

For every **gap** item, use Google/web search to research:
- Typical professional skills and tool stacks for that JD requirement in this role/industry.
- What interns or junior practitioners at similar companies (same industry, similar company stage/type) commonly work on for similar projects.
- Representative deliverables, workflows, or methodologies for the role and the specific technical area.

Use the search results as reference material only. Do not copy job postings or third-party descriptions verbatim. Instead, use the research to write original, realistic bullet points that:
- Are plausible given the company's known business and the project's known focus.
- Sound like something someone in that role, at that specific company, on that specific project, might genuinely have done.
- Align with the JD requirement being addressed.

**Hard constraints during gap-filling:**
- Keep every company name, internship company, and project title exactly as they appear in `baseline.md`. Do not rename, generalize, or invent new employers or projects.
- Keep all dates, school, degree, awards, and contact information exactly as in `baseline.md`.
- Do not invent specific metrics (numbers, percentages, scale figures) that are not already in `baseline.md` or explicitly provided by the user. Describe work qualitatively or use appropriately hedged language (`参与`、`协助`、`围绕 X 开展`、`基于 Y 推进`) instead.
- After filling a gap, label the source of the evidence in your internal reasoning (covered / transferred / web-researched) so you can check whether each bullet is defensible. Do not expose these labels in the final output.

4. Rewrite only what increases match quality.

Prioritize:
- summary / personal advantages
- project selection and bullet order
- internship bullet order
- skills and keywords, especially detailed professional-skill bullets mapped directly to JD requirements
- title wording
- reusable baseline phrasing when the target JD reveals a broadly useful recruiting emphasis

Before writing, create a compact JD-to-resume mapping:
- What the JD explicitly requires
- Which source experience proves or supports it
- Which requirement is only adjacent/transferable and must be phrased conservatively

Avoid repeating the same evidence in multiple sections. Each major fact should have one primary home:
- `专业技能` explains role capability and toolchain fit
- project/internship bullets prove the capability with concrete practice
- `匹配优势` is optional and should be removed in one-page technical PDFs when it would repeat the skills and experience sections

### `专业技能` is a flat tool list, not prose

`专业技能` is the single home for the toolchain. List every tool, framework, method, and `方向储备` there, once. One line per capability area, formatted as `**能力方向**：工具、技术栈、方法`, with items separated by `、`. Nothing else re-lists them.

Write no sentences in `专业技能`. Ban `具备…经验`、`服务过…场景`、`能够独立完成…`, explanatory clauses, and `\hl`. The section declares a stack; the experience bullets below prove it.

Before:

> **端侧部署与推理加速**：NVIDIA Jetson AGX Orin 端侧部署；PyTorch → ONNX → TensorRT 模型转换、量化与算子性能优化，具备真机检测/分割实时推理链路打通与端侧算子适配经验。

After:

> **端侧部署**：TensorRT、ONNX、NVIDIA Jetson AGX Orin、CUDA、模型量化与算子优化。

Write each experience and project bullet as `做了什么 → 方法或思路 → 产出或结果`. Name the method in business language — `规则化挖掘算子`、`分层控制链路`、`伪标签蒸馏管道`、`两阶段挖掘闭环` — instead of naming the tools that implement it. The bullet's job is to prove the capability; `专业技能` already declared the stack.

The test: delete the tool name and read the bullet. If it still says what was done and what came out, the tool name was decoration — leave it out.

**Keep a tool name when it is the fact, not a skill recap.** Model and version identifiers (`Qwen3-VL-8B`、`YOLO11s-seg`), cluster scale (`4 机 8 卡 H20`), self-built artifacts (`含 vLLM 与 autodistill 的 Docker 镜像`), hardware and sensor models (`Delsys Trigno`、`Sensel`、`Linker Hand G20`), and published paper titles carry information that no other section repeats.

Before:

> 参与 R11 接待机器人感知模块的 YOLO11s-seg 模型训练复现，基于 Python / PyTorch / Ultralytics YOLO 在 Ubuntu 环境下完成实验环境搭建、数据配置、训练参数调整与基础效果验证，推动检测/分割训练链路跑通。

After:

> 复现 R11 接待机器人检测/分割训练链路：完成实验环境搭建、数据配置与训练参数调整，跑通训练并完成基础效果验证，沉淀可复用训练配置。

`Python / PyTorch / Ultralytics YOLO / Ubuntu` all moved out to `专业技能`; `YOLO11s-seg` stays because it identifies which model.

### Every list uses itemize, never numbering

All lists in the resume use `\begin{itemize}` (bullet `•`): 实习经历、科研与项目经历、论文与研究成果、专业技能、荣誉. Do not emit `\begin{enumerate}` — the `1. 2. 3.` numbering reads as a checklist, costs horizontal room, and is not how this resume is typeset. When you touch a template that still uses `enumerate`, convert it to `itemize`.

Do not change:
- the existing LaTeX framework, section list, section order, heading names, icon choices, spacing commands, tabular/minipage structure, header/footer, photo/logo, font sizes, margins, or decorative assets in `template/main.tex`. The one sanctioned exception is the list environment — all lists are `itemize` (see *Every list uses itemize, never numbering*)
- school, degree, dates
- company names, internship dates
- papers, awards, contact information
- any metrics or achievements unless they already exist in source files or the user provides them

Keep the highlight convention: `\hl{...}` (bold DarkGoldenrod, defined in `template/settings.tex`) marks the flashpoint inside each internship and project entry — the quantified result or shipped outcome. Every entry gets at least one. Never put `\hl` on paragraph-opening category labels or institution names; keep those as plain `\textbf` or plain text. Do not use pure yellow `#FFFF00` — it is unreadable on white.

When updating `template/main.tex`, preserve the current framework exactly. Replace text inside existing fields, headers, `\item` bullets, and section bodies only. Do not add, remove, rename, or reorder sections; do not tune spacing such as `\vspace`, `\titlespacing`, `\setlist`, line breaks, or minipage widths unless the user explicitly asks for layout/format changes. The list environment is the one exception: convert any `enumerate` to `itemize`. If the target content does not fit the existing slots, ask the user before changing the structure.

5. Apply the rewrite rules in [`references/rewrite-rules.md`](rewrite-rules.md).

**Algorithm-role review standard.** When the target is a 大厂/一线公司算法岗（视觉、多模态、VLM、自动驾驶数据算法、机器人感知），or the user asks to 评审/诊断/压缩 a resume rather than tailor it to a JD, first read and apply [`references/algo-review-standards.md`](algo-review-standards.md). Where it conflicts with the general density preference below ("prefer thorough, detailed output"), the algorithm standard wins: for one-page algorithm resumes, over-stuffing is the primary failure mode. Its key moves: single memorable persona, 场景→方法→结果 bullets (max 3 per project), every number highlighted, conservative phrasing for anything that invites interview deep-dives (GRPO/后训练/reward 设计), formal publication status + author order, and 20–30% text reduction.

Keep the resume factual, concise, and targeted. Prefer stronger ordering and phrasing over adding unsupported claims.

When a JD exposes reusable technical recruiting language, update `../../baseline.md` conservatively so future role-specific resumes start from a stronger factual base. Only update baseline facts or broadly reusable phrasing that are supported by existing source material or explicit user-provided facts. Do not turn `baseline.md` into a single-company resume.

6. Write the role-specific Markdown output.

Default filename pattern:
- `梁邦一_江南大学_27届硕士_<岗位名称>.md`

Use the exact岗位名称 from the user/JD when it is short and filesystem-safe. If needed, normalize only unsafe filename characters (`\\ / : * ? " < > |`) to `-`; do not translate or embellish the role name.

7. Update the LaTeX resume and compile the PDF when the user asks for PDF output, a complete resume package, or mentions `template/main.pdf`.

Use `../../template/main.tex` as the LaTeX target unless the user provides another path. Preserve the current layout and framework exactly unless the user explicitly asks for visual, format, or structure changes. After editing, run from `../../template`:

```powershell
xelatex -interaction=nonstopmode main.tex
```

Run a second time if LaTeX asks for rerun. Confirm exit code 0 before saying the PDF is generated.

8. Save/copy the compiled PDF using the same base name as the Markdown output.

Default filename pattern:
- `梁邦一_江南大学_27届硕士_<岗位名称>.pdf`

If `template/main.pdf` is the compiled artifact, keep it as the working build output and also create the role-named PDF copy in the requested output directory or, if unspecified, in `Resume/`.

## Input Template

Use this exact collection format when gathering requirements:

- Role title
- Company name
- JD or key requirements
- Output style
- Must-emphasize points
- Must-avoid points
- Output file name

The ready-to-send template lives in [`references/input-template.md`](input-template.md).

## Output Standard

Default to Chinese. Prioritize completeness and specificity over brevity — write thorough, detailed output for each section. Compress only when space is genuinely tight (one-page PDF layout) and the user has not explicitly requested a full version.
Prefer 3 to 5 lines for summary / personal advantages; expand to 6 lines when the role has multiple distinct fit dimensions worth covering.
Keep bullets concrete, specific, and action-oriented. Each bullet should carry at minimum: what was done, with which tools/methods, and what it produced or enabled. Avoid vague bullets like "参与项目开发" with no further detail.
Front-load role-relevant keywords in the first half of the resume.
For technical, algorithm, robotics, VLA, multimodal, or research roles, prefer a `专业技能` section over a separate `匹配优势` section in the one-page PDF when space is tight. Merge fit statements into skill bullets, and bold the strongest keywords or toolchains.
Write `专业技能` as `**能力方向**：工具、技术栈、方法` — a flat, comma-separated list. No prose, no `具备…经验`, no outcome clauses; those belong in the experience and project bullets. This is the only section that names the toolchain — experience and project bullets below it describe work and outcomes without re-listing tools (see `专业技能` is a flat tool list, not prose). First extract what the JD asks for, then list only related tools and methods. Include entries such as training/evaluation pipeline, data split, bad case analysis, ROS2 communication, Linux environment setup, VLA task evaluation, deployment/robot-task evaluation, RGB-D/force/tactile fusion, or point-cloud workflow only when supported by source material or explicitly provided by the user.
When the JD asks for skills not fully proven by source material, separate them as `方向储备`, `流程认知`, or `可迁移基础`; do not write `精通`, `掌握`, or `实际应用` unless the source files or user-provided JD context support it.
Produce Markdown first. If PDF output is requested, use the Markdown as the content source and place that content into the existing `../../template/main.tex` LaTeX section slots before compiling. Markdown and PDF must use the same role-tailored facts and naming base, but the PDF must follow the current `main.tex` section framework and ordering.
If the role is product-oriented, emphasize:
- AI understanding plus product expression
- user needs, cross-functional collaboration, prototype, communication, delivery
If the role is technical or research-oriented, emphasize:
- data pipeline, model experiments, engineering, environment setup, implementation, papers
- detailed professional skills with bolded role keywords
- VLA / multimodal / robotics evidence when present: real-robot practice, simulation practice, ROS2, robot-task evaluation, fine-tune readiness, training and evaluation framework maintenance
If the role is screening-oriented, shorten wording and increase keyword density.

## Guardrails

**Factual anchors — never change these:**
- Company names, internship companies, project titles, dates, school, degree, awards, and contact information must match `baseline.md` exactly.
- Do not invent specific metrics (numbers, headcounts, percentages, dataset sizes) that are not already present in `baseline.md` or explicitly provided by the user.

**Gap-filling — allowed with constraints:**
- When `baseline.md` has no direct evidence for a JD requirement, use Google/web search to research what practitioners in that role, at that company type, on that project type, commonly do. Write original bullet points based on that research that are plausible within the known company and project context.
- Phrase web-researched bullets conservatively: use `参与`、`协助`、`围绛 X 开展`、`基于 Y 推进`、`负责配合` rather than `主导`、`精通`、`独立完成` unless the source material supports the stronger claim.
- Separate clearly unproven skills as `方向储备`、`流程认知`、or `可迁移基础` in the `专业技能` section; do not write `精通`、`掌握`、or `实际应用` unless `baseline.md` supports it.
- Do not copy third-party JD text or competitor descriptions verbatim into the resume.

**Quality and density:**
- Do not overuse generic adjectives.
- Produce thorough, complete output for each section — prefer detailed, action-oriented bullets over compressed or vague summaries. Err on the side of including more specific detail rather than less, especially for internship and project bullets; the user can trim.
- Do not stuff the same JD keyword into multiple sections. One precise mention in `专业技能` plus one proof point in the most relevant experience is the ceiling.
- Do not blindly include JD technology keywords (Java, QT, MySQL, Oracle, OpenGL, point-cloud libraries, RealSense, deployment claims) unless supported. If useful but only adjacent, phrase as learning focus or transferable foundation.
- When two source files conflict, prefer the most recent file in `Resume/` or ask the user only if the conflict materially affects accuracy.


## Markdown to PDF Template

Treat the generated Markdown file as the content source of truth for the final resume package. Do not directly render the Markdown to PDF.

To create the PDF, map the Markdown sections and bullets into the existing LaTeX template:

- Template source: `../../template/main.tex`
- Build output: `../../template/main.pdf`
- Compiler command: `xelatex -interaction=nonstopmode main.tex` from `../../template`

Preserve the current LaTeX layout, header/footer, photo, logo, spacing, section style, section order, environments, and overall framework unless the user explicitly asks for visual/format/structure changes. The Markdown and LaTeX/PDF must contain the same role-tailored facts and emphasis, but never force Markdown structure onto `main.tex` by changing the template framework.

## Filename Rules

Default base name: `梁邦一_江南大学_27届硕士_<岗位名称>`.

Apply the same base name to every generated deliverable:
- Markdown: `<base>.md`
- PDF: `<base>.pdf`
- Keep `template/main.tex` and `template/main.pdf` as editable/build artifacts unless the user explicitly asks to rename or move them.

Examples:
- 岗位名称 `投资分析(技术方向)` -> `梁邦一_江南大学_27届硕士_投资分析(技术方向).md` and `.pdf`
- 岗位名称 `机器人感知算法实习生` -> `梁邦一_江南大学_27届硕士_机器人感知算法实习生.md` and `.pdf`

## Quick Mapping

- "帮我投国企 / 央企 / 研究所" -> start from `template/main_guoqi.tex`（政治面貌、校园组织经历前置）
- "帮我投 AI 产品经理" -> start from `baseline_AI PM版.md`
- "做一个更容易过筛的版本" -> start from `baseline_招聘筛选版.md`
- "按这个 JD 重写简历" -> choose the closest baseline, then rewrite summary, selected bullets, and skills around the JD
- "保守一点，不要太花" -> keep the structure stable and only tighten language
- "突出团队协作和推进能力" -> strengthen summary, campus leadership, and product-side bullets

## References

- Source file map: [`references/source-files.md`](source-files.md)
- Rewrite rules: [`references/rewrite-rules.md`](rewrite-rules.md)
- Input template: [`references/input-template.md`](input-template.md)
- 大厂算法岗评审与重写标准: [`references/algo-review-standards.md`](algo-review-standards.md)




