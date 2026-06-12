# Length of Extremal Rays and Characterizations of Projective Space
## — A Survey with Focus on the Foliation Version

---

## Part I. Survey of History and Known Results

### 1.1 Length of Extremal Rays in Classical Mori Theory

Let $X$ be a $\mathbb{Q}$-factorial projective variety. An *extremal ray* $R \subset \overline{NE}(X)$ is a $K_X$-negative ray that can be contracted. The *length* of $R$ is defined by

$$\ell(R) = \min\{ -K_X \cdot C \mid [C] \in R,\ C \text{ a rational curve} \}.$$

A classical result states that for an $n$-dimensional $\mathbb{Q}$-factorial Fano variety with Picard number $\rho(X) = 1$, the length of the unique extremal ray satisfies $\ell(R) \leq n + 1$, with equality if and only if $X \cong \mathbb{P}^n$. For the toric case, **FI13** proved that the set of possible lengths for $n$-dimensional $\mathbb{Q}$-factorial toric Fano varieties with Picard number one satisfies the ascending chain condition (ACC). Recently, **FJLR26** gave a new complex-analytic proof that characterizes $\mathbb{P}^n$ by the property that the length of an extremal ray attains the maximal value $\dim X + 1$.

Meanwhile, the bend-and-break technique of Mori gives the bound $\ell(R) \leq \dim X + 1$ for any $K_X$-negative extremal ray on a smooth variety. The optimal constant $n+1$ realizes the Mori cone structure of $\mathbb{P}^n$.

### 1.2 Fano Foliations: The Foliation Version of Fano Theory

A *foliation* $\mathcal{F} \subset T_X$ on a normal projective variety $X$ is a saturated subsheaf of the tangent sheaf that is closed under the Lie bracket. Its canonical class is $K_{\mathcal{F}} = -c_1(\mathcal{F})$. The anti-canonical class is $-K_{\mathcal{F}} = c_1(\mathcal{F})$.

In the seminal paper **AD13**, Araujo and Druel initiated the systematic study of Fano foliations: $\mathcal{F}$ is called a *$\mathbb{Q}$-Fano foliation* if its anti-canonical class $-K_{\mathcal{F}}$ is an ample $\mathbb{Q}$-Cartier divisor. The *index* $i_{\mathcal{F}}$ is defined as the largest rational number such that $-K_{\mathcal{F}} \equiv i_{\mathcal{F}} H$ for some Cartier divisor $H$. A Fano foliation of rank $r$ and index $i_{\mathcal{F}} = r-1$ is called a *del Pezzo foliation*.

**AD13** classified del Pezzo foliations on smooth complex projective manifolds: they are algebraically integrable with exactly one exceptional case — when the ambient variety is a projective space. This exceptional case provides the first bridge between Fano foliations and characterizations of $\mathbb{P}^n$.

**AD14a** extended the theory to the singular setting, proving a Kobayashi–Ochiai-type theorem for $\mathbb{Q}$-Fano foliations: under mild conditions, $i_{\mathcal{F}} \leq \operatorname{rank}(\mathcal{F}) = r$, with a full classification of the equality case $i_{\mathcal{F}} = r$. Codimension 1 del Pezzo foliations on mildly singular varieties are then completely classified. **AD14b** further studied the Picard number one case.

### 1.3 Length of Extremal Rays for Foliations

For a foliation $\mathcal{F}$, one can mimic the Mori-theoretic definition: a $K_{\mathcal{F}}$-negative extremal ray $R$ has *length*

$$\ell_{\mathcal{F}}(R) = \min\{ -K_{\mathcal{F}} \cdot C \mid [C] \in R,\ C \text{ a rational curve} \}.$$

**FS24** established a striking result for toric foliations: if a toric foliation $\mathcal{F}$ of rank $r$ on a $\mathbb{Q}$-factorial projective toric variety has an extremal ray whose length satisfies $\ell_{\mathcal{F}}(R) > r$, then the extremal contraction $\varphi_R : X \to Y$ is a $\mathbb{P}^r$-bundle and $\mathcal{F}$ coincides with the *relative tangent sheaf* $T_{X/Y}$ of the contraction. In other words: for toric foliations, a length exceeding the rank forces the foliation to be the "most rigid" possible one — the relative tangent sheaf of a projective space bundle.

This is the foliation analog of the classical statement that $\ell(R) = \dim X + 1$ characterizes $\mathbb{P}^n$. The bound $r$ (rank) plays the role of $\dim X + 1$ (dimension plus one).

### 1.4 Algebraic Integrability of Leaves

A foliation $\mathcal{F}$ is *algebraically integrable* if its leaves are algebraic subvarieties (i.e., it is induced by a rational map $X \dashrightarrow Z$). Determining algebraic integrability of the leaves is a central problem.

**KCT06** gave the first modern treatment following Bogomolov–McQuillan: on a normal variety, under certain positivity conditions, a foliation has algebraic and rationally connected leaves. The proof uses only standard Mori theory and a vanishing theorem for vector bundles in positive characteristic.

**HP19** proved that if a foliation $\mathcal{F}$ has numerically trivial canonical bundle $K_{\mathcal{F}} \equiv 0$, is sufficiently stable, and satisfies $c_2(\mathcal{F}) \neq 0$, then $\mathcal{F}$ is algebraically integrable.

**AD19** studied the positivity of distributions on complex projective manifolds, obtaining characterizations of generic $\mathbb{P}^r$-bundles in terms of movable curve classes. This was then applied to give a lower bound on the *algebraic rank* (the maximal rank of an algebraically integrable sub-foliation) in terms of positivity invariants, with a classification of foliations attaining the bound.

**JL23** established a Kobayashi–Ochiai theorem for Fano foliations in terms of algebraic rank, gave lower bounds via Seshadri constants, and classified foliations attaining the bound. Several examples answered open questions of Araujo–Druel.

**LX25** proved a *non-algebraicity criterion*: for a log canonical foliation of rank $\leq d$, the inequality of Kodaira dimensions $\nu \neq \kappa$ implies that $\mathcal{F}$ is *not* algebraically integrable. This answered a question of Ambro–Cascini–Shokurov–Spicer. The result is unconditional in dimension $\leq 3$.

### 1.5 Bend-and-Break for Foliations

The classical bend-and-break technique says that for any rational curve $C$ with $-K_X \cdot C > \dim X + 1$, the curve must break. **LSJ26** proved the optimal foliation version: for a foliation $\mathcal{F}$ of rank $r$, the optimal bend-and-break constant for rational curves tangent to $\mathcal{F}$ is $r+1$. The proof combines the Bogomolov–McQuillan method with the bend-and-shatter technique developed by Jovinelly–Lehmann–Riedl.

This result provides the technical underpinning for length bounds: $\ell_{\mathcal{F}}(R) \leq r+1$ is the natural optimal bound for $K_{\mathcal{F}}$-negative extremal rays, generalizing the classical $\ell(R) \leq \dim X + 1$.

### 1.6 The Minimal Model Program for Algebraically Integrable Foliations

Once a foliation is known to be algebraically integrable, one can run the MMP. **CS23** proved that termination of flips for $\mathbb{Q}$-factorial klt pairs in dimension $r$ implies the existence of minimal models for lc algebraically integrable foliations of rank $r$.

**LMX24** made a major breakthrough: without assuming termination of flips, they proved the base-point-freeness theorem, contraction theorem, and existence of flips for lc algebraically integrable foliations on klt varieties. This resolved the Cascini–Spicer conjecture and established the full MMP. As a corollary, $\mathbb{Q}$-factorial klt varieties with lc algebraically integrable Fano foliation structures are Mori dream spaces.

**CLW25** proved that minimal models of lc algebraically integrable foliated triples are connected by flops, and **CLW26** established the Sarkisov program for algebraically integrable foliations.

---

## Part II. Main Ideas of Key Papers

### FJLR26 — Fujino, Jovinelly, Lehmann, Riedl (2026)
**A characterization of projective space via lengths of extremal rays.** The paper establishes a new proof that a $\mathbb{Q}$-factorial Fano variety $X$ with $\rho(X) = 1$ whose extremal ray attains length $\dim X + 1$ is isomorphic to $\mathbb{P}^n$. The approach combines bend-and-shatter techniques with the analysis of families of rational curves, refining the classical Mori-type argument.

### FS24 — Fujino, Sato (2024)
**A remark on toric foliations.** The key idea: on a toric variety, the Mori cone and foliations admit a combinatorial description via fans. For a toric foliation $\mathcal{F}$ of rank $r$, if an extremal ray generated by a torus-invariant curve $C$ satisfies $-K_{\mathcal{F}} \cdot C > r$, then the corresponding wall in the fan forces the extremal contraction to be a $\mathbb{P}^r$-bundle. The foliation is then forced to be $T_{X/Y}$. The proof is combinatorial: the inequality on intersection numbers translates into a statement about primitive relations in the fan.

### AD13 — Araujo, Druel (2013)
**On Fano foliations.** The paper develops the foundational theory: defines $\mathbb{Q}$-Fano foliations, establishes basic properties (rational curves tangent to $\mathcal{F}$ always exist), and classifies del Pezzo foliations on smooth manifolds. The method uses the algebraic integrability criterion (Bogomolov–McQuillan) together with an analysis of the structure of the family of rational curves tangent to $\mathcal{F}$. The exceptional non-algebraically-integrable case occurs precisely on projective space, establishing the first link between Fano foliations and $\mathbb{P}^n$.

### AD14a — Araujo, Druel (2014)
**On codim 1 del Pezzo foliations on singular varieties.** Extends AD13 to the singular setting. The Kobayashi–Ochiai-type bound $i_{\mathcal{F}} \leq r$ is proved by considering the Harder–Narasimhan filtration of $\mathcal{F}$ and using Miyaoka's semi-positivity theorem. The classification of $i_{\mathcal{F}} = r$ involves showing that $X$ admits a $\mathbb{P}^r$-bundle structure, with $\mathcal{F}$ being the relative tangent sheaf. The classification of $i_{\mathcal{F}} = r-1$ (del Pezzo) proceeds case-by-case using the structure of the family of minimal rational curves.

### JL23 — Liu Jie (2023)
**Fano foliations with small algebraic ranks.** Introduces algebraic rank as a tool: for a Fano foliation, the algebraic rank $r_a(\mathcal{F})$ is the maximal rank of an algebraically integrable sub-foliation. The Kobayashi–Ochiai bound becomes $i_{\mathcal{F}} \leq r_a(\mathcal{F})$. Uses Seshadri constants of $-K_{\mathcal{F}}$ to bound algebraic rank from below: $\varepsilon(-K_{\mathcal{F}}, x) \geq 1$ for general $x$ implies $r_a(\mathcal{F}) \geq i_{\mathcal{F}}$. Classification results employ the minimal rational curve approach.

### KCT06 — Kebekus, Solá Conde, Toma (2006)
**Rationally connected foliations after Bogomolov and McQuillan.** The main
technical tool is a theorem of Bogomolov–McQuillan: a foliation whose restriction
to a general complete intersection curve is ample (in the sense of Hartshorne)
has algebraic and rationally connected leaves. The paper provides a clean proof
combining this with the Graber–Harris–Starr theorem on rationally connected
fibrations. A key ingredient is a vanishing theorem for vector bundles in
positive characteristic used to verify the ampleness condition.

### HP19 — Höring, Peternell (2019)
**Algebraic integrability with $K_{\mathcal{F}} \equiv 0$.** The main new idea: if
$\mathcal{F}$ is a stable reflexive sheaf with $c_1(\mathcal{F}) = 0$ and $c_2(\mathcal{F}) \neq 0$,
then $\mathcal{F}$ is flat on some open set. Flatness implies that the foliation
has algebraic leaves, using a criterion involving the algebraicity of the
foliation on the projectivized tangent bundle. The stability condition ensures
that $\mathcal{F}$ cannot decompose as a direct sum.

### AD19 — Araujo, Druel (2019)
**Positivity, generic $\mathbb{P}^r$-bundles, and algebraicity.** The paper develops
a theory of "movable curve class" positivity for distributions. A distribution
with big slope with respect to movable curves induces a rational map whose
generic fiber is $\mathbb{P}^r$. Applied to foliations: the algebraic rank $r_a$
is bounded below by the maximum slope of $\mathcal{F}$. This provides a
differential-geometric criterion for algebraic integrability.

### LSJ26 — Liu, Sun, Jiang (2026)
**Optimal bend-and-break for foliations.** The proof scheme: (1) assume a
rational curve $C$ tangent to $\mathcal{F}$ has $-K_{\mathcal{F}} \cdot C > r+1$;
(2) use the Bogomolov–McQuillan method to produce a surface covered by rational
curves tangent to $\mathcal{F}$; (3) apply the bend-and-shatter technique of
Jovinelly–Lehmann–Riedl to "shatter" the curve into components with smaller
degree; (4) reaching a contradiction when the degree is below the threshold.
The result is sharp: on $\mathbb{P}^r$ (foliation by points), lines satisfy
$-K_{\mathcal{F}} \cdot \ell = r+1$.

### LX25 — Liu, Xu (2025)
**Non-algebraicity via Kodaira dimensions.** The key idea: for a foliation $\mathcal{F}$,
define its Kodaira dimension $\kappa(\mathcal{F})$ and numerical dimension
$\nu(\mathcal{F})$ (using the canonical class $K_{\mathcal{F}}$). If $\mathcal{F}$ were
algebraically integrable with base $Z$, then $\kappa(\mathcal{F}) = \nu(\mathcal{F}) =
\dim Z$. Hence $\nu \neq \kappa$ is an obstruction to algebraic integrability.
The proof uses abundance-type arguments for adjoint foliated structures.

### LMX24 — Liu, Meng, Xie (2024)
**MMP for algebraically integrable foliations on klt varieties.** The core
technical innovation is the systematic use of *adjoint foliated structures*
(generalized foliated quadruples). Given an algebraically integrable foliation
$\mathcal{F}$ with first integral $X \dashrightarrow Z$, the MMP on $X$ relative
to $Z$ is linked to the MMP for a generalized pair structure on $Z$. Base-point-freeness
is proved via the canonical bundle formula, and flips are constructed by
reducing to the classical flip for generalized pairs.

---

## Part III. Conjectures

Based on the surveyed results, we propose several conjectures concerning the
characterization of varieties and foliations when the length of extremal rays
attains the optimal bound.

### Conjecture A (Generalized Length Bound)

> Let $\mathcal{F}$ be a foliation of rank $r$ with F-dlt singularities on a
> $\mathbb{Q}$-factorial klt projective variety $X$. Let $R$ be a
> $K_{\mathcal{F}}$-negative extremal ray. Then
>
> $$\ell_{\mathcal{F}}(R) \leq r + 1.$$

**Motivation:**
- Classical case ($\mathcal{F} = T_X$, $r = n$): $\ell(R) \leq n+1$, the standard Mori bound, with equality characterizing $\mathbb{P}^n$.
- **LSJ26** proves that the optimal bend-and-break constant for rational curves tangent to a rank $r$ foliation is exactly $r+1$. This strongly suggests that $\ell_{\mathcal{F}}(R) \leq r+1$ is the natural optimal bound.
- **FS24** (toric case): if $\ell_{\mathcal{F}}(R) > r$, then $X \to Y$ is a $\mathbb{P}^r$-bundle and $\mathcal{F} = T_{X/Y}$. This shows the bound $r$ is sharp in the toric setting, but the inequality $\ell_{\mathcal{F}}(R) \leq r+1$ would still hold.

The gap between $r$ and $r+1$ is subtle: FS24 shows that *exceeding* $r$ already forces very rigid structure (relative tangent of a $\mathbb{P}^r$-bundle). But the optimal bend-and-break constant $r+1$ suggests that $\ell_{\mathcal{F}}(R) = r+1$ may be attained by foliations that are *not* relative tangent sheaves, specifically the *foliation by points* on $\mathbb{P}^r$, for which $-K_{\mathcal{F}} \cdot \ell = r+1$ for a line $\ell$.

### Conjecture B (Characterization of Equality $\ell_{\mathcal{F}}(R) = r+1$)

> Let $\mathcal{F}$ be a $\mathbb{Q}$-Fano foliation of rank $r$ with
> canonical singularities on a $\mathbb{Q}$-factorial klt Fano variety $X$
> with $\rho(X) = 1$. Suppose $\ell_{\mathcal{F}}(R) = r+1$. Then $X \cong \mathbb{P}^r$
> and $\mathcal{F} \cong T_{\mathbb{P}^r}$ (the foliation by points).
> In particular, $\dim X = r$.

**Motivation:**
- **AD13/AD14a**: For a $\mathbb{Q}$-Fano foliation with $\rho = 1$, the length and the index coincide: $\ell_{\mathcal{F}}(R) = i_{\mathcal{F}}$. The Kobayashi–Ochiai theorem says $i_{\mathcal{F}} \leq r$, and the case $i_{\mathcal{F}} = r$ is fully classified: it yields a $\mathbb{P}^r$-bundle structure.
- **FJLR26**: For the classical tangent foliation ($r = \dim X$), $\ell(R) = n+1$ characterizes $\mathbb{P}^n$.
- **AD13** (del Pezzo, $i_{\mathcal{F}} = r-1$): algebraically integrable except on $\mathbb{P}^n$.
- Thus the "extremal" case $\ell_{\mathcal{F}}(R) = r+1$ should correspond to the maximal possible index $i_{\mathcal{F}} = r$, forcing the ambient variety to be $\mathbb{P}^r$ with the foliation-by-points.

### Conjecture C (Generalization of FS24: $\ell_{\mathcal{F}}(R) > r$)

> Let $\mathcal{F}$ be a foliation of rank $r$ with F-dlt singularities on a
> $\mathbb{Q}$-factorial klt projective variety $X$. Let $R$ be a
> $K_{\mathcal{F}}$-negative extremal ray such that
> $\ell_{\mathcal{F}}(R) > r$. Then:
>
> 1. The extremal contraction $\varphi_R : X \to Y$ is a $\mathbb{P}^r$-bundle
>    (in the orbifold sense if $X$ is singular), and
> 2. $\mathcal{F} = T_{X/Y}$ is the relative tangent sheaf of $\varphi_R$.

**Motivation:**
- **FS24** proved exactly this for toric foliations on toric varieties. The conjecture proposes that the toric hypothesis can be removed.
- The gap $\ell_{\mathcal{F}}(R) \in (r, r+1]$ contains no integers, so $\ell_{\mathcal{F}}(R) > r$ is equivalent to $\ell_{\mathcal{F}}(R) \geq r+1$ *for integral values*. In light of Conjecture B, $\ell_{\mathcal{F}}(R) = r+1$ with $\rho = 1$ gives $\mathbb{P}^r$ and the foliation by points. For $\rho > 1$, $\ell_{\mathcal{F}}(R) > r$ should give a $\mathbb{P}^r$-bundle structure with $\mathcal{F} = T_{X/Y}$.
- The proof strategy would likely combine **LSJ26** (bend-and-break) with the algebraic integrability criteria of **KCT06**, **HP19**, and **AD19**, and the MMP for foliations from **LMX24**.

### Conjecture D (ACC for Lengths of Extremal Rays of Foliations)

> Fix integers $n, r$ with $r \leq n$. The set of lengths
> $\ell_{\mathcal{F}}(R)$ for $\mathbb{Q}$-Fano foliations $\mathcal{F}$ of rank $r$
> with F-dlt singularities on $n$-dimensional $\mathbb{Q}$-factorial klt
> Fano varieties satisfies the ACC.

**Motivation:**
- **FI13** proved the ACC for toric Fano varieties with $\rho = 1$ in the classical setting.
- **DLM23** (Das–Liu–Mascharak) proved the ACC for lc thresholds for algebraically integrable foliations, providing a model result for ACC-type statements in foliation theory.
- The conjecture would unify FI13 (classical ACC) with DLM23 (foliation ACC for thresholds).

### Conjecture E (Algebraic Integrability and Length)

> Let $\mathcal{F}$ be a $\mathbb{Q}$-Fano foliation of rank $r$ with
> canonical singularities on a $\mathbb{Q}$-factorial klt projective variety $X$.
> Let $R$ be a $K_{\mathcal{F}}$-negative extremal ray.
>
> 1. If $\ell_{\mathcal{F}}(R) < r$, then $\mathcal{F}$ is algebraically integrable.
> 2. If $\ell_{\mathcal{F}}(R) = r$, then either $\mathcal{F}$ is
>    algebraically integrable, or $X$ is a $\mathbb{P}^r$-bundle and
>    $\mathcal{F} = T_{X/Y}$.
> 3. If $\ell_{\mathcal{F}}(R) = r+1$, then $\mathcal{F}$ is *not*
>    algebraically integrable, and $X \cong \mathbb{P}^r$ with
>    $\mathcal{F} \cong T_{\mathbb{P}^r}$.

**Motivation:**
- **AD13**: del Pezzo foliations ($i_{\mathcal{F}} = r-1$) are algebraically integrable, with the sole exception on $\mathbb{P}^n$.
- **LX25**: If $\nu(\mathcal{F}) \neq \kappa(\mathcal{F})$, then $\mathcal{F}$ is not algebraically integrable. In the Fano case, $\kappa(\mathcal{F}) = -\infty$, so $\nu \neq \kappa$ reduces to $\nu(\mathcal{F}) \neq -\infty$, which is related to positivity of $-K_{\mathcal{F}}$.
- **AD19**: algebraic rank is bounded below by positivity invariants of $-K_{\mathcal{F}}$, and $\ell_{\mathcal{F}}(R)$ is a measure of this positivity.
- This conjecture proposes a precise trichotomy linking extremal ray length to algebraic integrability, unifying the observations from AD13, FS24, and FJLR26.

### Conjecture F (Foliated Kobayashi–Ochiai via Extremal Ray Length)

> Under the same hypotheses as Conjecture B, the extremal ray length satisfies
>
> $$\ell_{\mathcal{F}}(R) \leq r_a(\mathcal{F}) + 1,$$
>
> where $r_a(\mathcal{F})$ is the algebraic rank of $\mathcal{F}$.
> Equality holds iff $X \cong \mathbb{P}^{r_a}$ and $\mathcal{F}$ is the
> foliation by points.

**Motivation:**
- **JL23**: for Fano foliations, $i_{\mathcal{F}} \leq r_a(\mathcal{F})$, a strengthening of AD14a's $i_{\mathcal{F}} \leq r$.
- This would further refine Conjecture B, showing that it is the *algebraic* rank, rather than the geometric rank, that governs the length bound.

---

## Part IV. References

1. **AD13** — C. Araujo, S. Druel. *On Fano foliations.* Advances in Mathematics 238 (2013), 70–118. DOI: [10.1016/j.aim.2013.02.003](https://doi.org/10.1016/j.aim.2013.02.003). arXiv: [1112.4512](https://arxiv.org/abs/1112.4512).

2. **AD14a** — C. Araujo, S. Druel. *On codimension 1 del Pezzo foliations on varieties with mild singularities.* Mathematische Annalen 360 (2014), 769–798. DOI: [10.1007/s00208-014-1053-3](https://doi.org/10.1007/s00208-014-1053-3). arXiv: [1210.4013](https://arxiv.org/abs/1210.4013).

3. **AD14b** — C. Araujo, S. Druel. *On Fano foliations 2.* In: *Foliation Theory in Algebraic Geometry*, Springer, 2016, 1–13. DOI: [10.1007/978-3-319-24460-0_1](https://doi.org/10.1007/978-3-319-24460-0_1). arXiv: [1404.4628](https://arxiv.org/abs/1404.4628).

4. **AD19** — C. Araujo, S. Druel. *Characterization of generic projective space bundles and algebraicity of foliations.* Commentarii Mathematici Helvetici 94 (2019), 833–853. DOI: [10.4171/cmh/475](https://doi.org/10.4171/cmh/475). arXiv: [1711.10174](https://arxiv.org/abs/1711.10174).

5. **ADK08** — C. Araujo, S. Druel, S. Kovács. *Cohomological characterizations of projective spaces and hyperquadrics.* Inventiones Mathematicae 174 (2008), 233–253. DOI: [10.1007/s00222-008-0130-1](https://doi.org/10.1007/s00222-008-0130-1).

6. **CLW25** — Y. Chen, J. Liu, Y. Wang. *Flop between algebraically integrable foliations on potentially KLT varieties.* International Journal of Mathematics 36 (2025), 2550035. DOI: [10.1142/S0129167X25500351](https://doi.org/10.1142/S0129167X25500351).

7. **CLW26** — Y. Chen, J. Liu, Y. Wang. *Sarkisov Program for Algebraically Integrable and Three-fold Foliations.* International Mathematics Research Notices 2026(6) (2026), rnag045. DOI: [10.1093/imrn/rnag045](https://doi.org/10.1093/imrn/rnag045).

8. **CS23** — P. Cascini, C. Spicer. *MMP for algebraically integrable foliations.* Preprint (2023). arXiv: [2303.07528](https://arxiv.org/abs/2303.07528).

9. **DLM23** — O. Das, J. Liu, R. Mascharak. *ACC for lc thresholds for algebraically integrable foliations.* Preprint (2023). arXiv: [2307.08444](https://arxiv.org/abs/2307.08444).

10. **FI13** — O. Fujino, Y. Ishitsuka. *On the ACC for lengths of extremal rays.* Tohoku Mathematical Journal 65 (2013), 93–103. DOI: [10.2748/tmj/1365452627](https://doi.org/10.2748/tmj/1365452627).

11. **FJLR26** — O. Fujino, E. Jovinelly, B. Lehmann, E. Riedl. *A characterization of projective space via lengths of extremal rays.* Preprint (2026). arXiv: [2602.22192](https://arxiv.org/abs/2602.22192).

12. **FS24** — O. Fujino, H. Sato. *A remark on toric foliations.* Archiv der Mathematik 122 (2024), 621–627. DOI: [10.1007/s00013-024-01991-1](https://doi.org/10.1007/s00013-024-01991-1).

13. **HP19** — A. Höring, T. Peternell. *Algebraic integrability of foliations with numerically trivial canonical bundle.* Inventiones Mathematicae 216 (2019), 395–419. DOI: [10.1007/s00222-018-00853-2](https://doi.org/10.1007/s00222-018-00853-2). arXiv: [1710.06183](https://arxiv.org/abs/1710.06183).

14. **JL23** — J. Liu. *Fano foliations with small algebraic ranks.* Advances in Mathematics 423 (2023), 109038. DOI: [10.1016/j.aim.2023.109038](https://doi.org/10.1016/j.aim.2023.109038). arXiv: [2206.00840](https://arxiv.org/abs/2206.00840).

15. **KCT06** — S. Kebekus, L. Solá Conde, M. Toma. *Rationally connected foliations after Bogomolov and McQuillan.* Journal of Algebraic Geometry 16 (2007), 65–81. DOI: [10.1090/S1056-3911-06-00435-8](https://doi.org/10.1090/S1056-3911-06-00435-8). arXiv: [math/0505222](https://arxiv.org/abs/math/0505222).

16. **LMX24** — J. Liu, F. Meng, L. Xie. *Minimal model program for algebraically integrable foliations on klt varieties.* Preprint (2024). arXiv: [2404.01559](https://arxiv.org/abs/2404.01559).

17. **LSJ26** — J. Liu, Z. Sun, J. Jiang. *Optimal bend-and-break for foliations.* Preprint (2026). arXiv: [2605.20754](https://arxiv.org/abs/2605.20754).

18. **LX25** — J. Liu, Z. Xu. *Non-algebraicity of non-abundant foliations and abundance for adjoint foliated structures.* Preprint (2025). arXiv: [2510.04419](https://arxiv.org/abs/2510.04419).

