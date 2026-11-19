# Contributing to JustiTeX

Thank you for contributing to **JustiTeX**! Our mission is to bridge the Access to Justice (A2J) gap by giving pro se litigants, legal aid clinics, and public defenders deterministic, courtroom-grade pleading typesetting (Oregon UTCR 2.010 28-line grid and Federal District Court LR 10-1) under the **GNU AGPLv3** license.

---

## 🧭 Core Architectural Invariants

1. **Frozen 28-Line Grid Geometry (`templates/oregon_28line_FROZEN.tex` & `templates/pleading.sty`)**
   - Do **not** alter line pitch (`\textheight/28` / `23.1425bp`), baseline offset (`+3.5pt`), double left margin rules (`1.3in` and `1.265in`), or page margins (`1.5in` left, `1in` top/bottom/right) without explicit regression verification against the golden reference PDF.
   - Widow and orphan penalties must remain locked at `10000` (`\widowpenalty=10000`, `\clubpenalty=10000`, `\brokenpenalty=10000`).

2. **Local-First & Zero Exfiltration**
   - JustiTeX compiles pleadings locally (via `pdflatex` / `xelatex` in CLI mode, or client-side in the web studio preview). Never add telemetry or third-party scripts that transmit draft pleading contents off the user's machine.

3. **Accessibility First (WCAG 2.1 AA/AAA)**
   - Pro se litigants frequently rely on assistive technology, screen readers, keyboard navigation, and mobile devices.
   - All web studio form inputs must have programmatic `<label for="...">` associations, keyboard-accessible `<button type="button">` controls, logical heading hierarchy (`h1` → `h2` → `h3`), and high-contrast focus rings.

---

## 🛠️ Development Workflow

### 1. Environment Setup

```bash
git clone https://github.com/Executive-Function-OS/JustiTeX.git
cd JustiTeX
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Ensure a TeX Live distribution is installed if testing full PDF compilation:

```bash
# Ubuntu / Debian
sudo apt-get install texlive-latex-base texlive-latex-extra texlive-fonts-recommended
```

### 2. Running the Test Suite

Run the unit and regression tests before submitting a pull request:

```bash
python3 -m unittest discover -s tests
```

All tests in `tests/test_compile.py` verify caption detection, claim-heading formatting, LaTeX special-character escaping in running footers, and line-7 first-page body offset.

### 3. Adding a New Jurisdiction Template

To add support for a new state or federal local rule:
1. Place the `.tex` template in `templates/<jurisdiction>_pleading.tex` using `DYNAMIC_FOOTER_TITLE` as the running footer placeholder and `% BODY_PLACEHOLDER` (or `\end{document}`) as the injection target.
2. Add jurisdiction detection or an explicit `--format` option in `justitex/compile.py` (documented in [`docs/compile.md`](docs/compile.md)).
3. Add a sample Markdown fixture in `examples/` and unit tests in `tests/test_compile.py`.

---

## 📋 Pull Request Checklist

- [ ] `python3 -m unittest discover -s tests` passes with zero failures.
- [ ] Frozen 28-line Oregon UTCR 2.010 geometry is preserved.
- [ ] All LaTeX special characters (`&`, `%`, `$`, `#`, `_`) are escaped via `_escape_latex()`.
- [ ] Web studio edits (`index.html`) maintain zero `axe-core` critical or serious accessibility violations.
- [ ] Documentation (`README.md` or `docs/compile.md`) is updated for any CLI or macro changes.

---

## ⚖️ License

By contributing to JustiTeX, you agree that your contributions will be licensed under the [GNU Affero General Public License v3.0 (AGPLv3)](LICENSE).
