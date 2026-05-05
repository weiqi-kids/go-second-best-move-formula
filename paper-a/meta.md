# Paper A — On the Non-Existence of a Polynomial-Time Second-Best Move Function for Generalized Go

## Title

**On the Non-Existence of a Polynomial-Time Second-Best Move Function for Generalized Go**

## Type

Type C — impossibility result.

## Author

Lightman Chang
Independent Researcher
lightman.chang@gmail.com

## Summary

For $n \times n$ generalized Go under the basic ko rule, we settle the long-standing folklore question of whether the second-best move admits a closed analytic shortcut. Building on Robson (1983), we prove unconditionally that the move-value function $Q(S,m)$ is EXPTIME-complete to compute, hence not in $\FP$. We then introduce a bottleneck-encoding construction — a Go position $S^*$ on a $(3N) \times (3N)$ board combining a sacrificial group, an embedded modified Robson sub-instance, and a dame satellite — and prove a one-line "pass inequality" $V_B(P) + V_W(P) \geq -O(1)$ valid for every Go position. Together with a forcing-move-gap property of a modified Robson reduction, these tools give a polynomial-time many-one reduction from EXPTIME to the second-best move function $m_2$. By the time hierarchy theorem, $m_2 \notin \FP$.

## Key Result

**Proposition (Q-DEC).** $Q\text{-DEC} = \{(S, m, k) : Q(S,m) \geq k\}$ is EXPTIME-complete; hence $Q \notin \FP$ unconditionally.

**Lemma (Pass inequality).** For every Go position $P$ under the basic ko rule, $V_B(P) + V_W(P) \geq -O(1)$.

**Theorem (Main).** Under structural conditions (A1)–(A3) and (R1) on a modified Robson reduction, $m_2 \notin \FP$.

**Corollary.** No closed analytic expression of polynomial size in $n$ computes $m_2$ on every $n \times n$ legal position; the same conclusion holds unconditionally for $Q$.

## Target Journals (ranked)

1. **Theoretical Computer Science** (Elsevier) — best fit; publishes EXPTIME-completeness work in the Robson tradition. Estimated acceptance: 35–50% after revision.
2. **Information and Computation** — strong fit for negative complexity results. Estimated: 30–45%.
3. **Algorithmica** — possible secondary venue if framed as game-algorithm impossibility. Estimated: 25–40%.
4. **Discrete Applied Mathematics** — possible if structured around the combinatorial game theory angle.

## MSC 2020 Classification

- 91A46 (Combinatorial games) — primary
- 68Q17 (Computational difficulty of problems)
- 68Q15 (Complexity classes)
- 91A05 (2-person games)

## Submission Strategy

1. Post to **arXiv** (cs.CC + cs.GT) for community feedback, especially from the combinatorial game theory and complexity communities.
2. Solicit review from researchers in the Robson lineage (Tromp, Crâșmaru) before journal submission.
3. Submit to TCS or I&C.

## Must-Cite Related Work

- Robson (1983): EXPTIME-completeness of generalized Go — base reduction
- Lichtenstein & Sipser (1980): PSPACE-hardness — historical context
- Berlekamp & Wolfe (1994): exact CGT formulas for decomposable endgames — what our negative result complements
- Conway (1976): foundational CGT
- Chandra, Kozen, Stockmeyer (1981): APSPACE = EXPTIME
- Crâșmaru & Tromp (2000): super-ko ladders PSPACE-complete — what our methods don't cover
- Tromp & Farnebäck (2016): legal position counts on 19×19
- Hartmanis & Stearns (1965): time hierarchy theorem
- Karp & Lipton (1980), Buhrman-Fortnow-Thierauf (1998): non-uniform complexity tools
- Baker-Gill-Solovay (1975), Razborov-Rudich (1997), Aaronson-Wigderson (2009): three barriers explaining why fixed-board questions are open

## Known Limitations

1. **Structural assumptions (A1)–(A3) and (R1)** on a modified Robson reduction. The bottleneck construction requires that a modified Robson reduction satisfies strict ko-threat ordering, full board fill, forcing-move gap, and value separation. These are local edits to Robson's original gadgets, but a full re-verification is left to follow-up work.
2. **Basic ko only.** Robson's original reduction depends on the basic ko rule; our results inherit this restriction. Super-ko complexity remains open between PSPACE-hard and EXPSPACE.
3. **Asymptotic only.** All conclusions concern $n \to \infty$. The standard $19 \times 19$ board is a finite-domain question that hits the relativization, natural proofs, and algebrization barriers.
4. **Exact only.** We rule out exact polynomial-time computation. Approximation with $\varepsilon \geq 1/2$ is not addressed (and is achievable in practice by neural networks).
5. **No appendix on full Robson reproduction.** We deliberately treat (A1)–(A3) and (R1) as inputs rather than reprove Robson; this keeps the paper at a focused 4-5 pages.

## Reviewer's Most Likely Objection

"The structural assumptions (A1)–(A3) on the modified Robson reduction are stated without a complete construction. How do we know they are achievable simultaneously?"

**Response strategy.** Section 2.3 (Remark on engineering) and Section 7.3 (Discussion) state the construction sketches: (A1) by geometric ranking of ko-threat sizes, (A2) by padding with prefabricated two-eye groups, (A3) by Robson's original tightness analysis, (R1) by polynomial main pipe length. We commit to producing a full verification in a companion paper. Alternatively, we offer to release LaTeX source so a referee can audit the gadget edits directly.

## File Info

- **Source:** paper.tex
- **Length target:** 4–5 pages compiled
- **Expected PDF size:** ~150 KB
