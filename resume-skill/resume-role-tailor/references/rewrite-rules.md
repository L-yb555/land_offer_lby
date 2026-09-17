# Rewrite Rules

## Goal

Increase match quality for a target role without inventing facts.

## Summary rewriting

- Keep it to 3 to 5 lines in Chinese.
- Start with the strongest role-fit signal.
- Include only 3 or 4 strengths that directly matter for the target role.
- Prefer concrete wording over broad self-praise.

## Experience and project bullets

- Reorder or rewrite bullet content by role relevance only inside the existing section framework.
- Put the most target-relevant keyword in the first sentence.
- Each bullet must carry: action + tool/method + result or output. Do not leave bullets as bare verb phrases with no detail.
- Prefer thorough, detailed bullets over compressed ones. Only shorten when fitting content into a one-page PDF layout.
- When `baseline.md` lacks evidence for a JD requirement, use web-research to write a plausible, original bullet under the relevant existing company or project. Keep company names, project titles, and dates unchanged. Phrase conservatively: `参与`、`协助`、`围绕 X 开展` rather than `主导`、`精通`. Do not invent specific numbers.
- Compress long bullets only as a last resort for layout; otherwise keep the full detail.

## Format preservation

- When producing PDF through `template/main.tex`, keep the existing LaTeX framework unchanged.
- Do not add, delete, rename, or reorder sections unless the user explicitly asks for format or structure changes.
- Do not change icons, spacing, margins, fonts, header/footer, photo/logo, minipage/tabular layout, list settings, or LaTeX commands that control appearance.
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

## Preferred wording pattern

Use short, direct phrases such as:

- "积极拥抱 AI，学习适应能力强"
- "曾担任班长，能够快速融入团队"
- "具备较强的协同合作与推进能力"
- "能够在技术与业务之间建立有效连接"
