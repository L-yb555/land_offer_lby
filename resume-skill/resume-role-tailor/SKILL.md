---
name: resume-role-tailor
description: Tailor resumes for a specific role, industry, company, or JD using the Resume workspace. Use when the user wants JD-based resume Markdown/PDF output, role-specific rewriting, unified file naming, or automatic routing between direct generation and interactive refinement without inventing facts.
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

Do not change:
- the existing LaTeX framework, section list, section order, heading names, icon choices, spacing commands, list settings, tabular/minipage structure, header/footer, photo/logo, font sizes, margins, or decorative assets in `template/main.tex`
- school, degree, dates
- company names, internship dates
- papers, awards, contact information
- any metrics or achievements unless they already exist in source files or the user provides them

When updating `template/main.tex`, preserve the current framework exactly. Replace text inside existing fields, headers, `\item` bullets, and section bodies only. Do not add, remove, rename, or reorder sections; do not add or delete list environments; do not tune spacing such as `\vspace`, `\titlespacing`, `\setlist`, line breaks, or minipage widths unless the user explicitly asks for layout/format changes. If the target content does not fit the existing slots, ask the user before changing the structure.

5. Apply the rewrite rules in [`references/rewrite-rules.md`](rewrite-rules.md).

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
Write professional-skill bullets as `JD能力方向 + 具体工具/任务 + 已有实践或可迁移场景`, not as flat keyword lists. First extract what the JD asks for, then write only related skills and practice. Include concrete details such as training/evaluation pipeline, data split, bad case analysis, ROS2 communication, Linux environment setup, paper tracking, experiment notes, VLA task evaluation, deployment/robot-task evaluation, RGB-D/force/tactile fusion, or point-cloud workflow only when supported by source material or explicitly provided by the user.
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

- "帮我投 AI 产品经理" -> start from `baseline_AI PM版.md`
- "做一个更容易过筛的版本" -> start from `baseline_招聘筛选版.md`
- "按这个 JD 重写简历" -> choose the closest baseline, then rewrite summary, selected bullets, and skills around the JD
- "保守一点，不要太花" -> keep the structure stable and only tighten language
- "突出团队协作和推进能力" -> strengthen summary, campus leadership, and product-side bullets

## References

- Source file map: [`references/source-files.md`](source-files.md)
- Rewrite rules: [`references/rewrite-rules.md`](rewrite-rules.md)
- Input template: [`references/input-template.md`](input-template.md)




