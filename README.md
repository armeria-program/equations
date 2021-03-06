Equation DSL based on lenses
============================

A farmer has sheep, cows and chicken. If they count sheep and cows together,
there are 20. Cows and chicken come out to 30, and chicken and sheep come out
to 40. How many chicken, sheep and cows are there?

```haskell
import Control.Equation
import Control.Lens
import Control.Monad.State

data Playground = Playground { _sheep :: Float
                             , _cows :: Float
                             , _chicken :: Float
                             }

makeLenses ''Playground

equations = [ evar sheep + evar cows =:= 20
            , evar cows + evar chicken =:= 30
            , evar chicken + evar sheep =:= 40
            ]

solution = execState (solveLinear equations) (Playground 0 0 0)
```

This is a DSL and a solver for a system of linear equations which doesn't
require writing out matrices or massaging the data into a specific format. All
you need is a data type representing the solution space, a lens for every
variable and a set of equations.

Usage
-----

Define the solution space and a set of lenses that operate on it. They can be
either autogenerated record accessors like above, the ones that `Control.Lens`
bundles (for example, `_2` for working with a tuple) or completely custom ones.

The equations are written as usual with `+`, `-`, `*` and other `Num` members,
and `=:=` separating the sides. Variables (lenses) have to be referenced as
`evar lens`. Equations have to be linear - multiplying a variable to another
will result in an error.

To solve the equation system, supply an initial world state and get it back
satisfying all the equations - see the example above.

Caveats
-------

This is a proof of concept, the API is likely to change.

The initial state has to be fully defined as the solver needs to operate on it
(that is, the lenses must not produce errors when applied to it). This is
largely to do with the fact that lenses aren't `Eq`.

For the same reason, lenses have to be independent of each other. That is,
using `radius` and `diameter` of something in the same equation system is not
supported. The solver can try and figure out such pairs, but if there is
something like `area` and `perimeter`, the equation is not even linear anymore.

The errors when trying to construct a non-linear equation are runtime because
`Num` is re-used for equations.

Finally, the code just passes tests and can use a lot of clean-up.
