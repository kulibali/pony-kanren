# Pony-Kanren

Pony-Kanren is an implementation of [microKanren](http://minikanren.org/) for
the [Pony programming language](https://www.ponylang.org).

The easiest way to use Pony-Kanren in your code is via
[Pony-Stable](https://github.com/ponylang/pony-stable): run

```bash
stable add github kulibali/pony-kanren
```
to add Pony-Kanren as a dependency to your `bundle.json`, and then
```pony
use "kanren"
```

in your code to import the library.

You can find library documentation
[here](http://kulibali.github.io/pony-kanren/kanren--index/).

## Workflow

To set up a logic expression and query its results:

- Create an empty `State` value, with a functor that can unify the data type
  that your variables refer to.
- Create a `Goal` that represents a logical expression.
  - Use the `Goal.fresh()` function to obtain a new state that knows about a
    variable.  That variable will now be visible to subgoals.  Note that if you
    use `Goal.fresh()` more than once, the variables will be disjoint.  If their
    scopes intersect, you will get errors.
  - Use `Goal.conj()` for the logical AND operation.  This will succeed only if
    both its subgoals succeed.
  - `Goal.disj()` is logical OR.  This will succeed if either of its subgoals
    succeeds.
  - `Goal.unify_vars()` will unify two variables, if possible.
  - `Goal.unify_vals()` will attempt to structurally unify two values.
  - `Goal.unify_val()` will bind a variable to a value, if possible.
- A goal is a function that returns a sequence of states.  Each state may contain
  variable bindings that represent a solution to the logical expression.
  - If there is no solution for the expression, the sequence will be empty.
  - If an error occurrs when solving, the sequence will contain a solution with
    unbound variables and some error messages obtainable via its `get_errors()`
    method.
- You can query the value of any variable in a particular state using the
  state's `apply()` method.

## Example

The following Pony code initializes and solves the expression
`A = B && (B = 123 || B = 456)`.  Running the goal will result in a sequence of
two states, one where `A` and `B` are bound to `123`, and one where `A` and `B`
are bound to `456`:

```pony
use "kanren"

let a = Var("A")
let b = Var("B")
let g =
  Goals.fresh[USize](a,
    Goals.fresh[USize](b,
      Goals.conj[USize](
        Goals.unify_vars[USize](b, a),
        Goals.disj[USize](
          Goals.unify_val[USize](b, 123),
          Goals.unify_val[USize](b, 456)
        ))))

let results = g(State[USize](UnifyEq[USize]))
let s1 = results.next()?
s1(a) == 123 // true
s1(b) == 123 // true
let s2 = results.next()?
s2(a) == 456 // true
s2(b) == 456 // true
```
