# Rewrite Rules

## Goal

Increase match quality for a target role without inventing facts.

## Summary rewriting

- Keep it to 3 to 5 lines in Chinese.
- Start with the strongest role-fit signal.
- Include only 3 or 4 strengths that directly matter for the target role.
- Prefer concrete wording over broad self-praise.

## Toolchain placement

- `专业技能` is a **flat tool list**, not prose. One line per capability area: `**能力方向**：工具、技术栈、方法`, items separated by `、`. List every tool, framework, method, and `方向储备` there, once.
- Write no sentences in `专业技能`. Ban `具备…经验`、`服务过…场景`、`能够独立完成…`, explanatory clauses, and `\hl`. The section declares a stack; the experience bullets below prove it.

  Before:
  > **端侧部署与推理加速**：NVIDIA Jetson AGX Orin 端侧部署；PyTorch → ONNX → TensorRT 模型转换、量化与算子性能优化，具备真机检测/分割实时推理链路打通与端侧算子适配经验。

  After:
  > **端侧部署**：TensorRT、ONNX、NVIDIA Jetson AGX Orin、CUDA、模型量化与算子优化。

- Experience and project bullets do not re-list tools. Write them as `做了什么 → 方法或思路 → 产出或结果`, naming the method in business language (`规则化挖掘算子`、`分层控制链路`、`伪标签蒸馏管道`) rather than the tools that implement it.
- Test: delete the tool name. If the bullet still says what was done and what came out, the name was decoration.
- Keep a name when it is the fact rather than a skill recap: model and version identifiers (`Qwen3-VL-8B`、`YOLO11s-seg`), cluster scale (`4 机 8 卡 H20`), self-built artifacts (`含 vLLM 与 autodistill 的 Docker 镜像`), sensor and hardware models (`Delsys Trigno`、`Sensel`、`Linker Hand G20`), and paper titles.
- A `专业技能` bullet already names the tools; the experience bullet that follows proves them. Saying both is repetition, not emphasis.

## Experience and project bullets

- Reorder or rewrite bullet content by role relevance only inside the existing section framework.
- Put the most target-relevant keyword in the first sentence.
- Each bullet must carry: action + method/思路 + result or output. Do not leave bullets as bare verb phrases with no detail.
- Prefer thorough, detailed bullets over compressed ones. Only shorten when fitting content into a one-page PDF layout.
- When `baseline.md` lacks evidence for a JD requirement, use web-research to write a plausible, original bullet under the relevant existing company or project. Keep company names, project titles, and dates unchanged. Phrase conservatively: `参与`、`协助`、`围绕 X 开展` rather than `主导`、`精通`. Do not invent specific numbers.
- Compress long bullets only as a last resort for layout; otherwise keep the full detail.

## Highlight marking

`\hl{...}` (defined in `template/settings.tex` as bold DarkGoldenrod `RGB 184,134,0`) marks the **flashpoint inside an entry** — the quantified result or shipped outcome that makes that internship or project worth reading. Every internship and project entry gets at least one; a section with no flashpoint marked reads as filler.

Good targets: `\hl{标签验收准确率达 90\%}`、`\hl{在 H20 8GPU 集群完成全参数 SFT}`、`\hl{测试集准确率 0.8984 创历史新高}`、`\hl{历时 12 个月完成端到端项目闭环}`、`\hl{50+ 份《产品建议书》}`、`\hl{国家三等奖}`、`\hl{JCR 一区}`.

Do **not** put `\hl` on paragraph-opening category labels (`\textbf{算法与模型}`-style skill headers) or on institution/affiliation names — those are structural or already stated in 教育背景, and highlighting them buries the real flashpoints. Keep `\textbf` for structural labels and section headings. `\hl` already applies bold, so do not nest one inside the other. Do not use pure yellow `#FFFF00`: at roughly 1.07:1 against white it cannot be read.

## List style

- Every list in the resume uses `\begin{itemize}` (bullet `•`). Never `\begin{enumerate}` — `1. 2. 3.` reads as a checklist, costs horizontal room, and is not how this resume is typeset.
- Applies to every section: 实习经历、科研与项目经历、论文与研究成果、专业技能、荣誉.
- Convert any `enumerate` you find in a template to `itemize`. This is the one sanctioned structural change.

## Format preservation

- When producing PDF through `template/main.tex`, keep the existing LaTeX framework unchanged.
- Do not add, delete, rename, or reorder sections unless the user explicitly asks for format or structure changes.
- Do not change icons, spacing, margins, fonts, header/footer, photo/logo, or minipage/tabular layout.
- All lists are `itemize`, never `enumerate` — see **List style** above.
- Replace content only inside existing fields, headings, section bodies, and `\item` bullets.
- If the JD requires content that cannot fit the current framework, ask before changing the structure.

## Role-specific emphasis

### AI PM / product

- Emphasize AI understanding, user needs, prototyping, communication, cross-functional delivery, business-context translation.
- Surface campus leadership if it supports collaboration and execution.

### Technical / algorithm / research

- Emphasize experiments, pipelines, implementation, engineering environment, model optimization, and papers.

### Screening / ATS

- Shorten sentences.
- Increase keyword density.
- Avoid long narrative paragraphs.

## Things to avoid

- Invented metrics
- Generic filler like "责任心强、学习能力强" repeated without context
- Excessive buzzwords
- Claims that are stronger than the source material
- Prose in `专业技能` — the section is a tool list, not a paragraph

## Preferred wording pattern

Use short, direct phrases such as:

- "积极拥抱 AI，学习适应能力强"
- "曾担任班长，能够快速融入团队"
- "具备较强的协同合作与推进能力"
- "能够在技术与业务之间建立有效连接"
