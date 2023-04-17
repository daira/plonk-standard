# Specification of the Plonkish Relation

## Objectives
- Need to agree on the API between “zkInterface for Plonkish” and the proving system.  Specify a general statement that the proving system has to implement.
- Section 2 of [https://eprint.iacr.org/2022/777.pdf](https://eprint.iacr.org/2022/777.pdf) : describes high level API of zk Interface for Plonkish statements

This is intended to be read in conjunction with the [Plonkish backend optimizations](https://github.com/daira/plonk-standard/blob/main/optimizations.md) document, which describes how to compile the abstract constraint system described here into a concrete circuit.

## Dependencies

Plonkish arithmetisation depends on a scalar field over a prime modulus $p$. We represent this field as the object $\mathbb{F}$. We denote the additive identity by $0$ and the multiplicative identity by $1$. Integers, taken modulo the field modulus $p$, are called scalars; arithmetic operations on scalars are implicitly performed modulo p. We denote the sum, difference, and product of two scalars using the +, -, and * operators, respectively. 

## The Plonkish Relation
The relation $\mathcal{R}_{\mathsf{plonkish}}$ contains pairs of public instances $\mathsf{instance}$ and private advice $w$.  We say that $\mathsf{instance}$ is a valid instance whenever their exists some advice $w$ such that $(\mathsf{instance}, w) \in \mathcal{R}_{\mathsf{plonkish}}$.  The Plonkish language $\mathcal{L}_{\mathsf{plonkish}}$ contains all valid instances.

$$
\mathcal{R}_ {\mathsf{plonkish}} =
\left\{ \begin{array}{cc | c}
    (\mathbb{F}, (f, \phi), \equiv_ A, S_I, \{\mathsf{CUS}_ {u}, p_u\}_ u, \{ \mathsf{LOOK}_ v, \mathsf{TAB}_ v, q_ v \}), & &  \\
   w \in \mathbb{F}^{m_ A \times n}, f \in \mathbb{F}^{m_ F \times n}, \phi \in \mathbb{F}^{m_ I \times n} & & \\
   \equiv_ A \subset ([0,m_ A) \times [0,n)) \times ([0,m_ A) \times [0,n)) & & (i,j) \equiv_ A (k,\ell) \Rightarrow w[i, j] = w[k, \ell] \\
   S_ I \subset [0,m_ A) \times [0,n) \times [0,m_ I) \times [0,n) & & (i,j,k,\ell) \in S_ I \Rightarrow w[i, j] = \phi[k, \ell] \\
   S_ F \subset [0,m_ A) \times [0,n) \times [0,m_ F) \times [0,n) & & (i,j,k,\ell) \in S_ F \Rightarrow w[i, j] = f[k, \ell] \\
   \forall u, \ \mathsf{CUS}_ u \subset [0,n), \ p_ u: \mathbb{F}^{m_ F + m_ A} \mapsto \mathbb{F} & & j \in \mathsf{CUS}_ u \Rightarrow p_ u( f[0, j], \ldots, f[m_ F - 1, j], w[0, j], \ldots, w[m_ A - 1,j] ) = 0 \\
   \mathsf{LOOK}_ v \subset  [0,n),  \mathsf{TAB}_ v \subset \mathbb{F}^{m_ F}  & & j \in \mathsf{LOOK}_ v \Rightarrow q_v(w[0, j], \ldots, w[m-1, j] ) \in \mathsf{TAB}_ k \\ 
\end{array} \right\}
$$

The relation $\mathcal{R}_ {\mathsf{plonkish}}$ takes as public inputs the instances 
$$ \mathsf{instance} = (\mathbb{F}, (f, \phi), S_ A, S_ I, \{\mathsf{CUS}_ {k}, p_ k\}_k, \{ \mathsf{LOOK}_ k, \mathsf{TAB}_ k \})$$  of the following form.

### Inputs into $\mathcal{R}_ {\mathsf{plonkish}}$ 

| Public Inputs | Description | 
| -------- | -------- | 
| $\mathbb{F}$ | A prime field. |
| $m_A > 0$ | Number of advice columns. |
| $m_ I$ | Number of instance columns. |
| $m_ F$ | Number of fixed columns. |
| $n > 0$ | Number of rows. |
| $f$ | Fixed columns $f : \mathbb{F}^{m_ F \times n}$. |
| $\phi$ | Instance columns $\phi :  \mathbb{F}^{m_I \times n}$. |
| $\equiv_ A$ | An equivalence relation on $[0,m_ A) \times [0,n)$ indicating which advice entries are equal. | 
| $S_ I$ | A set $S_ I \subseteq [0,m_ A) \times [0,n) \times [0,m_ I) \times [0,n)$ indicating which instance entries must be used in the advice. | 
| $S_ F$ | A set $S_ F \subseteq [0,m_ A) \times [0,n) \times [0,m_ F) \times [0,n)$ indicating which fixed entries must be used in the advice. |
| $\mathsf{CUS}_ u$ | Sets $\mathsf{CUS}_ u \subseteq [0,n)$ indicating which rows the custom functions $p_ u: \mathbb{F}^{m_ F + m_ A} \mapsto \mathbb{F}$ are applied to.     |
| $p_ u$ | Custom multivariate polynomials $p_ u: \mathbb{F}^{m_ F + m_ A}  \mapsto \mathbb{F}$ | 
| $\mathsf{LOOK}_ v$ | Sets $\mathsf{LOOK}_ v \subseteq [0,n)$ indicating which advice rows are contained in the lookup tables $\mathsf{TAB}_ v$. | 
| $\mathsf{TAB}_ v$ | Lookup tables $\mathsf{TAB}_ v\subseteq \mathbb{F}^{m_ F}$ with an unbounded number of entries in the field $\mathbb{F}^m$ |
| $q_ v$ | Custom scaling functions $q_ v: \mathbb{F}^{m_ A}  \mapsto \mathbb{F}^{m_ {q_ v}}$ where $q_ v \leq m_ A$ | 

> TODO: do we need to generalise lookup tables to support dynamic tables (in advice columns)? Probably too early, but we could think about it.

| Private Inputs | Description | 
| -------- | -------- | 
| $w$ | Advice columns $w : \mathbb{F}^{m_ A \times n}$.    | 

### Conditions satisfied by statements in $\mathcal{R}_ {\mathsf{plonkish}}$

There are three types of constraints that a Plonkish statement $(\mathsf{instance}, w) \in \mathcal{R}_ {\mathsf{Plonkish}}$ must satisfy:

* Copy constraints
* Custom constraints
* Lookup constraints

#### Copy constraints

Copy constraints that enforce that advice entries must be equal to other inputs.  Plonkish allows custom constraints between the instance, fixed, and advice constraint entries.

| Copy Constraints | Description | 
| -------- | -------- | 
| $(i,j) \equiv_ A (k,\ell) \Rightarrow w[i, j] = w[k, \ell]$ | $\equiv_ A$ is an equivalence relation indicating which advice entries are constrained to be equal. |
| $(i,j,k,\ell) \in S_ I \Rightarrow w[i, j] = \phi[k, \ell]$ | The $(k, \ell)$th instance entry is equal to the $(i,j)$th advice entry for all $(i,j,k,\ell) \in S_ I$.    |
| $(i,j,k,\ell) \in S_ F \Rightarrow w[i, j] = f[k, \ell]$ | The $(k, \ell)$th fixed entry is equal to the $(i,j)$th advice entry for all $(i,j,k,\ell) \in S_ F$.    |

#### Custom constraints

Custom constraints that enforce that fixed entries and advice entries satisfy some multivariate polynomial.  Here $p_u$ could indicate a multiplication gate, an addition gate, or any other custom case that can be generated using a combination of multiplication gates and addition gates.

| Custom Constraints | Description |
| -------- | -------- | 
| $j \in \mathsf{CUS}_ u \Rightarrow p_ u( f[0, j], \ldots ,  f[m_ F-1, j], w[0, j], \ldots, w[m_ A - 1, j] ) = 0$ | $u$ is the index of a custom constraint. $j$ ranges over the set of rows $\mathsf{CUS}_ u$ for which the custom constraint is switched on. |

Here $p_ u: \mathbb{F}^{m_ F + m_ A} \mapsto \mathbb{F}$ is a function such that $$ p_ u( X_ 0, \ldots, X_ {m_ F + m_ A - 1}) = p_ {u,0} * X_ 0 \ \ * / + \ \ p_ {u,1} * X_1 \ \ * / + \ \ \cdots \ \ * / + \ \ p_ {u,m_ F + m_ A - 1} X_{m_ F + m_ A - 1}$$ where $p_ {u,i} \in \mathbb{F}$ are scalars and where $p_ u$ determines whether to use the multiplication $*$ or addition $+$ operation at each step.

#### Lookup constraints

Lookup constraints enforce that advice entries are contained in some table. 

Fixed lookup tables are determined in advance, whereas dynamic lookup tables are determined by the advice.  

| Lookup Constraints | Description |
| -------- | -------- | 
| $j \in \mathsf{LOOK}_ v \Rightarrow q_ v( w[0, j], \ldots, w[m_ A - 1, j] ) \in \mathsf{TAB}_ v$ | $v$ is the index of a lookup table. $j$ ranges over the set of rows $\mathsf{LOOK}_ v$ for which the lookup constraint is switched on. |

Here $q_ v: \mathbb{F}^{ m_ A } \mapsto \mathbb{F}^{m_ {q_ v}}$ is a scaling function that behaves as follows.  Let $\vec{q}_ v \in \mathbb{F}^{ m_ A }$ be a vector of scalars.  Then 
$$q_ v( X_ 0, \ldots, X_ {m_ A - 1}) = (q_ {i_ 0} X_ {i_ 0}, \ldots, q_ {i_ {m_ {q_ v}}} X_ {i_ {m_ {q_ v}}})$$
where $( i_ 0, \ldots, i_ {m_ {q_ v}}) \subset (0, \ldots, m_ A)$ are all the indices such that $q_ {v, i_ j} \neq 0$.

----
Here $q_ v: \mathbb{F}^{ m_ A } \mapsto \mathbb{F}^{ m_ A }$ is a scaling function that behaves as follows.  Let $\vec{q}_ v \in \mathbb{F}^{ m_ A }$ be a vector of scalars.  Then 
$$q_ v( X_ 0, \ldots, X_ {m_ A - 1}) = (q_ {v,0} X_ {i_ 0}, \ldots, q_ {v, m_ A -1} X_ {v, m_ A - 1})$$

----
Here $q_ {v,i}: \mathbb{F}^{m_ F + m_ A} \mapsto \mathbb{F}$ is an evaluation function that maps the fixed and advice cells on the lookup row to a tuple of field elements that must match a row of the table. Then 
$$q_ {v,i}(X_ 0, \ldots, X_ {m_ F + m_ A - 1}) = \text{polynomial as for custom constraints}$$

----

(This change log is for both this document and https://github.com/daira/plonk-standard/blob/main/optimizations.md)

#### Changes made by Daira since the 2023-03-03 meeting

* Change $S_ A$ to an equivalence relation
  * This removes unnecessary degrees of freedom in specifying copy constraints between advice cells, and makes it easier to define correctness of the abstract $\rightarrow$ concrete compilation.
* Use $u$ instead of $k$ to index custom constraints, and $v$ instead of $k$ to index lookups.
  * This avoids overloading of $k$.
* Define the $\triangledown$ operator for addition modulo $n$.
* Define "used" cells.
* For "With rotations (option A)", change $E_u$ to be a list of rotations.
  * This avoids the need for $z_ u$ as a separate variable.
* For "With rotations (option B)", give more detail and specify a correctness condition for compiling an abstract circuit to a concrete circuit using hints.


#### Changes made by Mary since 2023-15-03 

* Add rotation constraints into copy constraints
  * This is an alternative proposal to Option B of the hint suggestion.
  * If we use this approach we could keep the description for copy constraints as is. 

* Suggest enforcing that rotation constraints must be fully implied by $S_A$
    * Avoids having to define "used" cells and therefore simplifies.
    * We can use the (under construction) greedy algorithm anyway.
    
* Tried to be more precise about what $p_ u$ constraints are valid.  
    * Before we just said $p_ u$ is constructed using addition and multiplication gates which is potentially too vague.
    * Current suggestion is a bit ugly and needs checking for correctness.


#### Changes made by Daira and Str4d on 2023-03-17

* Rename some variables for consistency, and use $\triangledown$ also in Mary's definition of rotation constraints.
* Complete the "Greedy algorithm for choosing $\mathbf{r}$".
* Daira: Delete the paragraph describing the $\mathsf{Rot}$ constraints as hints, and add a note explaining the disadvantages of that approach. 

#### Changes made by Mary on 2023-28-03 

* Added $q_ v$ scaling functions to describe input to lookup tables.
* Completed first attempt at describing lookup constraints
* Have not yet tackled dynamic tables.

#### Changes made by Daira on 2023-04-06

* ...