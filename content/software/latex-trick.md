---
title: Latex Tricks
date: 2025-05-04
tags:
  - latex
  - geek
  - tools
categories: handbook
---

# Latex tricks

## note

## comment

use following to comment mulilines:

```tex
\iffalse
comments
another line
\fi
```

```tex

\newcommand{\chen}[1]{{\textcolor{red}{[Chen: #1]}}}
\newcommand{\liu}[1]{{\textcolor{magenta}{[Liu: #1]}}}
\newcommand{\wang}[1]{{\textcolor{blue}{[Wang: #1]}}}

\begin{document}
\liu{test}

\chen{test}

\wang{test}
\end{document}
```

## diff

`latexdiff` to diff in pdf. tldr:

```
  Determine differences between two LaTeX files.
  More information: <https://ctan.org/pkg/latexdiff>.

  Determine changes between different versions of a LaTeX file (the resulting LaTeX file can be compiled to show differences underlined):

      latexdiff old.tex new.tex > diff.tex

  Determine changes between different versions of a LaTeX file by highlighting differences in boldface:

      latexdiff --type=BOLD old.tex new.tex > diff.tex

  Determine changes between different versions of a LaTeX file, and display minor changes in equations with both added and deleted graphics:

      latexdiff --math-markup=fine --graphics-markup=both old.tex new.tex > diff.tex

```

`git-latexdiff` compare pdf in git history

Important options:

```
Usage: git latexdiff [options] OLD [NEW]
OLD and NEW are Git revision identifiers. NEW defaults to HEAD.
If "--" is used for NEW, then diff against the working directory.
    --latex               run latex instead of pdflatex
    --xelatex             run xelatex instead of pdflatex
    --lualatex            run lualatex instead of pdflatex
    --main <file>         name of the main LaTeX, R Sweave,
    --tmpdirprefix        where temporary directory will be created (default: /tmp).
                            Relative path will use repository root as a base
    --ignore-latex-errors keep on going even if latex gives errors, so long as
                          a PDF file is produced
```

For example, use `git latexdiff HEAD -- --tmpdirprefix tmp --main main.tex` to check differences between worktree and last commit.

## picture

If fix position:

```tex
\usepackage{float} % 在导言区添加
\usepackage{graphicx} % 用于插入图片

...

\begin{figure}[H] % 使用 [H] 固定位置
    \centering
    \includegraphics[width=0.8\textwidth]{example-image}
    \caption{固定位置的图片}
    \label{fig:example}
\end{figure}
```

centering:

```tex
\usepackage{graphicx} % 在导言区添加

...

{
    \centering
    \includegraphics[width=0.8\textwidth]{example-image}
    \par % 确保居中生效
}
```
