# Paper B — A Space-Compression Argument for the Best-Move Function in EXPTIME-Complete Games

## Title

**A Space-Compression Argument for the Best-Move Function in EXPTIME-Complete Games**

## Type

Type C — conditional impossibility / structural complexity result.

## Author

Lightman Chang
Independent Researcher
lightman.chang@gmail.com

## Summary

For two-player perfect-information games with EXPTIME-complete value problems (such as generalized Go, chess, and checkers), the natural self-reduction from the value problem to the best-move function $m_1$ fails because optimal-play trajectories can have superpolynomial length. We replace polynomial time with polynomial space: simulate optimal play using two position pointers and Floyd cycle detection, all within $\PSPACE$. The resulting reduction shows $m_1 \in \FP \Rightarrow \EXPTIME = \PSPACE$ for any compactly representable EXPTIME-complete game.

As a structural by-product we obtain a meta-theorem: every compactly representable EXPTIME-complete game has worst-case game length not bounded by any polynomial in $n$, conditional only on $\EXPTIME \neq \PSPACE$. This identifies game length as the structural reason why time-based self-reductions cannot succeed for these games.

The paper avoids any Robson-specific gadget hypothesis and relies only on (i) polynomial position size, (ii) memoryless optimal play (which holds under basic ko but fails under super-ko), and (iii) EXPTIME-completeness of the value problem.

## Key Result

**Theorem A (Space compression).** For any compactly representable EXPTIME-complete game, $m_1 \in \FP \Rightarrow \EXPTIME = \PSPACE$.

**Theorem B (Game-length meta-theorem).** Every compactly representable EXPTIME-complete game has worst-case game length $L_\mathcal{G}(n)$ that is not polynomially bounded, conditional on $\EXPTIME \neq \PSPACE$.

**Corollaries.** Specializations to generalized Go (basic ko), generalized chess, and generalized checkers.

## Target Journals (ranked)

1. **Computational Complexity** (Springer) — best fit; specializes in conditional separation results. Estimated acceptance: 35–45% after revision.
2. **Theoretical Computer Science** (Elsevier) — strong fit for game complexity. Estimated: 40–50%.
3. **Information Processing Letters** — possible if compressed to a 3-page note. The space-compression argument and meta-theorem are clean enough for this venue. Estimated: 50–60%.
4. **ACM Transactions on Computation Theory** — possible secondary venue.

## MSC 2020 Classification

- 91A46 (Combinatorial games) — primary
- 68Q15 (Complexity classes)
- 68Q17 (Computational difficulty of problems)
- 68Q05 (Models of computation)

## Submission Strategy

1. Post to **arXiv** (cs.CC + cs.GT) for community feedback.
2. The meta-theorem (Theorem B) is the most novel piece — flag it in the cover letter.
3. Submit to Computational Complexity or TCS.
4. Consider IPL as a fast-publication backup.

## Must-Cite Related Work

- Robson (1983): EXPTIME-completeness of Go — base reduction
- Robson (1984): EXPTIME-completeness of checkers
- Fraenkel & Lichtenstein (1981): EXPTIME-completeness of chess
- Lichtenstein & Sipser (1980): PSPACE-hardness of Go
- Floyd (1967): cycle detection algorithm
- Hartmanis & Stearns (1965): time hierarchy theorem
- Berlekamp & Wolfe (1994): exact CGT formulas — what we don't address
- Conway (1976): foundational CGT
- Adachi-Kamio-Iwata-style results on PSPACE-complete games — contrast

## Known Limitations

1. **Conditional only.** The conclusion is $m_1 \in \FP \Rightarrow \EXPTIME = \PSPACE$, not unconditional non-existence. Whether $\EXPTIME = \PSPACE$ is open.
2. **Basic ko only for Go.** Memoryless optimal play (Remark 3.5) requires that the position encoding capture all rule-relevant state. Super-ko requires history, which violates the polynomial position-size assumption.
3. **No improvement over Robson for $V$ itself.** Theorem A is a reduction from $V$-DEC to $m_1$; Robson already gives EXPTIME-completeness of $V$. Our contribution is methodological: showing how to bridge the $V$-to-$m_1$ gap that the standard self-reduction cannot bridge.
4. **The meta-theorem (Theorem B) is folklore-adjacent.** The argument is essentially "if game length is polynomial, depth-bounded minimax decides $V$ in PSPACE." We do not claim novelty for the technique itself; the contribution is the explicit statement and the connection it makes to $m_1$ lower bounds.

## Reviewer's Most Likely Objection

"Theorem B is folklore — depth-bounded minimax in PSPACE is well-known."

**Response strategy.** Acknowledge the folklore nature in Section 4.4. Position the contribution as: (i) the explicit framework of compactly representable games, which makes the meta-theorem applicable across multiple games at once; (ii) the connection to the $m_1$ lower bound via Theorem A; (iii) the negative implication for self-reduction strategies. We are not claiming a new technique; we are crystallizing a structural phenomenon.

A secondary objection: "$\EXPTIME = \PSPACE$ is so unlikely that the result feels weak." Response: the result is the strongest currently provable; further strengthening would require resolving $\EXPTIME$ vs $\PSPACE$.

## Independence from Paper A

Paper B is fully independent of Paper A:
- Paper A uses the Robson reduction's gadget structure (modified to satisfy A1–A3 and R1) and the bottleneck/pass-inequality construction to attack $m_2$.
- Paper B uses no gadget-level hypothesis. It only needs polynomial position size, polynomial-time transition, and EXPTIME-completeness of the value problem — all standard inputs.

The two papers can be read in either order; they share Section 1's motivation but no proofs.

## File Info

- **Source:** paper.tex
- **Length target:** 4–5 pages compiled
- **Expected PDF size:** ~140 KB
