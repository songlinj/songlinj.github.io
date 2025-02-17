# Lean Lessons that May or May Not Carry Over to Coq

## Automation

In Lean proofs, the main form of automation goes with the `simp` tactic. It is not to be confused with the `simpl` tactic in Coq, or any other existing tactics in Coq. Actually, it could serve many different purposes:

- `autorewrite with ...`: you may register a rewriting lemma using the annotation `@[simp]`, and then `simp` will try to rewrite using these lemmas whenever it can. Like hint databases in Coq, you may register additional simplification attributes, and using them as `simp [myattrs...]`. Customized discharger for premises could be provided by supplying `(disch := ...)`.
- `repeat setoid_rewrite ...`: the rewriting performed by `simp` is actually not the familiar `rewrite` seen in Coq, but `setoid_rewrite`, which rewrites under the binders as well.
- `simpl`: by adding reduction lemmas of functions into the `simp` database, `simp` can be used to simulate the evaluation, as done using `simpl` in Coq. Still, you get a chance to prevent unwanted oversimplifications that make the terms too verbose, by defining reduction only for the _good_ forms. You could set `Opaque` to achieve similar results in Coq.
- `auto`: a special form `simp [*]` allows using local hypotheses as simplification facts. You may also provide names to specific facts instead of `*`
- Recursive pattern matching: `simp` is by default recursive. The exactly same `simp` call will be applied towards all subgoals generated from a success rewriting step, including the premises of the rewriting and the result of the rewriting. Importantly, this recursive procedure (1) is protected with a fuel, (2) refuses to instantiate any existential variables, and (3) will not succeed unless all premises are proven true -- which prevents you from failing the usual ways in Coq.
- `intuition`. Although `simp` itself is not a decision procedure, you can easily simplify a logical formulae using a list of local facts. Along with other absorbing logical facts (e.g., true or, false and), it could take significantly shorter time to solve a goal than `tauto`, if that ever succeeds.

Besides, `simp` comes with good tracing support, so:

- You can use tracing to diagnose why desired rewriting is not performed.
- You can use tracing to diagnose why rewriting goes into the wrong direction, and easily disable the unwanted rewriting fact using `simp [-bad_fact]`.
- You can generate a replacement `simp only [...]` call which doesn't require an extensive search, by simply writing `simp?`.

In addition, the way `simp` is designed encourages users to write lemmas of equivalences instead of implications, which could lead to more general lemmas overall.

## Set / Qualifier Encoding

First thing first: what kind of sets do we need?

The base of our sets should ideally be _PropSets_, which is (1) our `pl` (2) Lean's `Set` and (3) stdpp's `PropSet`. Statements over such sets could be easily solved using logical solvers or simple tactics, and can be used to encode some operations without an obvious computing solution (e.g. transitive closures without telescope). Still, they don't work well in surface languages, where sets should better be decidable.

Our `ql` set is encoded as compositions over Boolean functions, thus being decidable, minus one operation: subset. For subset relations to be decidable, the sort of sets should support enumerating all elements within an instance, thus being finite. Our `ql` can easily encode an infinite set, and there's no algorithm to test the subset relations between infinite sets. As a result, we have `psub`, but not `qsub`, unless we rely on an external domain bound.

Finite sets are, however, not so simple to implement. Basically, they have to provide both membership testing and domain enumeration, and there is some invariant to maintain for every operation, so that the two stay consistent. For instance, Lean's [Finset](https://leanprover-community.github.io/mathlib4_docs/Mathlib/Data/Finset/Defs.html#Finset) is defined as a List accompanied with a witness that there is no duplication. Still, they should be usable, as long as they can be easily lifted to propsets.

In terms of lifting, actually two things could be done. Referring to Lean's mathlib, where `↑s` is similar to `plift s` in our mechanizations:
```lean
@[simp]
theorem Finset.coe_union {α} [DecidableEq α] (s₁ s₂ : Finset α) :
	↑(s₁ ∪ s₂) = ↑s₁ ∪ ↑s₂
@[simp]
theorem Finset.mem_union {α} [DecidableEq α] {s t : Finset α} {a : α} :
	a ∈ s ∪ t ↔ a ∈ s ∨ a ∈ t
```

So, `.coe_union` is our `plift_or`. However, we don't actually need it, as long as we are not mixing finite sets with sets, e.g. `↑s_fin ∪ s_inf`. If we are only discharging a membership testing on finite sets, `.mem_union` suffices, as it also provides us with a disjunction of two primitive clauses. Both lemmas are registered as `@[simp]`, so that either lifting happens implicitly.

So, in my Lean mechanization, I have the following definitions, where the finite `ql` sets are used as surface qualifiers across `sem_type` and `has_type`, and unbounded `pl` sets are used to denote locations in logical relations. Specifically, `pl` enjoys the complementing operation and the inductive definition of transitive closure `locs_locs_stty`, while `vars_trans` on `ql` now has a decidable definition by recursing on the context.
```lean
abbrev ql := Finset id
abbrev pl := Set Nat
```

For Coq: [stdpp.sets](https://plv.mpi-sws.org/coqdoc/stdpp/stdpp.sets.html#lab118) defines a `set_unfold` tactic, which claims to be breaking up a membership testing into a logical formulae. Interestingly, it is defined using type class applications.

-----------------

The second question: when we are adding locally nameless, there's going to be both bound variables and free variables. Should they reside in two separate sets, or a single set over two cases?

Our current Coq mechanizations stick to the first path, using separate sets. For the syntactic proof, there's `nats` (boolean), `Nats` (prop), and then `qual` as a product of `nats`, and `Qual` as a product of `Nats`. In my initial LR version, I made `ql2` which is a pair of `ql`s, while `pl` is still single. This `ql2` is actually not so bad -- as I don't even have a `pl2` lifting, I always work on the free variable set part (`qfvs`), which is still a `ql`. The down side is that this version is not so _faithful_ compared to the text, as, for example, `qfvs q1 ⊆ qfvs q2` does not really imply `q1 ⊆ q2` in the text.

Beginning my Lean experience, I repeated this _sets of sets_ pattern, but wanted to add some native operations (subset/union/inter/sdiff/...) for `ql2`, so that we can use a full set rather than only a set component in the surface language. This is similar to what we have in the syntactic proofs, and likewise, this _sets of sets_ duplication in Lean also became quite unpleasant.

So first of all: by adding a layer of abstraction, we have to write another layer of lemmas. An instance is the `.mem_union` lemma discussed above. While it is already there in mathlib, we have to define another lifting lemma for our joint sets. And repeat this for all other set operations. This is not too bad -- just a constant factor.

A worse issue is that whenever you are doing simplification/unfolding/rewriting, you've got a choice to make. Considering a simple `s ⊆ t`, now there are two ways to interpret it:
1. Reveal the membership predicate first: `∀ x ∈ s, x ∈ t`
2. Branching on the components first: `qbvs s ⊆ qbvs t ∧ qfvs s ⊆ qfvs t`

The first option could easily drive membership lifting as discussed above, while the second  works more easily with closedness, which does have a simple component-first interpretation:
```lean
def closed_ql bv fv q: Prop :=
	qbvs q ⊆ range bv ∧ qfvs q ⊆ range fv
```

What makes things worse is that these two options do preclude each other, whereas reasoning involving both subset and closedness is pretty common. Thus, to do a reasoning, we must lift everything up to a logical formulae of membership testings on individual component sets, which is unnecessarily verbose for most cases, and often too pathological for solvers as well.

To make things a little nicer, we could go back and revise the definition of closedness, so that it doesn't break components up:
```lean
def closed_ql bv fv q: Prop := q ⊆ ⟨range bv, range fv⟩
```

But then the point is: If we are always hiding components anyways, we don't have to define them in the first place, at the cost of writing a full set of new lifting lemmas. So, I eventually went back to a flat set encoding, and here's what I have:
```lean
inductive id: Type
| fresh
| bv (n: Nat)
| fv (n: Nat)
deriving DecidableEq

notation "✦" => id.fresh
prefix:max "%" => id.fv  -- $ has been reserved by Lean Macros
prefix:max "#" => id.bv

abbrev ql := Finset id
```

------------

Another thing I've been trying to do is to have bv/fv-neutral definitions and lemmas. Now that they do not make a difference towards sets, it would be great if they share other definitions as well.

For example, the `no_occur` predicate (similar to `not_free` in syntactic proofs) I have goes:
```lean
def no_occur (T: ty) (x: id): Prop :=
  match T with
  | .TRef T q =>
    no_occur T x ∧ x+1 ∉ q
  | .TFun T1 q1 T2 q2 =>
    no_occur T1 (x+1) ∧ x+1 ∉ q1 ∧ no_occur T2 (x+2) ∧ x+2 ∉ q2
  | .TUnit | .TBool => True
```

The key to making this neutral to bv/fv is those `x+n` notations, which only changes the index of `x` if it is a bound variables, and is a noop otherwise. In practice, this rarely causes any added burden, as `x` is often a concrete bv/fv, and thus simplifications apply.

A more interesting example is opening, which converts a bound variable to a free variable, and is the main added complication from locally nameless. For qualifiers, the definition goes:
```lean
-- [x ↦ xs] q
def ql.substq x xs q := q \ {x} ∪ ite (x ∈ q) xs ∅
```

This operation is actually called `substq` instead of opening, as we don't really limit it to be _from bound to free_. Then there's a few interesting things we can express, without writing too many lemmas:

- Split a set opening into substitution over simple opening: `[#0 ↦ qf] [#1 ↦ qx] q = [%‖G‖ ↦ qf] [%(‖G‖+1) ↦ qx] [#0 ↦ %‖G‖] [#1 ↦ %(‖G‖+1)] q`
- Absorb opening and unopening: `[%‖G‖ ↦ #0] [#0 ↦ %‖G‖] q = q`

One more thing: At times we have to reason about the `no_occur` predicate on a type with opening/substitution applied. We have a _single_ lemma for that:
```lean
@[simp]
lemma nooccur_substq:
  no_occur ([n ↦ q] T) x ↔ (x ≠ n → no_occur T x) ∧ (x ∈ q → no_occur T n) 
```

You may count how many lemmas we could have written if we had separate versions of definitions for bv/fv, and if we had separated all those miss/hit conditions. Moreover, as many times `x, n, q` are all known constants, a goal like `no_occur ([n1 ↦ q1] [n2 ↦ q2] T) x` could be solved using a single `simp` tactic.

## Functional Rewriting / Induction

I've been boasting Lean for ease in dealing with well-founded recursion, and the secret sauce there is definitely [functional induction](https://lean-lang.org/blog/2024-5-17-functional-induction/). In Coq, [the equations plugin](https://coq.inria.fr/platform-docs/Tutorial_Equations_basics.html) and [the Function vernac](https://coq.inria.fr/doc/v8.17/refman/using/libraries/funind.html) could achieve a similar job. There are two main things that make this happen:

1. Generate a rewriting lemma for each of the function cases.
2. Generate an induction principle according to the function structure.

When you reason about a function call, you (1) apply the induction principle to get the cases split, and (2) use rewriting lemmas to reveal the function definition for each of the cases.

Rewriting lemmas are actually essential to well-founded recursions, as such fixpoint definitions could not be easily unfolded. My prior experience with `Program Fixpoint` in Coq is that we'd better have some rewriting lemmas, and that compiler's generating such lemmas definitely saves some effort. Interestingly, Lean generates such lemmas under the names `your_fixpoint.eq_<n>`, which is automatically involved when you call `simp [your_fixpoint]`, making it the same experience to use a WF fixpoint as a structural fixpoint.

A new induction principle -- one may question if that is really necessary. Well indeed, it would not be super interesting if your fixpoint is a fully structural one, but there are a few cases it can get helpful:

- Dealing with the measure in a WF fixpoint. This usually involves adding a generalizing condition for the induction hypothesis and induction on the decreasing measure instead of the matching variable.
- Dealing with mutual recursions. We have to provide the induction invariant for each of the mutual functions (usually in disjunctive cases), and there the generated induction principle can help to select the correct IH.
- Dealing with wildcard cases. This is useful even in structural fixpoints. If your function has fewer cases than your datatype, by doing induction on the data, chances are that you will end up with too many, exactly the same cases. A generated induction principle without splitting those redundant cases can help you avoid the duplication.

On the other hand, such generation mechanisms can be fragile given their nontrivial nature. A few instances:

- In the `TFun T1 T2` case of the induction principle generated by Lean for `val_type`, there's is no IH for `T1`, since the recursive call occurs in the contravariant position.
- In Coq, `Function` refuses to compile `val_type` at all, as none of the recursive calls occur as _the end of the branch_. Actually, they occur either at the contravariant position or as part of an inner function scope (inside `exists...`)

In such cases, we may still find it helpful to write our own induction principles, so that we can decouple the induction structure from other interesting parts in the proof. It is also worth noting that the `induction` tactic has made it easy to change the induction principle (by `using ...`) and tweak generalization (by `generalizing ...`).
