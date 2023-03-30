# Plonkish backend optimizations

## Background

The relation described in [specification of the relation for a Plonkish circuit](https://github.com/daira/plonk-standard/blob/main/relation.md) gives an abstract model of a Plonkish constraint system that is sufficiently expressive, but does not take into account some of the optimizations that are commonly used in Plonkish implementations, such as the use of rotations in Plonkish custom gates. The tradeoff here is that the abstract model allows rows to be arbitrarily reordered.

> It would have been possible to directly include rotations in the constraint system model, but the most obvious ways of doing this would require the circuit programmer to think about copy constraints that don't naturally arise from the intended circuit definition.
>
> By separating the abstract constraint system from the concrete circuit, the programmer can instead locally add only the copy constraints that they know are needed. They are not forced to add artificial copy constraints corresponding to rotations. Although we still use numbered rows for convenience in the model, it would be sufficient to use any set with $n$ unique, not necessarily ordered elements. The abstract $\rightarrow$ concrete compilation can then reorder the rows as needed without changing the meaning of the circuit. This allows layout optimizations that would be impractical or error-prone to do manually, because they rely on global rather than local knowledge (such as which neighbouring cells are used or free, which can depend on gates that are logically unrelated).

## Compiling to a concrete circuit

TODO: define variables.

Below we will use the convention that variables marked with a prime ($'$) refer to *concrete* column or row indices.

We say that an advice cell with coordinates $(i, j)$ is "used" if it appears in some nontrivial copy constraint or custom constraint, i.e. $\mathsf{used}(i, j)$ is true iff any of the following hold:
\begin{array}{rcl}
\exists (k, \ell) \neq (i, j) &:& (i, j) \equiv_A (k, \ell) \\
\exists k, \ell &:& (i, j, k, \ell) \in S_I \\
\exists k, \ell &:& (i, j, k, \ell) \in S_F \\
\exists u &:& j \in \mathsf{CUS}_u \text{ and the } w[i, \_] \text{ argument to } p_u \text{ is used nontrivially}.
\end{array}

$w$ represents $m_A$ _abstract_ columns, that do not directly correspond 1:1 to actual columns in the compiled circuit (but may do so if the backend chooses).

Rotations are represented by hints $\mathbf{h} : [0,m_A) \mapsto [0,m_A') \times [0,n)$ where $m_A'$ is the number of concrete columns. To simplify the programming model, the hints are not supposed to affect the meaning of a circuit (i.e. the set of public inputs for which it is satisfiable, and the knowledge required to find a witness).

For convenience define:

$$
j \;\triangledown\; e = (j + e) \bmod n \\
\mathbf{H}(i, j') = (i', j' \;\triangledown\; e_i) \text{ where }\mathbf{h}(i) = (i', e_i)
$$

Tesselation between custom constraints is just represented by equivalence under $\equiv_A$. When the rotation hints indicate that two cells in the same concrete column are equivalent, the backend can optimise overall circuit area by reordering the rows so that the equivalent cells overlap.

More specifically, to compile the abstract circuit to a concrete circuit using the hints $\mathbf{h}$, we construct an injective mapping of abstract row numbers to concrete row numbers (before rotation), $\mathbf{r} : [0, n) \mapsto [0, n')$ with $n' \geq n$, such that the abstract cell with coordinates $(i, j)$ maps to the concrete cell with coordinates $\mathbf{H}(i, \mathbf{r}(j)) = (i', \mathbf{r}(j) \;\triangledown\; e_i)$ where $\mathbf{h}(i) = (i', e_i)$.

In order for this compilation to be correct, we must choose $\mathbf{r}$ so that every *used* abstract cell is represented by a distinct concrete cell, except that abstract cells that are equivalent under $\equiv_A$ *may* be identified. That is, $\mathbf{r}$ must be chosen such that:
$$
\forall (i, j, k, \ell) \in [0, m_A) \times [0, n) \times [0, m_A) \times [0, n) : \\
\mathsf{used}(i, j) \;\wedge\; \mathsf{used}(k, \ell) \;\wedge\; (i, j) \not\equiv_A (k, \ell) \Rightarrow \mathbf{H}(i, \mathbf{r}(j)) \neq \mathbf{H}(k, \mathbf{r}(\ell))
$$

> In other words, no two *used* advice cells map to the same concrete advice cell unless they are equivalent under $\equiv_A$. It is alright if one or more *unused* abstract cells map to the same concrete cell as a used abstract cell, because that will not affect the meaning of the circuit. Notice that specifying $\equiv_A$ as an equivalence relation helps to simplify this definition (as compared to specifying it as a set of copy constraints), because an equivalence relation is by definition symmetric, reflexive, and transitive.

Since correctness does not depend on the specific hints provided by the circuit programmer, it is also valid to use any subset of the provided hints, or to infer hints that were not provided.

The used abstract advice cells $w : \mathbb{F}^{m_A \times n}$ are translated to concrete advice cells $w' : \mathbb{F}^{m'_A \times n'}$:
$$
\mathsf{used}(i, j) \Rightarrow w'(\mathbf{H}(i, \mathbf{r}(j))) = w(i, j)
$$

The values of concrete cells not corresponding to any used abstract cell are arbitrary.

The instance columns are zero-extended to $n'$ rows.

The abstract fixed cells $f : \mathbb{F}^{m_F \times n}$ are translated to concrete fixed cells $f' : \mathbb{F}^{m_F \times n'}$ using just the row mapping, filling the fixed cells of added rows with zeros:
$$
f'(i, \mathbf{r}(j)) = f(i, j) \\
f'(i, j') = 0\text{ if } j' \not\in Im(\mathbf{r})
$$

The abstract coordinates appearing in $\equiv_A$, $S_I$, and $S_F$ are similarly mapped to their concrete coordinates.

The conditions for custom gates in the concrete circuit are then given by:
$$
\mathsf{CUS}'_u = \{ \mathbf{r}(j) : j \in \mathsf{CUS}_u \} \\
j' \in \mathsf{CUS}'_u \Rightarrow p_u\left( \begin{array}{rcl}
f'[0, j'], &\ldots,& f'[m_F-1, j'], \\
w'[\mathbf{H}(0, j')], &\ldots,& w'[\mathbf{H}(m_A-1, j')]
\end{array}
\right) = 0
$$

### Greedy algorithm for choosing $\mathbf{r}$

There is a greedy algorithm for deterministically choosing $\mathbf{r}$ that maintains ordering of rows (i.e. $\mathbf{r}$ is strictly increasing), and simply inserts a gap in the row mapping whenever the above constraint would not be met for all rows so far:

For $t \in [0, n)$, define
\begin{array}{rcl}
\mathsf{ok\_up\_to}(t, \mathbf{r}) &=& \forall (i, j, k, l) \in [0, m_A) \times [0, t] \times [0, m_A) \times [0, t] : \\
&&\mathsf{used}(i, j) \;\wedge\; \mathsf{used}(k, \ell) \;\wedge\; (i, j) \not\equiv_A (k, \ell) \Rightarrow \mathbf{H}(i, \mathbf{r}(j)) \neq \mathbf{H}(k, \mathbf{r}(\ell))
\end{array}

That is, $\mathsf{ok\_up\_to}(t, \mathbf{r})$ means that the correctness criterion above holds for the subset $[0, t]$ of abstract rows.

> set $\mathbf{r} := \{\}$
> set $v := 0$
> for $t$ from $0$ up to $n-1$:
> $\hspace{2em}$ find the minimal $u \geq v$ such that $\mathsf{ok\_up\_to}(t, \mathbf{r} \cup \{t \mapsto u\})$
> $\hspace{2em}$ set $\mathbf{r} := \mathbf{r} \cup \{t \mapsto u\}$ and $v := u+1$.
> let $n' = v$ be the number of concrete rows.

This algorithm can be proven correct by induction on $n$. It is complete because for each step it is always possible to find a suitable $u$. That is, we can always choose $u$ large enough that any additional conditions $$
\mathsf{used}(i, j) \;\wedge\; \mathsf{used}(k, \ell) \;\wedge\; (i, j) \not\equiv_A (k, \ell) \Rightarrow \mathbf{H}(i, \mathbf{r}(j)) \neq \mathbf{H}(k, \mathbf{r}(\ell))
$$ for $j = t$ or $\ell = t$ are met, because $\mathbf{r}(t) = u$ and both $\mathbf{H}(i, u)$ and $\mathbf{H}(k, u)$ can be made not to overlap with any previously used cell.
