Strukt Generator
===

Typed PHP class generation, lightweight text templating, and PHPDoc annotation
utilities.

## Install

```sh
composer require strukt/generator
```

Requires PHP 8.2+ and depends on `strukt/base` and `strukt/commons`.

## Templating

```php
$html = template(
    '<h1>{{title}}</h1>',
    ['title' => 'Dashboard'],
);
```

Template syntax:

### Values

```text
Hello {{name}}!
```

Missing values render as an empty string. Null and false values also render as
empty text. Scalars are converted to strings; arrays and objects are JSON encoded
unless an object exposes `__toString()`.

### Lists

```text
{{begin:people}}
<b>{{name}}</b> {{surname}}
{{end:people}}
```

The value at `people` must be an array of arrays. Each item is rendered against
its own values.

### Conditions

```text
{if enabled}
This section is visible.
{/if}
```

The section is trimmed when enabled and removed when the value is missing or
false.

### Rendering order

`Templator::create()` applies directives in this order:

1. list loops;
2. direct values;
3. conditions.

## Templator API

```php
use Strukt\Templator;

Templator::create($template, $data);
Templator::element($template, $data);
Templator::loop($template, $data);
Templator::condition($template, $data);
Templator::console($template);
Templator::trimBlanks($text);
```

`console()` resolves `{color name}` sections when a console color class and
`color()` helper are available.

## DocBlocker

`Strukt\DocBlocker` converts between plain annotation text and PHPDoc blocks.

```php
use Strukt\DocBlocker;

$plain = "@Route(/)\n@Method(GET, POST)";
$docblock = (string) new DocBlocker($plain);

DocBlocker::deBlock($docblock);
```

The helper `deblock($input)` is also available.

## BasicBlock

`Strukt\BasicBlock` parses single-line annotations with parenthesized values.

```php
use Strukt\BasicBlock;

$annotations = BasicBlock::parse(<<<'DOC'
/**
 * @Route(/users)
 * @Method(GET, POST)
 * @Middleware(AuthToken)
 * @Middleware(GVerify)
 */
DOC);

// [
//     'Route' => '/users',
//     'Method' => ['GET', 'POST'],
//     'Middleware' => ['AuthToken', 'GVerify'],
// ]
```

A single argument returns as a string, comma-separated arguments return as a
list, and repeated annotation names are merged into one list. Quoted text and
nested brackets are respected when splitting arguments.

## ClassBuilder

`Strukt\ClassBuilder` renders a PHP class declaration with inheritance,
interfaces, constructors, methods, and method annotations.

```php
use Strukt\ClassBuilder;

$source = (string) new ClassBuilder([
    'ns' => 'App\\Controller',
    'class_name' => 'UserController',
    'extends' => 'BaseController',
    'methods' => [
        [
            'name' => 'index',
            'params' => '',
            'body' => "\treturn [];",
            'block' => ['return' => 'array'],
        ],
    ],
]);
```

Required keys are `ns` and `class_name`. Optional keys are `extends`,
`implements`, and `methods`. Missing or empty namespace/class names throw
`InvalidArgumentException`.

Add a constructor fluently:

```php
$builder->addInit(['string $name', 'int $id']);
```

Method definitions accept `name`, `params`, `body`, and `block`; the block map
becomes PHPDoc tags. Rendering is repeatable and does not duplicate methods.

## Annotation parser

`Strukt\Annotation\Parser` reads `@Name`, `@Name(item)`, `@Name(a, b, c)`, and
`@Name(key=value)` annotations from a class, its properties, and its methods.

```php
use Strukt\Annotation\Parser;

$notes = (new Parser(new ReflectionClass(Worker::class)))->getNotes();
```

The result contains a `class_name` key plus optional `class`, `properties`, and
`methods` keys. Repeated names are merged into one entry whose `items` list holds
every value.

## Global helpers

| Helper | Purpose |
| --- | --- |
| `template($template, $data)` | Render values, loops, and conditions. |
| `huecli($template)` | Resolve optional console color sections. |
| `deblock($input)` | Remove PHPDoc comment delimiters. |

The helpers are declared conditionally and do not overwrite application
functions.
