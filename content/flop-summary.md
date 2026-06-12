# Flop Summary

This note summarizes the main results and proof ideas of Hashizume (2020) and Chen--Liu--Wang (2025) in the context of flops connecting minimal models, and explains how to set up a Hashizume-style approach for algebraically integrable foliations in general dimension. All mathematical nouns and technical terms are written in English; formulas use LaTeX.

Contents

- Main theorems and proof ideas: Hashizume (Has20) and CLW25
- Has20-style setup for lc foliations: how to specify the initial birational geometry
- Constructing a Has20-style "good minimal model" lift for foliations
- Potential problems and strategies
- What adjoint foliated structures enable and associated ideas
- Two tracks: accepting the standard MMP-like hypotheses vs working without them
- References

---

## 1. Main theorems and proof ideas

Notation: work over the complex numbers. For a foliated pair we write $(X,\mathcal{F},\Delta)$ where $X$ is projective, $\mathcal{F}$ is an algebraically integrable foliation, and $\Delta$ is an effective \(\mathbb{Q}\)-divisor compatible with $\mathcal{F}$ as in the adjoint foliated-structure literature. The adjoint divisor is

$$
K_{\mathcal{F}}+\Delta.
$$

- Has20 (Hashizume, 2020). Main theorem (informal): For a fixed log canonical pair $(X,\Delta)$ and for any two log minimal models $X_1, X_2$ obtained by log MMP from $(X,\Delta)$, there exist small birational models of $X_1$ and $X_2$ that are connected by a sequence of flops. Proof idea: work on a common resolution, use crepant comparisons of canonical classes, produce a single model $Y$ (often via canonical ring / Proj) that centralizes the canonical data, and then show that the birational relation factors through $Y$ and decomposes into flops between chambers of the movable cone. The Hashizume argument emphasizes starting from a crepant birational relation and building a "good minimal model" lift that makes the flop decomposition transparent.

- CLW25 (Chen, Liu, Wang, 2025). Main theorem (informal): For algebraically integrable foliated triples on potentially klt varieties, any two minimal models (in the foliated sense) are connected by a sequence of flops; when foliation is algebraically integrable the foliated MMP (cone theorem, contractions, flips) can be run in the expected way. Proof idea: work in the foliated MMP framework, Q-factorialize when needed, run two foliated MMPs, then compare outputs; use algebraic integrability to reduce local foliation issues and ensure flops are foliation-compatible. The CLW25 approach is operational: Q-factorialize, run MMPs, then relate the outputs.

Key contrast (Has20 vs CLW25):

- Has20 starts from a crepant birational setup (specific birational map satisfying crepant-type conditions) and builds a central model $Y$ (a good minimal model or Proj of a canonical ring) to which both outputs relate as small models; it then decomposes the map into flops on $Y$.
- CLW25 constructs the birational map by running two (foliated) MMPs from a common origin; the induced birational map is therefore produced by the MMP runs and they use Q-factorialization as the preferred lift in non-Q-factorial situations.

---

## 2. Has20-style setup for lc foliations: initial birational geometry

Goal: given two foliated minimal models $X_1, X_2$ of the same initial adjoint foliated structure, produce a Has20-style crepant setup and a single model $Y$ so that $X_1$ and $X_2$ are small models of $Y$ and the birational map $X_1\dashrightarrow X_2$ factors through $Y$ and decomposes into flops.

2.1. Requirements on the birational map

Following Hashizume, the starting birational map should satisfy crepant-type conditions on canonical classes. Concretely, pick a common resolution

$$
\xymatrix{ & W \ar[dl]_{\rho_1} \ar[dr]^{\rho_2} & \
X_1 \ar@{-->}[rr]^{\phi} & & X_2 }
$$

and require (in the foliated sense) on $W$ that

$$
\rho_1^*(K_{\mathcal{F}_1}+\Delta_1) = \rho_2^*(K_{\mathcal{F}_2}+\Delta_2) =: \Gamma_W.
$$

This equality is the crepant equality: the two models are crepant equivalent via $W$. Hashizume then uses this equality to centralize the canonical divisor.

2.2. How the map arises in practice

- CLW25 case: the map is induced by two foliated MMP runs from the same origin (after Q-factorialization). The map is therefore canonical from the MMP outputs.
- Has20 case: the map may be given abstractly (assume crepant equivalence) and then used to construct a common model $Y$ via canonical ring methods. For foliations we mimic this by demanding foliated crepant equivalence on a common resolution.

  2.3. Translation to foliated setting

Replace $K_X+\Delta$ by $K_{\mathcal{F}}+\Delta$ and use the notion of foliated discrepancy. The crepant condition becomes

$$
\rho_1^*(K_{\mathcal{F}_1}+\Delta_1)=\rho_2^*(K_{\mathcal{F}_2}+\Delta_2)
$$

as divisors on the common resolution $W$. This equality must be interpreted using foliated discrepancy theory (use references in Section "References").

---

## 3. Constructing a Has20-style good minimal model lift for foliations

High level prescription (general dimension, algebraically integrable foliation):

1. Take a foliated dlt modification $\mu\colon V\to X$ so that $(V,\mathcal{F}_V,\Delta_V)$ is foliated-dlt and $K_{\mathcal{F}_V}+\Delta_V$ is \(\mathbb{Q}\)-Cartier. (This is a standard reduction step; refer to Cascini et al. for algebraically integrable adjoint structures.)

2. Consider the adjoint ring (foliated canonical ring)

   $$
   R(V, K_{\mathcal{F}_V}+\Delta_V)=\bigoplus_{m\ge 0} H^0\big(V, \lfloor m(K_{\mathcal{F}_V}+\Delta_V)\rfloor\big).
   $$

   If $R$ is finitely generated then set

   $$
   Y:=\operatorname{Proj} R(V,K_{\mathcal{F}_V}+\Delta_V).
   $$

   Then $Y$ is a good minimal model for the adjoint foliated structure on $V$ and $K_{\mathcal{F}_Y}+\Delta_Y$ is semiample.

3. Show that $X_1$ and $X_2$ are small birational models of $Y$: because both models come from $V$ by running foliated MMP in different chambers, their maps factor through $Y$ and the pullbacks of the adjoint divisors agree on a common resolution. Conclude that the birational map $X_1\dashrightarrow X_2$ factors through $Y$ and can be decomposed into flops inside the movable cone chambers of $Y$.

4. Verify foliation-compatibility: each flop must be foliation-compatible. For algebraically integrable foliations this is feasible because the foliation is induced by a fibration structure and local analysis (as in CLW25) shows flops can be chosen to preserve the foliation class.

Remarks: the central steps that require external input are finite generation of the adjoint ring and existence of foliated dlt modifications; these are addressed in the adjoint foliated MMP literature (Cascini et al.). If finite generation fails, one must work with a relative MMP target or keep the construction conditional.

---

## 4. Potential problems and strategies

List of likely obstacles and approaches.

- Finite generation of $R(V,K_{\mathcal{F}_V}+\Delta_V)$.
  - Strategy if accepted: cite results in the adjoint foliated-structure program (Cascini et al.) that establish finite generation in the algebraically integrable adjoint setting under klt hypotheses.
  - Strategy if not accepted: keep statements conditional; or replace Proj construction by producing a relative good model via relative MMP and nef reduction (work with pseudoeffective cone and movable cone decomposition instead of Proj).

- Existence of foliated dlt modification.
  - Strategy if accepted: use construction in Cascini et al. or CLW25 for algebraically integrable cases.
  - If not accepted: restrict to cases where a foliated dlt modification is known (surface case, dimension 3 under extra hypotheses) or assume a dlt modification exists as a hypothesis.

- Flops preserving foliation.
  - For algebraically integrable foliations, reduce to base changes along the fibration defining the foliation; use CLW25 local analysis to ensure flops can be chosen foliation-compatible.

- Non-Q-factorial starting models.
  - Hashizume suggests using a good minimal model lift (via Proj) rather than Q-factorialization. That approach requires finite generation. CLW25 prefers Q-factorialization; both strategies are valid depending on which hypotheses one accepts.

---

## 5. What adjoint foliated structures enable

Working in the adjoint foliated-structure framework expands available tools:

- Cone theorem, contraction theorem, existence of flips in the algebraically integrable adjoint category (Cascini et al.) under klt and Q-factorial hypotheses.
- Finite generation of adjoint rings and existence of good minimal models for adjoint foliated structures of general type (Cascini et al. preprints/manuscripts).

Consequences for the Has20-style program:

- If adjoint ring finite generation holds, the Proj construction supplies the central model $Y$ and Hashizume-style lifting is straightforward (subject to verifying crepant equalities in the foliated sense).
- The adjoint framework clarifies the discrepancy computations for foliations and allows the use of canonical ring techniques familiar from non-foliated MMP.

---

## 6. Two tracks: accepting vs not accepting hypotheses

Track A: Accept standard adjoint-foliated-MMP hypotheses (recommended for general-dimension statements)

- Hypotheses:
  1. Existence of foliated dlt modification for algebraically integrable adjoint foliated structures.
  2. Finite generation of the adjoint ring $R(V,K_{\mathcal{F}_V}+\Delta_V)$ in the relevant class.
  3. Cone, contraction, flips and termination properties in the algebraically integrable adjoint category (use Cascini et al.).

- Consequences:
  - You can construct $Y=\operatorname{Proj} R$ and realize $X_1,X_2$ as small models of $Y$.
  - The birational map decomposes into foliation-compatible flops on $Y$.
  - Many arguments from Hashizume lift verbatim, replacing $K_X+\Delta$ by $K_{\mathcal{F}}+\Delta$ and using foliated discrepancy.

Track B: Do not accept those hypotheses (conservative)

- Strategy:
  - Restrict to lower-dimensional cases (surfaces; dimension 3) where more is known, or to special algebraically integrable foliations coming from fibrations with good base MMP behaviour.
  - Keep statements conditional: prove "if there exists a good minimal model Y for $K_{\mathcal{F}}+\Delta$ then...".
  - Use Q-factorialization + CLW25-style comparison when Q-factorialization is feasible; accept that this yields a different lift and that some Hashizume-style conclusions might require stronger hypotheses.

---

## 7. References

- Kaw08: Yujiro Kawamata, "Flops connect minimal models".
- Hash20: Kenta Hashizume, "Relations between two log minimal models of log canonical pairs", Int. J. Math., 2020.
- CLW25: Yifei Chen, Jihao Liu, Yanze Wang, "Flop between algebraically integrable foliations on potentially KLT varieties", Intl. J. Math., 2025.
- Casc24: Paolo Cascini, Jingjun Han, Jihao Liu, Fanjun Meng, Calum Spicer, Roberto Svaldi, Lingyao Xie, "Minimal model program for algebraically integrable adjoint foliated structures" (preprint, 2024).
- Casc24-II / Casc25: sequels and manuscripts by the same group on finite generation and good minimal models for adjoint foliated structures (see Cascini et al. preprints/manuscripts).
- Lu+Wu+Xu papers on adjoint divisors for foliated surfaces (useful for surface case and local analysis).

---

End of summary.
