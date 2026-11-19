# `justitex.compile` — Compiler API & CLI Reference

`justitex/compile.py` provides the `JustiTeXCompiler` class and CLI entrypoint for converting plain Markdown legal drafts into court-compliant PDF pleadings (Oregon UTCR 2.010 28-line state format and U.S. District Court LR 10-1 federal format).

---

## 1. Command-Line Interface (CLI)

```bash
python3 -m justitex.compile \
  --input examples/sample_motion.md \
  --output build/Sample_Motion.pdf \
  [--format {auto,state,federal}] \
  [--template PATH_TO_CUSTOM_TEX]
```

### CLI Arguments

| Flag | Default | Description |
|------|---------|-------------|
| `--input`, `-i` | *(required)* | Path to the source Markdown (`.md`) or text pleading file. |
| `--output`, `-o` | *(required)* | Destination path for the compiled PDF (and intermediate `.tex` artifact). |
| `--format`, `-f` | `auto` | Target court geometry: `auto` (inferred from caption header), `state` (Oregon 28-line UTCR 2.010), or `federal` (U.S. District Court LR 10-1). |
| `--template`, `-t` | `None` | Optional path to a custom `.tex` template overriding the built-in template. |

---

## 2. Python API (`JustiTeXCompiler`)

```python
from justitex.compile import JustiTeXCompiler

compiler = JustiTeXCompiler(court_format="auto")
output_pdf = compiler.compile(
    input_md_path="examples/sample_motion.md",
    output_pdf_path="build/Sample_Motion.pdf",
    court_format="auto",
)
```

### Methods

#### `detect_court_format(md_text: str) -> str`
Inspects the first 12 lines of the Markdown document (the court caption header):
- Returns `"state"` if `"IN THE CIRCUIT COURT"` or `"FOR THE COUNTY OF"` appears.
- Returns `"federal"` if `"UNITED STATES DISTRICT COURT"` appears.
- Defaults to `"state"` when unspecified.

#### `extract_document_title(md_text: str, default: str = "LEGAL PLEADING") -> str`
Scans the first 35 lines for pleading title keywords (`COMPLAINT`, `MOTION`, `DECLARATION`, `PETITION`, `MEMORANDUM`, `ANSWER`, `REPLY`, `NOTICE`), strips Markdown formatting, and truncates to 65 characters for the running page footer (`DYNAMIC_FOOTER_TITLE`).

#### `parse_markdown_to_latex(md_text: str, court_format: str) -> str`
Transforms Markdown syntax into LaTeX pleading macros:
- **Top Court Heading (`# IN THE ...`)**: Rendered centered and bold at the top of page 1.
- **Major Section Headings (`# ` / `## `)**: Emitted as `\romanhead{...}` (centered bold, widow-protected).
- **Sub-Headings (`### `)**: Emitted as `\subhead{...}` (left-aligned bold) unless detected as a Claim/Count heading.
- **Claim / Count Headings (`## FIRST CLAIM FOR RELIEF` / `### COUNT I`)**: Emitted via `\claimheading{LABEL}{SUBTITLE}` with centered regular-weight subtitle consumption.
- **Numbered Paragraphs (`1. ...`)**: Emitted with bold paragraph number and 0.5em horizontal spacing.
- **Lettered Sub-Paragraphs (`a. ...`)**: Indented `0.3in` with bold letter prefix.
- **Signature Rules (`_____`)**: Converted to `\rule{3in}{0.5pt}`.
- **Dynamic Execution Dates**: Placeholders like `Executed on Month ____, YYYY` or `Dated: ____` are automatically populated with the compilation date.

#### `compile(input_md_path: str, output_pdf_path: str, template_path: str = None, court_format: str = "auto") -> str`
1. Reads `input_md_path` and resolves `court_format`.
2. Loads `templates/oregon_28line_FROZEN.tex` (`state`) or `templates/federal_district_court.tex` (`federal`).
3. Escapes `doc_title` via `_escape_latex()` and replaces `DYNAMIC_FOOTER_TITLE`.
4. Injects the compiled LaTeX body at `% BODY_PLACEHOLDER` (or before `\end{document}`).
5. Writes the `.tex` file alongside `output_pdf_path` and runs `pdflatex -interaction=nonstopmode` twice to resolve `LastPage` references.
