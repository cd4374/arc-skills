# LaTeX Templates for ARC-Skills

This directory contains LaTeX templates for various AI/ML conferences and journals supported by the ARC research pipeline.

## Available Templates

### Conference Templates

| Venue | File | Description |
|-------|------|-------------|
| **NeurIPS** | `neurips/template.tex` + `neurips_2026.sty` | NeurIPS 2026 (single-column, numbered references) |
| **ICML** | `icml/template.tex` + `icml2026.sty` | ICML 2026 (two-column, 8 pages main + references) |
| **ICLR** | `iclr/template.tex` | ICLR 2026 (OpenReview template) |
| **AAAI** | `aaai/template.tex` | AAAI 2026 (7 pages main + references) |
| **ACL Family** | `acl/template.tex` | ACL, EMNLP, NAACL (requires acl.sty from official release) |
| **CVPR Family** | `cvpr/template.tex` | CVPR, ICCV, ECCV (requires cvpr.sty from official release) |
| **IEEE Conference** | `ieee/template_conference.tex` | IEEE conference proceedings |

### Journal Templates

| Venue | File | Description |
|-------|------|-------------|
| **IEEE Journal** | `ieee/template_journal.tex` | IEEE Transactions journals |
| **Elsevier** | `elsevier/template.tex` | Elsevier journals (Pattern Recognition, etc.) |
| **Springer** | `springer/template.tex` | Springer journals (LNCS, etc.) |

## Using Templates

### Basic Usage

Templates are referenced by `template_id` in the pipeline:

```json
{
  "target_venue": "neurips",
  "template_version": "neurips_2026"
}
```

### Template Selection Priority

1. Explicit `template_version` parameter
2. `target_venue` alias matching
3. Default based on `publication_type`

### Template IDs

```
Conferences:
- neurips_2026, neurips_2025
- icml_2026, icml_2025
- iclr_2026, iclr_2025
- aaai_2026
- acl_2025, emnlp_2025, naacl_2025
- cvpr_2026, iccv_2025, eccv_2026
- kdd_2025
- ieee_conference

Journals:
- ieeetran_journal
- elsevier_cas_sc
- pattern_recognition
- springer_lncs_journal
```

## Math Commands

Include the math commands file for common ML notation:

```latex
\input{math_commands.tex}
```

This provides standardized notation for:
- Vectors/matrices: `\vect{x}`, `\mat{W}`
- Probability: `\E`, `\prob{}`, `\gauss{}{}`
- Optimization: `\argmin`, `\grad{w}`
- ML notation: `\dataset`, `\loss{}`, `\relu`, etc.

See `math_commands.tex` for full documentation.

## Template Structure

Each template includes these sections:

```latex
\section{Introduction}      % Problem + motivation
\section{Related Work}     % Literature review
\section{Method}            % Proposed approach
\section{Experiments}       % Empirical evaluation
\section{Conclusion}        % Summary + future work
\section*{Broader Impact}   % Societal implications (if required)
\section*{Limitations}      % Honest limitations (required)
```

## Compilation Requirements

### Required LaTeX Packages

- `pdflatex` or `xelatex`
- `bibtex` or `biber`
- Standard packages: `amsmath`, `graphicx`, `hyperref`, etc.

### Compilation Command

```bash
pdflatex main.tex
bibtex main
pdflatex main.tex
pdflatex main.tex
```

Or using `latexmk`:

```bash
latexmk -pdf main.tex
```

## Notes on Official Style Files

Some templates require official style files from conference organizers:

- **ACL/EMNLP/NAACL**: Download `acl.sty` from [acl-org/acl-style-files](https://github.com/acl-org/acl-style-files)
- **CVPR/ICCV/ECCV**: Download `cvpr.sty` from [cvpr-org/author-kit](https://github.com/cvpr-org/author-kit)
- **ICML**: Download official style package from [ICML website](https://icml.cc/)
- **NeurIPS**: Style file included in this repo (`neurips_2026.sty`)

Place official style files in the same directory as your `main.tex`.

## Template Status

| Template | Status | Notes |
|----------|--------|-------|
| NeurIPS | Ready | Self-contained style file |
| ICML | Ready | Self-contained style file |
| ICLR | Ready | OpenReview compatible |
| AAAI | Ready | AAAI Press format |
| ACL | Partial | Requires official acl.sty |
| CVPR | Partial | Requires official cvpr.sty |
| IEEE | Ready | Uses standard IEEEtran class |
| Elsevier | Ready | Uses standard elsarticle class |
| Springer | Ready | Uses standard svjour3 class |

## Adding New Templates

To add a new venue template:

1. Create a new directory under `templates/`
2. Add `template.tex` with standard sections
3. Add style file if self-contained (`.sty`)
4. Update `arc-07-00-template-resolve` SKILL.md with new `template_id`
5. Update this README with template information

## References

- [NeurIPS Author Guidelines](https://nips.cc/virtual/2026/)
- [ICML Author Guidelines](https://icml.cc/Conferences/2026/)
- [ICLR Author Guidelines](https://iclr.cc/Conferences/2026/)
- [AAAI Author Guidelines](https://aaai.org/conference/aaai/)
- [ACL Style Files](https://github.com/acl-org/acl-style-files)
- [IEEE Author Guidelines](https://ieeeauthorcenter.ieee.org/)
- [Elsevier Author Guidelines](https://www.elsevier.com/authors)
- [Springer Author Guidelines](https://www.springer.com/gp/authors-editors/authors)
