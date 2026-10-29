Strukt Math
===

Immutable numeric and matrix primitives. The package keeps the existing
`strukt4/math` behaviour and adds aggregation, roots, deltas, vector products,
and explicit transformation pipelines.

## Install

```sh
composer require strukt/math
```

## Number

`Strukt\Number` wraps one `int` or `float` and returns a new instance from every
operation.

```php
use Strukt\Number;

$sum = Number::sum([1, 2, 2.5]); // 5.5, a Number
$mean = Number::sum($column)->parts(count($column));

$average = number(10)->add(0.2)->parts(3); // 3.4
```

### Arithmetic

```php
$num = number(1000)
    ->add(200)       // 1200
    ->subtract(100)  // 1100
    ->times(2)       // 2200
    ->parts(4);       // 550

$num->mod(9);
$num->raise(2);
$num->ratio(1, 3);   // two parts in the given ratio

$num->yield();
$num->reset();
echo $num;
```

`parts()` throws `DivisionByZeroError` on a zero divisor. Use `partsOr()` for a
safe fallback:

```php
number(10)->partsOr(0, 7); // 7
number(10)->partsOr(0);    // 0
```

### Aggregation and clamping

```php
Number::sum([1, 2, 3]);  // 6, int
Number::sum([1, 2.5]);   // 3.5, float
Number::sum([]);         // 0
Number::sum($iterable);  // any iterable

number(0)->max(1)->yield(); // 1
number(7)->max()->yield();  // 7
number(9)->min(5)->yield(); // 5
```

`sum()` preserves the input kind: an `int` only when every input is an `int`.
`min()` and `max()` accept raw numbers or `Number` objects.

### Sign, rounding, and roots

```php
number(-4.5)->abs();   // 4.5
number(2.1)->ceil();   // 3, integer
number(2.9)->floor();  // 2, integer
number(20.5111111)->round(2); // 20.51, float

number(8)->log(2);     // 3
number(1000)->log10(); // 3
number(1)->exp();      // 2.718281828459045
```

`ceil()` and `floor()` yield integers so they can index a sequence; `round()`
still yields a float. `log()` defaults to the natural logarithm and rejects a
base of zero or less.

### Comparison

```php
number(5)->compare(3);          // 1
number(5)->compare(5.0);        // 0
number(5)->compare(number(3));  // 1

$num->gt(302499);
$num->gte(302500);
$num->lt(302501);
$num->lte(302501);
$num->equals(-302500);
$num->negate();
```

Floating-point comparisons can use an absolute delta:

```php
number(0.1)->add(0.2)->equals(0.3);               // false
number(0.1)->add(0.2)->equalsWithDelta(0.3);      // true
number(0.1)->add(0.2)->equalsWithDelta(0.3, 0.0); // false
```

Other helpers include `format()`, `type()`, and `Number::random($count, $min,
$max)`.

## Matrix

`Strukt\Matrix` is an immutable matrix of `Number` values. Every operation
returns a new matrix.

```php
use Strukt\Matrix;

$a = Matrix::create([[1, 2], [3, 4]]);
$b = Matrix::create([[1, 1], [1, 1]]);

$a->add($b)->yield();      // [[2, 3], [4, 5]]
$a->subtract($b)->yield(); // [[0, 1], [2, 3]]
$a->scale(2)->yield();     // [[2, 4], [6, 8]]
```

`add()` and `subtract()` require matching shapes and raise `Strukt\Raise`
otherwise.

### Multiplication

```php
$a = matrix([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);
$b = matrix([[11, 22, 33], [44, 55, 66], [77, 88, 99]]);

echo $a->multiply($b);
$a->multiply($b)->yield();
```

### Vector products

```php
Matrix::dot([1, 2], [3, 4]);                    // 11
Matrix::dot([1, 0], [0, 1]);                    // 0

Matrix::cosineSimilarity([1, 2, 3], [1, 2, 3]); // 1.0
Matrix::cosineSimilarity([1, 0], [0, 1]);       // 0.0
```

Entries are paired by position, so input keys are irrelevant. Mismatched lengths
throw `InvalidArgumentException`; a zero vector makes `cosineSimilarity()` throw
`DivisionByZeroError`.

### Norms

```php
Matrix::create([[3, 4]])->norm();    // 5, Euclidean
Matrix::create([[1, -2]])->norm(1);  // 3, sum of absolute values
```

Any other norm order throws `InvalidArgumentException`.

## Transformations

### Pipeline

`Pipeline` is mutable and eager: `next()` returns the same object, and each step
runs when it is added.

```php
use Strukt\Pipeline;

pipeline([12, 3, 2])
    ->next(static fn (int $m, int $x): int => $m * $x)
    ->next(static fn (int $mx, int $c): int => $mx + $c)
    ->next(static fn (int $result): int => $result)
    ->yield(); // 38
```

Positional pipelines pair values by position. Scoped pipelines bind them by
parameter name:

```php
scoped(['c' => 12, 'm' => 3, 'x' => 2])
    ->next(static fn (int $m, int $x): int => $m * $x)
    ->next(static fn (int $mx, int $c): int => $mx + $c)
    ->next(static fn (int $result): int => $result)
    ->yield(); // 18
```

`with()` replaces a scope value explicitly. A scoped step may not reuse a key
that was already in the scope. A step that throws leaves the pipeline unchanged;
by-reference parameters are rejected.

```php
$pipeline = pipeline([1]);

$pipeline->isPending(); // true
$pipeline->hasResult(); // false
```

`monos()` is a deprecated shim. Use `pipeline()` or `scoped()`.

### Monad

`Monad` is immutable and holds exactly one value. `of` lifts a value, `bind`
composes a step that may return a monad, `map` is `bind` composed with `of`, and
`join` reveals the value.

```php
use Strukt\Monad;

Monad::of(12)
    ->bind(static fn (int $c): Monad => Monad::of(3)->map(static fn (int $m) => $m * 2))
    ->bind(static fn (int $mx): Monad => Monad::of($mx + 12))
    ->join(); // 18
```

A monad can be shared and joined repeatedly:

```php
$base = Monad::of(2);

$base->map(static fn (int $v): int => $v * 2)->join(); // 4
$base->map(static fn (int $v): int => $v * 3)->join(); // 6
$base->join();                                        // 2
```

`bind` is strict, so errors surface where they occur. `map` does not flatten a
returned monad; use `bind` when flattening is required.
