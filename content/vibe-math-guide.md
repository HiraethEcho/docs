# Vibe Math Guide

This guide records practical workflows, prompts, and techniques for using an AI agent to support active mathematical research. It is intended as a reproducible reference for continuing work with agents on problems in birational geometry and foliations.

Contents

- Principles and framing
- Useful prompt patterns
- Typical agent workflow for a theorem-level task
- How to manage assumptions and conditional reasoning
- Verification and citation hygiene
- Working with computations, examples, and counterexamples
- Recommended file and versioning practices

---

## Principles and framing

- Be precise about the mathematical object: always give the triple or tuple you work with, e.g. $(X,\mathcal{F},\Delta)$ and state whether $\mathcal{F}$ is algebraically integrable.
- State which standard conjectures/results you accept as hypotheses (e.g. finite generation, abundance, existence of dlt modifications). Use agent to mark which steps rely on which hypothesis.
- Keep logical claims explicit: use "Assume ..." and "Then ..." rather than eliding dependence.

---

## Useful prompt patterns

- "Find papers about <concept>" — useful keywords: "adjoint foliated structure", "foliated MMP", "algebraically integrable foliation", "foliated dlt modification".
- "Extract theorem statements and exact hypotheses" — ask agent to return LaTeX-ready theorem statements and exact citation lines.
- "Compare X and Y" — ask agent to produce a point-by-point comparison: objects, hypotheses, central lemmas used, where each uses finiteness or abundance.

---

## Typical agent workflow for a theorem-level task

1. Bibliographic scan: ask agent to search Zotero / arXiv for keywords; gather candidate references.
2. Extract exact statements: for each candidate, extract theorem numbers, exact hypotheses, and precise conclusions in LaTeX form.
3. Map dependencies: produce a dependency graph listing which lemmas rely on which external results or conjectures.
4. Produce a plan: write a step-by-step proof sketch that explicitly cites which external results are used at every step.
5. Implement conditional branches: produce two parallel proof sketches — one accepting certain hypotheses, one conditional if those are not accepted.
6. Verify critical steps: ask agent to fetch specific lemmas used as black boxes and check whether their hypotheses are satisfied in your situation.

---

## How to manage assumptions and conditional reasoning

- Maintain a short assumptions list at top of your working file; label each assumption with a tag (A1, A2, ...).
- When a step uses an assumption, mark it inline: "(uses A1)". This makes it straightforward to produce an unconditional vs conditional theorem.
- If a needed assumption is nonstandard, ask the agent for references, counterexamples, or partial results that might replace it.

---

## Verification and citation hygiene

- Always request exact theorem statements and page/section numbers from the agent when relying on a result.
- Preserve provenance: when the agent paraphrases a claim, ask for the literal quote from the paper or the theorem number.
- Flag anything that is only in a preprint or unpublished manuscript and treat it as conditional if necessary.

---

## Working with computations, examples, and counterexamples

- Use small-dimensional examples (surfaces, threefolds) to test constructions.
- Ask the agent to produce explicit blowup/flip/flop examples in coordinates when possible.
- Use counterexamples to clarify the sharpness of hypotheses (ask agent: "provide a counterexample if X is removed").

---

## Recommended file and versioning practices

- Keep a working directory with files named by topic: e.g. "flop-summary.md", "adjoint-lemmas.tex", "examples/".
- Track assumptions in a single file ASSUMPTIONS.md to avoid losing which branch of conditional proofs you're using.
- Commit incremental proof sketches and label commits with the assumption tags used.

---

Short templates

- "Search template": `Search for papers on "adjoint foliated structure" and return titles, authors, year, and a one-sentence summary.`
- "Extract theorem template": `From [citation], extract Theorem X full statement and hypotheses in LaTeX.`
- "Compare template": `Compare proof strategy of [paper1] and [paper2] in bullet points focusing on the origin of the birational map and the lift used.`

End of guide.
