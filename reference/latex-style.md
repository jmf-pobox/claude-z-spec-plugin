# LaTeX Style Guidelines for Z Specifications

Consistent LaTeX formatting improves readability of Z specifications. This guide covers formatting conventions for Z notation documents.

## Document Structure

### Required Packages

Always include these packages in order:

```latex
\documentclass[a4paper,10pt,fleqn]{article}
\usepackage[margin=1in]{geometry}
\usepackage[colorlinks=true,linkcolor=blue,citecolor=blue,urlcolor=blue]{hyperref}
\usepackage{fuzz}
```

The `hyperref` package must come before `fuzz` to avoid compatibility issues.

### Document Metadata

Include hypersetup for PDF metadata:

```latex
\hypersetup{
  pdftitle={System Name: A Z Specification},
  pdfauthor={Author Name},
  pdfsubject={Z Specification},
  pdfcreator={fuzz/probcli}
}
```

### Table of Contents

Include `\tableofcontents` and `\newpage` after `\maketitle` for navigable documents.

## Line Width Guidelines

### Maximum Line Length

Schema predicates and declarations should not exceed **80 characters** to avoid horizontal overflow in PDF output.

### Breaking Long Lines

#### Schema Declarations

Break after type declarations:

```latex
% BAD - too long
\begin{schema}{State}
receiveLevel, sendLevel, currentStreak, longestStreak : \nat
\end{schema}

% GOOD - split across lines
\begin{schema}{State}
receiveLevel : \nat \\
sendLevel : \nat \\
currentStreak : \nat \\
longestStreak : \nat
\end{schema}
```

#### Schema Predicates

Break at logical operators (`\land`, `\lor`, `\implies`):

```latex
% BAD - single long line
accuracy? \geq 90 \land level < maxLevel \land daysSinceStart \geq minDaysAtLevel

% GOOD - break at logical operators
accuracy? \geq 90 \\
\land level < maxLevel \\
\land daysSinceStart \geq minDaysAtLevel
```

#### Set Comprehensions

Break after `|` and before closing brace:

```latex
% BAD - too long
completedLevels = \{ l : 1 \upto 26 | l < currentLevel \land stats(l).accuracy \geq 90 \}

% GOOD - break for readability
completedLevels = \{ l : 1 \upto 26 | \\
\t1 l < currentLevel \\
\t1 \land stats(l).accuracy \geq 90 \}
```

#### Function Definitions

Break enumerated mappings across lines:

```latex
% BAD - single line
kochOrder = \{ 1 \mapsto K, 2 \mapsto M, 3 \mapsto R, 4 \mapsto S, 5 \mapsto U, ... \}

% GOOD - grouped by 5
kochOrder = \{ \\
\t1 1 \mapsto K, 2 \mapsto M, 3 \mapsto R, 4 \mapsto S, 5 \mapsto U, \\
\t1 6 \mapsto A, 7 \mapsto P, 8 \mapsto T, 9 \mapsto L, 10 \mapsto O, \\
\t1 ... \}
```

### Indentation in Z Schemas

Use `\t1` through `\t9` for indentation within predicates:

```latex
\begin{schema}{ComplexOperation}
\Delta State \\
input? : \nat
\where
input? > 0 \\
\land (condition1 \\
\t1 \implies effect1 \\
\t1 \land effect2) \\
\land condition2 \implies effect3
\end{schema}
```

## Common Patterns

### Conditional Effects

Use `\implies` with aligned predicates:

```latex
accuracy? \geq threshold \implies level' = level + 1 \\
accuracy? < threshold \implies level' = level
```

### Frame Conditions

Group unchanged variables:

```latex
% Explicit preservation
receiveLevel' = receiveLevel \\
sendLevel' = sendLevel \\
currentStreak' = currentStreak
```

Or use schema notation for unchanged state:

```latex
\Xi OtherState  % OtherState unchanged
```

## Formatting Tools

### tex-fmt

If available, use `tex-fmt` to auto-format LaTeX:

```bash
tex-fmt docs/spec.tex
```

This handles:
- Consistent indentation
- Brace alignment
- Blank line normalization

### Manual Verification

After formatting, verify:
1. No horizontal overflow in PDF
2. Schema predicates align logically
3. Line breaks preserve mathematical meaning

## Checklist Before Commit

- [ ] `hyperref` package included
- [ ] `\tableofcontents` present
- [ ] No lines exceed 80 characters in schemas
- [ ] Long predicates broken at logical operators
- [ ] `tex-fmt` run (if available)
- [ ] PDF generated without overflow warnings
