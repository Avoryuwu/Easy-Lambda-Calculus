# easy_lambda_calculus

Simple and easy to write lambda calculus

## Examples

### lambda!():

Makes a new lambda from a string

```rust
use easy_lambda_calculus::*;

fn main() {
  let l = lambda!("%x|y.y")
  println!("{}", lambda!("%x|y.(x y) &{}", l));
}
//outputs (%x|y.(x y) &(%x|y.y))
```

#### Syntax:

%x.x to define function with input variable x and output defined after the dot.

%x|y.y to define a function with multiple inputs (equivalent to %x.(%y.y) ), variables separated by a pipe | character.

Variables can be multiple characters.
eg: %var.var or %true|false.true are also valid.

(x y) to apply y into function x.
(x y z) is not valid as the reduction order is ambiguous, define it with ((x y) z) or (x (y z)), see Lambda.reduce() for reduction order.

&(x) is used to mark section x for alpha reduction, so you can reuse variable names without any unintended interactions.

{} is used to input a lambda variable into the lambda, uses the same syntax as the `format!()` macro, &{} is shorthand for &({}).

### Lambda.reduce():

A single step of beta reduction

```rust
use easy_lambda_calculus::*;

fn main() {
  println!("{}", lambda!("(%x.(x x)) (%y|z.z)").reduce());
}
//outputs ((%y|z.z) (%y|z.z))
```

#### Reduction order:

Outer brackets are reduced first.
eg: ((λx.(x x)) ((λy.(y y)) (λz.z))) will beta reduce to (((λy.(y y)) (λz.z)) ((λy.(y y)) (λz.z))) and not (λx.(x x)) ((λz.z) (λz.z))

If the left side is an application into a function rather then a function, it will be beta reduced first.
eg: (((λx.x) (λy.(y y))) (λz.z)) will reduce to ((λy.(y y)) (λz.z))

For reduction with sections marked for alpha reduction, see Lambda.alpha_reduce().

### Lambda.alpha_reduce():

Alpha reduce any sections marked for alpha reduction

```rust
use easy_lambda_calculus::*;

fn main() {
  println!("{}", lambda!("(%z.(z z)) &(%z.z)").alpha_reduce());
}
//outputs ((%x.(x x)) (%y.y))
```

#### Alpha reduction properties:

Alpha reduction will rename every variable based on when they show up in the lambda.
They are renamed with the naming scheme: x, y, z, w, a, b ... u, v, xx, xy...

The renamed variables in any section marked for alpha reduction will be different from any other one.

The alpha reduction function can also be used when there are no sections marked for alpha reduction, to rename the lambdas variables based on the naming scheme.

The sections marked for alpha reduction cannot be reduced, however can be reduced into other functions
Note that reducing them into other functions does not remove that they are marked for alpha reduction, and can cause unwanted effects.
For example if multiple variables are substituted with the section marked for alpha reduction, when alpha reduced, every copy will have different variable names.

### Lambda.evaluate():

Evaluate a lambda

```rust
use easy_lambda_calculus::*;

fn main() {
  println!("{}", lambda!("(%x.&(%x.&(%x.x))) &(%x.x)").evaluate());
}
//outputs (%x|y.y)
```

#### Evaluation method:

Evaluation will first alpha reduce the lambda.
It will then automatically beta reduce the lambda until it cannot be reduced anymore.
Lastly it will alpha reduce the lambda again, to output it with predictable names.

### Lambda::from_u16(u16):

Makes a lambda from an integer using church encoding ( (%x|y.(x y)) = 1, (%x|y.x (x y)) = 2, (%x|y.x (x (x y))) = 3...).

### Lambda.as_u16():

Converts a lambda into an integer if it is valid church encoding, returns Option\<u16> if it is a integer, None if it is not.

### Lambda::succ():

Returns a lambda that adds one to church encoded integers where (succ integer) reduces to integer + 1.

### Lambda::add():

Returns a lambda that adds two church encoded integers together where ((add a) b) reduces to a + b.
