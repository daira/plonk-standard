# Specification of the Plonkish Relation

## Objectives
- Need to agree on the API between “zkInterface for Plonkish” and the proving system.  Specify a general statement that the proving system has to implement.
- Section 2 of [https://eprint.iacr.org/2022/777.pdf](https://eprint.iacr.org/2022/777.pdf) : describes high level API of zk Interface for Plonkish statements

This is intended to be read in conjunction with the [Plonkish backend optimizations](https://github.com/daira/plonk-standard/blob/main/optimizations.md) document, which describes how to compile the abstract constraint system described here into a concrete circuit.

## Dependencies

Plonkish arithmetisation depends on a scalar field over a prime modulus $p$. We represent this field as the object $\mathbb{F}$. We denote the additive identity by $0$ and the multiplicative identity by $1$. Integers, taken modulo the field modulus $p$, are called scalars; arithmetic operations on scalars are implicitly performed modulo p. We denote the sum, difference, and product of two scalars using the +, -, and * operators, respectively. 

## The Plonkish Relation
The relation $\mathcal{R}\_{\mathsf{plonkish}}$ contains pairs of public instances $\mathsf{instance}$ and private advice $w$.  We say that $\mathsf{instance}$ is a valid instance whenever their exists some advice $w$ such that $(\mathsf{instance}, w) \in \mathcal{R}\_{\mathsf{plonkish}}$.  The Plonkish language $\mathcal{L}\_{\mathsf{plonkish}}$ contains all valid instances.

$$\mathcal{R}\_{\mathsf{plonkish}} =
\left\\{ \begin{array}{cc | c}
    (\mathbb{F}, (f,  \phi), \equiv\_A, S\_I, \{\mathsf{CUS}\_{u}, p\_u\}\_u, \{ \mathsf{LOOK}\_v, \mathsf{TAB}\_v, q\_v \}), & &  \\
   w \in \mathbb{F}^{m\_A \times n}, f \in \mathbb{F}^{m\_F \times n}, \phi \in \mathbb{F}^{m\_I \times n} & & \\
   \equiv\_A \subset ([0,m\_A) \times [0,n)) \times ([0,m\_A) \times [0,n)) & & (i,j) \equiv\_A (k,\ell) \Rightarrow w[i, j] = w[k, \ell] \\
   S\_I \subset [0,m\_A) \times [0,n) \times [0,m\_I) \times [0,n) & & (i,j,k,\ell) \in S\_I \Rightarrow w[i, j] = \phi[k, \ell] \\
   S\_F \subset [0,m\_A) \times [0,n) \times [0,m\_F) \times [0,n) & & (i,j,k,\ell) \in S\_F \Rightarrow w[i, j] = f[k, \ell] \\
   \forall u, \ \mathsf{CUS}\_u \subset [0,n), \ p\_u: \mathbb{F}^{m\_F + m\_A} \mapsto \mathbb{F} & &  j \in \mathsf{CUS}\_u \Rightarrow p\_u( f[0, j], \ldots ,  f[m\_F-1, j], w[0, j], \ldots, w[m\_A-1,j] ) = 0 \\
   \mathsf{LOOK}\_v \subset  [0,n),  \mathsf{TAB}\_v \subset \mathbb{F}^{m\_F}  & & j \in \mathsf{LOOK}\_v \Rightarrow q\_v(w[0, j], \ldots, w[m-1, j] ) \in \mathsf{TAB}\_k \\ 
\end{array} \right\\}$$

The relation $\mathcal{R}\_{\mathsf{plonkish}}$ takes as public inputs the instances 
$$\mathsf{instance} = (\mathbb{F}, (f,  \phi), S\_A, S\_I, \{\mathsf{CUS}\_{k}, p\_k\}\_k, \{ \mathsf{LOOK}\_k, \mathsf{TAB}\_k \})$$  of the following form.

### Inputs into $\mathcal{R}\_{\mathsf{plonkish}}$ 

| Public Inputs | Description | 
| -------- | -------- | 
| $\mathbb{F}$ | A prime field. |
| $m\_A > 0$ | Number of advice columns. |
| $m\_I$ | Number of instance columns. |
| $m\_F$ | Number of fixed columns. |
| $n > 0$ | Number of rows. |
| $f$ | Fixed columns $f : \mathbb{F}^{m\_F \times n}$. |
| $\phi$ | Instance columns $\phi :  \mathbb{F}^{m\_I \times n}$. |
| $\equiv\_A$ | An equivalence relation on $[0,m\_A) \times [0,n)$ indicating which advice entries are equal. | 
| $S\_I$ | A set $S\_I \subseteq [0,m\_A) \times [0,n) \times [0,m\_I) \times [0,n)$ indicating which instance entries must be used in the advice. | 
| $S\_F$ | A set $S\_F \subseteq [0,m\_A) \times [0,n) \times [0,m\_F) \times [0,n)$ indicating which fixed entries must be used in the advice. |
| $\mathsf{CUS}\_u$ | Sets $\mathsf{CUS}\_u \subseteq [0,n)$ indicating which rows the custom functions $p\_u: \mathbb{F}^{m\_F + m\_A} \mapsto \mathbb{F}$ are applied to.     |
| $p\_u$ | Custom multivariate polynomials $p\_u: \mathbb{F}^{m\_F + m\_A}  \mapsto \mathbb{F}$ | 
| $\mathsf{LOOK}\_v$ | Sets $\mathsf{LOOK}\_v \subseteq [0,n)$ indicating which advice rows are contained in the lookup tables $\mathsf{TAB}\_v$. | 
| $\mathsf{TAB}\_v$ | Lookup tables $\mathsf{TAB}\_v\subseteq \mathbb{F}^{m\_F}$ with an unbounded number of entries in the field $\mathbb{F}^m$ |
| $q\_v$ | Custom scaling functions $q\_u: \mathbb{F}^{m\_A}  \mapsto \mathbb{F}^{m\_{q\_u}}$ where $q\_u \leq m\_A$ | 

> TODO: do we need to generalise lookup tables to support dynamic tables (in advice columns)? Probably too early, but we could think about it.

| Private Inputs | Description | 
| -------- | -------- | 
| $w$ | Advice columns $w : \mathbb{F}^{m\_A \times n}$.    | 

### Conditions satisfied by statements in $\mathcal{R}\_{\mathsf{plonkish}}$

There are three types of constraints that a Plonkish statement $(\mathsf{instance}, w) \in \mathcal{R}\_{\mathsf{Plonkish}}$ must satisfy:

* Copy constraints
* Custom constraints
* Lookup constraints

#### Copy constraints

Copy constraints that enforce that advice entries must be equal to other inputs.  Plonkish allows custom constraints between the instance, fixed, and advice constraint entries.

| Copy Constraints | Description | 
| -------- | -------- | 
| $(i,j) \equiv\_A (k,\ell) \Rightarrow w[i, j] = w[k, \ell]$ | $\equiv\_A$ is an equivalence relation indicating which advice entries are constrained to be equal. |
| $(i,j,k,\ell) \in S\_I \Rightarrow w[i, j] = \phi[k, \ell]$ | The $(k, \ell)$th instance entry is equal to the $(i,j)$th advice entry for all $(i,j,k,\ell) \in S\_I$.    |
| $(i,j,k,\ell) \in S\_F \Rightarrow w[i, j] = f[k, \ell]$ | The $(k, \ell)$th fixed entry is equal to the $(i,j)$th advice entry for all $(i,j,k,\ell) \in S\_F$.    |

#### Custom constraints

Custom constraints that enforce that fixed entries and advice entries satisfy some multivariate polynomial.  Here $p\_u$ could indicate a multiplication gate, an addition gate, or any other custom case that can be generated using a combination of multiplication gates and addition gates.

| Custom Constraints | Description |
| -------- | -------- | 
| $j \in \mathsf{CUS}\_u \Rightarrow p\_u( f[0, j], \ldots ,  f[m\_F-1, j], w[0, j], \ldots, w[m\_A-1, j] ) = 0$ | $u$ is the index of a custom constraint. $j$ ranges over the set of rows $\mathsf{CUS}\_u$ for which the custom constraint is switched on. |

Here $p\_u: \mathbb{F}^{m\_F + m\_A} \mapsto \mathbb{F}$ is a function such that $$p\_u( X\_0, \ldots, X\_{m\_F + m\_A - 1}) = p\_{u,0} * X\_0 \ \ * / + \ \ p\_{u,1} * X\_1 \ \ * / + \ \ \cdots \ \ * / + \ \ p\_{u,m\_F + m\_A - 1} X\_{m\_F+m\_A-1}$$ where $p\_{u,i} \in \mathbb{F}$ are scalars and where $p\_u$ determines whether to use the multiplication $*$ or addition $+$ operation at each step.

#### Lookup constraints

Lookup constraints enforce that advice entries are contained in some table. 

Fixed lookup tables are determined in advance, whereas dynamic lookup tables are determined by the advice.  

| Lookup Constraints | Description |
| -------- | -------- | 
| $j \in \mathsf{LOOK}\_v \Rightarrow q\_u( w[0, j], \ldots, w[m\_A-1, j] ) \in \mathsf{TAB}\_v$ | $v$ is the index of a lookup table. $j$ ranges over the set of rows $\mathsf{LOOK}\_v$ for which the lookup constraint is switched on. |

Here $q\_v: \mathbb{F}^{ m\_A } \mapsto \mathbb{F}^{m\_{q\_u}}$ is a scaling function that behaves as follows.  Let $\vec{q}\_v \in \mathbb{F}^{ m\_A }$ be a vector of scalars.  Then 
$$q\_v( X\_0, \ldots, X\_{m\_A - 1}) = (q\_{i\_0} X\_{i\_0}, \ldots, q\_{i\_{m\_{q\_v}}} X\_{i\_{m\_{q\_v}}})$$
where $( i\_0, \ldots, i\_{m\_{q\_v}}) \subset (0, \ldots, m\_A)$ are all the indices such that $q\_{v, i\_j} \neq 0$.


----

(This change log is for both this document and https://hackmd.io/6RHNkB6\_TlaqHSDHfwGzuQ)

#### Changes made by Daira since the 2023-03-03 meeting

* Change $S\_A$ to an equivalence relation
  * This removes unnecessary degrees of freedom in specifying copy constraints between advice cells, and makes it easier to define correctness of the abstract $\rightarrow$ concrete compilation.
* Use $u$ instead of $k$ to index custom constraints, and $v$ instead of $k$ to index lookups.
  * This avoids overloading of $k$.
* Define the $\triangledown$ operator for addition modulo $n$.
* Define "used" cells.
* For "With rotations (option A)", change $E\_u$ to be a list of rotations.
  * This avoids the need for $z\_u$ as a separate variable.
* For "With rotations (option B)", give more detail and specify a correctness condition for compiling an abstract circuit to a concrete circuit using hints.


#### Changes made by Mary since 2023-15-03 

* Add rotation constraints into copy constraints
  * This is an alternative proposal to Option B of the hint suggestion.
  * If we use this approach we could keep the description for copy constraints as is. 

* Suggest enforcing that rotation constraints must be fully implied by $S\_A$
    * Avoids having to define "used" cells and therefore simplifies.
    * We can use the (under construction) greedy algorithm anyway.
    
* Tried to be more precise about what $p\_u$ constraints are valid.  
    * Before we just said $p\_u$ is constructed using addition and multiplication gates which is potentially too vague.
    * Current suggestion is a bit ugly and needs checking for correctness.


#### Changes made by Daira and Str4d on 2023-03-17

* Rename some variables for consistency, and use $\triangledown$ also in Mary's definition of rotation constraints.
* Complete the "Greedy algorithm for choosing $\mathbf{r}$".
* Daira: Delete the paragraph describing the $\mathsf{Rot}$ constraints as hints, and add a note explaining the disadvantages of that approach. 

#### Changes made by Mary on 2023-28-03 

* Added $q\_v$ scaling functions to describe input to lookup tables.
* Completed first attempt at describing lookup constraints
* Have not yet tackled dynamic tables.
