```agda
open import Cat.Prelude

module Cat.FinitelyComplete {ℓ ℓ'} (C : Precategory ℓ ℓ') where
```

<!--
```agda
open import Cat.Diagram.Equaliser C
open import Cat.Diagram.Terminal C
open import Cat.Diagram.Pullback C
open import Cat.Diagram.Product C
open import Cat.Reasoning C
```
-->

# Finitely Complete Categories

A category is said to be **finitely complete** if it admits limits for
every diagram with a finite shape. While this condition might sound very
strong, and thus that it would be hard to come by, it turns out we can
get away with only the following common shapes of limits:

* A [terminal object] (limit over the empty diagram)
* Binary [products] (limits over diagrams of the form $\bullet\quad\bullet$, that is, two points)
* Binary [equalisers] (limits over diagrams of the form $\bullet\rightrightarrows\bullet$)
* Binary [pullbacks] (limits over diagrams of the form $\bullet\to\bullet\ot\bullet$)

[terminal object]: Cat.Diagram.Terminal.html
[Products]: Cat.Diagram.Product.html
[Equalisers]: Cat.Diagram.Pullback.html
[Pullbacks]: Cat.Diagram.Equaliser.html

In reality, the list above has some redundancy. Since we can build
products out of pullbacks and a terminal object, and conversely we can
build pullbacks out of products and equalisers, either of the following
subsets suffices:

* A terminal object, binary products, binary equalisers;
* A terminal object and binary pullbacks.

```agda
record Finitely-complete : Type (ℓ ⊔ ℓ') where
  field
    terminal   : Terminal
    products   : ∀ A B → Product A B
    equalisers : ∀ {A B} (f g : Hom A B) → Equaliser f g
    pullbacks  : ∀ {A B C} (f : Hom A C) (g : Hom B C) → Pullback f g

  _⊗_ : Ob → Ob → Ob
  A ⊗ B = products A B .Product.apex

  Eq : ∀ {A B} (f g : Hom A B) → Ob
  Eq f g = equalisers f g .Equaliser.apex

  Pb : ∀ {A B C} (f : Hom A C) (g : Hom B C) → Ob
  Pb f g = pullbacks f g .Pullback.apex

open Finitely-complete
```

## With equalisers

We now prove that having products and equalisers suffices to have all
pullbacks; Thus a terminal object, binary products and binary equalisers
suffice for finite completeness.

```agda
with-equalisers 
  : Terminal 
  → (∀ A B → Product A B)
  → (∀ {A B} (f g : Hom A B) → Equaliser f g)
  → Finitely-complete
with-equalisers top prod equ .terminal   = top
with-equalisers top prod equ .products   = prod
with-equalisers top prod equ .equalisers = equ
```

We'll build the pullback of $f : X \to Z$, $g : Y \to Z$ by carving out
the subobject of $X \times Y$ where $f$ and $g$ agree. In particular,
the pullback we want is the object $X \times_Z Y$ in the commutative
diagram below.

~~~{.quiver .short-15}
\[\begin{tikzcd}
  {X\times_ZY} & {X\times Y} & Z
  \arrow[from=1-1, to=1-2]
  \arrow["{f\pi_1}", shift left=1, from=1-2, to=1-3]
  \arrow["{g\pi_2}"', shift right=1, from=1-2, to=1-3]
\end{tikzcd}\]
~~~

```agda
with-equalisers top prod equ .pullbacks {A} {B} {C} f g = pb where
  module Prod = Product (prod A B)

  p1 : Hom Prod.apex C
  p1 = f ∘ Prod.π₁

  p2 : Hom Prod.apex C
  p2 = g ∘ Prod.π₂

  module Equ = Equaliser (equ p1 p2)

  open is-pullback
  open Pullback

  pb : Pullback f g
  pb .apex = Equ.apex
  pb .p₁ = Prod.π₁ ∘ Equ.equ
  pb .p₂ = Prod.π₂ ∘ Equ.equ
  pb .has-is-pb .square = 
    f ∘ Prod.π₁ ∘ Equ.equ ≡⟨ (assoc _ _ _ ·· Equ.equal ·· sym (assoc _ _ _)) ⟩
    g ∘ Prod.π₂ ∘ Equ.equ ∎
```

To show that this object really _is_ a pullback of $f$ and $g$, note
that we can factor any pair of arrows $P' \to X$ and $P' \to Y$ through
the Cartesian product $X \times Y$, and use the universal property of
equalisers to factor _that_ as a unique arrow $P' \to X \times_Z Y$.

~~~{.quiver}
\[\begin{tikzcd}
  {X\times_ZY} & {X\times Y} & Z \\
  {P'}
  \arrow[from=1-1, to=1-2]
  \arrow["{f\pi_1}", shift left=1, from=1-2, to=1-3]
  \arrow["{g\pi_2}"', shift right=1, from=1-2, to=1-3]
  \arrow["{\exists!}", dashed, from=2-1, to=1-1]
  \arrow["{\langle p_1',p_2' \rangle}"', from=2-1, to=1-2]
\end{tikzcd}\]
~~~

```agda
  pb .has-is-pb .limiting {p₁' = p₁'} {p₂' = p₂'} p = 
    Equ.limiting {e′ = Prod.⟨ p₁' , p₂' ⟩} (
      (f ∘ Prod.π₁) ∘ Prod.⟨ p₁' , p₂' ⟩ ≡⟨ pullr Prod.π₁∘factor ⟩
      f ∘ p₁'                            ≡⟨ p ⟩
      g ∘ p₂'                            ≡˘⟨ pullr Prod.π₂∘factor ⟩
      (g ∘ Prod.π₂) ∘ Prod.⟨ p₁' , p₂' ⟩ ∎
    )
  pb .has-is-pb .p₁∘limiting = pullr Equ.universal ∙ Prod.π₁∘factor
  pb .has-is-pb .p₂∘limiting = pullr Equ.universal ∙ Prod.π₂∘factor
  pb .has-is-pb .unique p q = 
    Equ.unique (sym (Prod.unique _ (assoc _ _ _ ∙ p) (assoc _ _ _ ∙ q)))
```

## With pullbacks

We'll now prove the converse: That a terminal object and pullbacks
implies having all products, and all equalisers. 

```agda
with-pullbacks 
  : Terminal 
  → (∀ {A B C} (f : Hom A C) (g : Hom B C) → Pullback f g)
  → Finitely-complete
with-pullbacks top pb = fc where
  module top = Terminal top
  mkprod : ∀ A B → Product A B
```

We'll start with the products, since those are simpler. Observe that we
can complete a product diagram (like the one on the left) to a pullback
diagram (like the one on the right) by adding in the unique arrows into
the terminal object $*$. 

<div class=mathpar>

~~~{.quiver .tall-2}
\[\begin{tikzcd}
  {P'} \\
  & {A\times B} && B \\
  \\
  & A
  \arrow["g", from=2-2, to=2-4]
  \arrow["f"', from=2-2, to=4-2]
  \arrow["{\langle f,g\rangle}", dashed, from=1-1, to=2-2]
  \arrow[curve={height=-12pt}, from=1-1, to=2-4]
  \arrow[curve={height=12pt}, from=1-1, to=4-2]
\end{tikzcd}\]
~~~

~~~{.quiver .tall-2}
\[\begin{tikzcd}
  {P'} \\
  & {A\times B} && B \\
  \\
  & A && {*}
  \arrow["g", from=2-2, to=2-4]
  \arrow["f"', from=2-2, to=4-2]
  \arrow["{\langle f,g\rangle}", dashed, from=1-1, to=2-2]
  \arrow[curve={height=-12pt}, from=1-1, to=2-4]
  \arrow[curve={height=12pt}, from=1-1, to=4-2]
  \arrow["{!}", from=2-4, to=4-4]
  \arrow["{!}"', from=4-2, to=4-4]
\end{tikzcd}\]
~~~

</div>
```agda
  mkprod A B = prod where
    module Pb = Pullback (pb (top.! {x = A}) (top.! {x = B}))
    open is-product
    open Product

    prod : Product A B
    prod .apex = Pb.apex
    prod .π₁ = Pb.p₁
    prod .π₂ = Pb.p₂
    prod .has-is-product .is-product.⟨_,_⟩ p1' p2' = 
      Pb.limiting {p₁' = p1'} {p₂' = p2'} (top.!-unique₂ _ _)
    prod .has-is-product .is-product.π₁∘factor = Pb.p₁∘limiting
    prod .has-is-product .is-product.π₂∘factor = Pb.p₂∘limiting
    prod .has-is-product .unique other p q = Pb.unique p q

  mkeq : ∀ {A B} (f g : Hom A B) → Equaliser f g
  mkeq {A = A} {B} f g = eq where
```

For equalisers, the situation is a bit more complicated. Recall that, by
analogy with the case in Set, we can consider the equaliser to be the
solution set of $f(x) = g(x)$, for some $f, g : A \to B$. We can
consider the two sides of this equation as a _single_ map $\langle f, g
\rangle : A \to B \times B$; The equation is solved where _this_ pairing
map equals some $(x,x)$. We can thus build equalisers by pulling back
along the diagonal map:

~~~{.quiver}
\[\begin{tikzcd}
  {\mathrm{eq}(f,g)} && A \\
  \\
  B && {B \times B}
  \arrow["{\mathrm{equ}}", from=1-1, to=1-3]
  \arrow[from=1-1, to=3-1]
  \arrow["{\langle f,g\rangle}", from=1-3, to=3-3]
  \arrow["\lrcorner"{anchor=center, pos=0.125}, draw=none, from=1-1, to=3-3]
  \arrow["{\langle \mathrm{id}, \mathrm{id}\rangle}"', from=3-1, to=3-3]
\end{tikzcd}\]
~~~

<!--
```agda
    module Bb = Product (mkprod B B)
    ⟨id,id⟩ : Hom B Bb.apex
    ⟨id,id⟩ = Bb.⟨ id , id ⟩

    ⟨f,g⟩ : Hom A Bb.apex
    ⟨f,g⟩ = Bb.⟨ f , g ⟩

    module Pb = Pullback (pb ⟨id,id⟩ ⟨f,g⟩)

    open is-equaliser
    open Equaliser
```
-->

The actual equaliser map is the top, horizontal face (what the code
calls `Pb.p₂`{.Agda}), so we must show that, composed with this map, $f$
and $g$ become equal. Here's where we use the fact that pullback
squares, well, commute: We know that $f$ is $\pi_1 \circ \langle f , g
\rangle$, and that $\langle f , g \rangle \circ \mathrm{equ} = \langle
\mathrm{id}, \mathrm{id} \rangle$ (since the square above is a pullback). 

But both projections out of $\langle \mathrm{id}, \mathrm{id} \rangle$
are equal, so we can apply commutativity of the square above _again_ to
conclude that $f \circ \mathrm{equ} = g \circ \mathrm{equ}$.

```agda
    eq : Equaliser f g
    eq .apex = Pb.apex
    eq .equ = Pb.p₂
    eq .has-is-eq .equal = 
      f ∘ Pb.p₂               ≡˘⟨ pulll Bb.π₁∘factor ⟩
      Bb.π₁ ∘ ⟨f,g⟩ ∘ Pb.p₂   ≡⟨ ap (Bb.π₁ ∘_) (sym Pb.square) ⟩
      Bb.π₁ ∘ ⟨id,id⟩ ∘ Pb.p₁ ≡⟨ pulll Bb.π₁∘factor ∙ sym (pulll Bb.π₂∘factor) ⟩
      Bb.π₂ ∘ ⟨id,id⟩ ∘ Pb.p₁ ≡⟨ ap (Bb.π₂ ∘_) Pb.square ⟩
      Bb.π₂ ∘ ⟨f,g⟩ ∘ Pb.p₂   ≡⟨ pulll Bb.π₂∘factor ⟩
      g ∘ Pb.p₂               ∎
```

We must now show that if $e'$ is another map which equalises $f$ and
$g$, then it fits into a commutative diagram like the one below, so that
we may conclude the dashed arrow $E' \to \mathrm{eq}(f,g)$ exists and is
unique.

~~~{.quiver .tall-2}
\[\begin{tikzcd}
  {E'} \\
  & {\mathrm{eq}(f,g)} && A \\
  \\
  & B && {B \times B}
  \arrow["{\mathrm{equ}}", from=2-2, to=2-4]
  \arrow["{\langle f, g \rangle}", from=2-4, to=4-4]
  \arrow["\lrcorner"{anchor=center, pos=0.125}, draw=none, from=2-2, to=4-4]
  \arrow[from=2-2, to=4-2]
  \arrow["{\langle \mathrm{id}, \mathrm{id} \rangle}"', from=4-2, to=4-4]
  \arrow["{e'}", curve={height=-6pt}, from=1-1, to=2-4]
  \arrow["{\exists!}"', dashed, from=1-1, to=2-2]
\end{tikzcd}\]
~~~

A bit of boring limit-chasing lets us conclude that this diagram _does_
commute, hence the dashed arrow _does_ exist (uniquely!), so that the
top face $\mathrm{equ} : \mathrm{eq}(f,g) \to A$ in our pullback diagram
is indeed the equaliser of $f$ and $g$.

```agda
    eq .has-is-eq .limiting {e′ = e′} p = 
      Pb.limiting (Bb.unique₂ _ refl refl (⟨f,g⟩ ∘ e′) (sym p1) (sym p2))
      where
        p1 : Bb.π₁ ∘ ⟨id,id⟩ ∘ f ∘ e′ ≡ Bb.π₁ ∘ ⟨f,g⟩ ∘ e′
        p1 = 
          Bb.π₁ ∘ ⟨id,id⟩ ∘ f ∘ e′   ≡⟨ cancell Bb.π₁∘factor ⟩
          f ∘ e′                     ≡˘⟨ pulll Bb.π₁∘factor ⟩
          Bb.π₁ ∘ ⟨f,g⟩ ∘ e′         ∎
        
        p2 : Bb.π₂ ∘ ⟨id,id⟩ ∘ f ∘ e′ ≡ Bb.π₂ ∘ ⟨f,g⟩ ∘ e′
        p2 =
          Bb.π₂ ∘ ⟨id,id⟩ ∘ f ∘ e′   ≡⟨ cancell Bb.π₂∘factor ⟩
          f ∘ e′                     ≡⟨ p ⟩
          g ∘ e′                     ≡˘⟨ pulll Bb.π₂∘factor ⟩
          Bb.π₂ ∘ ⟨f,g⟩ ∘ e′         ∎

    eq .has-is-eq .universal = Pb.p₂∘limiting
    eq .has-is-eq .unique {F} {e′ = e′} {lim' = lim'} e′=p₂∘l = 
      Pb.unique path (sym e′=p₂∘l) 
      where
        path : Pb.p₁ ∘ lim' ≡ f ∘ e′
        path = 
          Pb.p₁ ∘ lim'                   ≡⟨ insertl Bb.π₁∘factor ⟩
          Bb.π₁ ∘ ⟨id,id⟩ ∘ Pb.p₁ ∘ lim' ≡⟨ ap (Bb.π₁ ∘_) (extendl Pb.square) ⟩
          Bb.π₁ ∘ ⟨f,g⟩ ∘ Pb.p₂ ∘ lim'   ≡⟨ ap (Bb.π₁ ∘_) (ap (⟨f,g⟩ ∘_) (sym e′=p₂∘l)) ⟩
          Bb.π₁ ∘ ⟨f,g⟩ ∘ e′             ≡⟨ pulll Bb.π₁∘factor ⟩
          f ∘ e′                         ∎
```

Putting it all together into a record we get our proof of finite completeness:

```agda
  fc : Finitely-complete
  fc .terminal = top
  fc .products = mkprod
  fc .equalisers = mkeq
  fc .pullbacks = pb
```
