Strukt Rules
===

Data-driven validation rules, a parser, a registry, and a value validator.

Every built-in rule has a declaration in `src/Strukt/Rules/Spec/` describing its
arity, parameter format, failure message, and bail behaviour. Those
specifications compile into one table read by the shared engine, so parsing,
messages, and bail behaviour cannot drift apart.

This package has no runtime dependencies and powers `strukt/form`.

## Install

```sh
composer require strukt/rules
```

## Validating values

`Validator` takes field values and a rules definition keyed by field name. Each
entry is a pipe-separated string or a list of rule tokens.

```php
use Strukt\Rules\Validator;

$errors = (new Validator())->validate(
    ['age' => 12],
    ['age' => 'required|integer|min:18'],
);

// ['age' => ['Age must be at least 18.']]
```

The standalone helper is equivalent:

```php
$errors = check(
    ['age' => 12],
    ['age' => 'required|integer|min:18'],
);
```

Failures are collected per field in declaration order.

## Rules

### Presence

| Rule | Passes when |
| --- | --- |
| `required` | Value is not `null`, `''`, or `[]`. |
| `filled` | Same check as `required`. |
| `nullable` | Value is not `null`. |

`0` and `false` count as present. A non-required field with an absent value skips
its remaining rules.

### Types

| Rule | Passes when |
| --- | --- |
| `string` | Value is a string. |
| `integer` | An int, or a string of digits. `5.5` and `1e3` fail. |
| `numeric` | `is_numeric`. |
| `boolean` | Bool, or `0`/`1`/`true`/`false`/`on`/`off`/`yes`/`no`. |
| `array` | Value is an array. |

### Formats

| Rule | Passes when |
| --- | --- |
| `email` | `FILTER_VALIDATE_EMAIL`. |
| `url` | `FILTER_VALIDATE_URL`. |
| `uuid` | RFC 4122 UUID. |
| `ip` | `FILTER_VALIDATE_IP`. |
| `date` | A real date in the given format, default `Y-m-d`. |
| `regex` | The pattern matches. |
| `alpha` | Letters only, across scripts. |
| `alpha_num` | Letters and numbers, across scripts. |

`date` rejects invalid calendar dates rather than rolling them over.

```php
'born_on' => 'date',       // Y-m-d
'shipped' => 'date:m/d/Y', // 02/29/2024
```

`regex` accepts patterns with or without delimiters and keeps commas inside a
pattern.

```php
'zip'    => 'regex:^\d{5}$',
'ssn'    => 'regex:/^\d{3}-\d{4}$/',
'csv_id' => 'regex:^\d{3},\d{4}$',
```

### Size

| Rule | Passes when |
| --- | --- |
| `min:n` | Number is at least `n`. |
| `max:n` | Number is at most `n`. |
| `between:min,max` | Number is within the inclusive range. |
| `length:n` | String is exactly `n` characters. |
| `min_length:n` | String is at least `n` characters. |
| `max_length:n` | String is at most `n` characters. |
| `min_count:n` | Array has at least `n` items. |
| `max_count:n` | Array has at most `n` items. |

`min` compares numbers, `min_length` counts characters with `mb_strlen`, and
`min_count` counts items.

### Sets and fields

| Rule | Passes when |
| --- | --- |
| `in:a,b,c` | Value is one of the options. |
| `not_in:a,b,c` | Value is none of the options. |
| `same:field` | Value equals the named field. |
| `different:field` | Value differs from the named field. |

`in` and `not_in` compare loosely, so a submitted `'2'` matches an option list
containing `2`.

### Bailing

Shape-guarding rules such as `required`, `integer`, and `array` stop the field
when they fail, so dependent checks do not add a second confusing message.

```php
'age' => 'required|integer|min:18'
```

For `age => 'not a number'`, only `Age must be an integer.` is reported.

## Messages

Messages use a readable field label and the rule's wording. `email_address`
becomes `Email address`; `min_length:8` on `name` becomes
`Name must be at least 8 characters.`

Override a message per rule or per field/rule:

```php
use Strukt\Rules\Messages;
use Strukt\Rules\Validator;

$validator = new Validator(null, new Messages([
    'email.required' => 'We need an email address to reach you.',
    'min_length' => ':field is too short, use at least :0 characters.',
]));
```

An override is used as written and may use `:field`, `:0`, `:1`, `:list`, and
`:format`.

## Custom rules

Implement `RuleInterface`; a rule receives the value and a `FieldSource` for
reading sibling fields.

```php
use Strukt\Rules\Contract\FieldSource;
use Strukt\Rules\Contract\RuleInterface;

final class AfterRule implements RuleInterface
{
    public function __construct(
        private readonly string $field,
        private readonly string $other,
    ) {
    }

    public function passes(mixed $value, FieldSource $fields): bool
    {
        $other = $fields->value($this->other);

        return is_string($value) && is_string($other) && $value > $other;
    }

    public function message(): string
    {
        return ucfirst($this->field).' must be later than '.$this->other.'.';
    }

    public function bails(): bool
    {
        return false;
    }
}
```

Register globally or pass a private registry:

```php
use Strukt\Rules\RuleRegistry;
use Strukt\Rules\Validator;

RuleRegistry::shared()->extend(
    'after',
    fn (string $field, array $params) => new AfterRule($field, $params[0]),
);

$validator = new Validator(RuleRegistry::shared());
```

`extend()` accepts a rule instance or a factory called with the field name and
parsed parameters. Built-in names cannot be replaced. A misspelled rule name
throws `Strukt\Rules\Exception\UnknownRule` with the available rules.

## Building another engine

The validator has no concept of a form. `strukt/form` is one consumer; any
rule-based engine can be built on the same foundations. Keep a private registry
when an engine should not share custom rules with the rest of the application.

```php
$registry = RuleRegistry::create();
$registry->extend('port', fn (string $field, array $params) => new PortRule($field, $params));

$engine = new Validator($registry, new Messages([
    'required' => ':field is required in the config.',
]));

$errors = $engine->validate($values, $definition);
```
