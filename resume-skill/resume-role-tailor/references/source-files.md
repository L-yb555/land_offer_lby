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



