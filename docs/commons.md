Strukt Commons
===

Chainable arrays and strings, dot-separated collections, date ranges, token
queries, registries, message stacks, JSON helpers, and common global functions.

Requires PHP 8.2+ and `strukt/base`. Optional integrations such as Ramsey UUID
and Symfony's English inflector are used when installed; standalone fallbacks
remain available.

## Install

```sh
composer require strukt/commons
```

Composer loads the `Strukt\` namespace and the global helpers. Require the
normal autoloader before using the helpers.

## Collections and dot paths

`collect()` creates a mutable collection with dot-path reads and writes.

```php
$contact = collect([]);
$contact->set('mobile', '+2540770123456');
$contact->set('work-phone', '+2540202345678');

$user = collect([]);
$user->set('contacts', $contact->yield());

echo $user->get('contacts.mobile');
$user->exists('contacts.mobile'); // true

$user->remove('contacts.work-phone');
$user->keys('contacts');          // ['mobile']
```

The `dot()`, `attach()`, and `detach()` helpers work directly on arrays.

```php
$data = [
    'user' => [
        'firstname' => 'Gene',
        'db' => [
            'config' => ['username' => 'root'],
        ],
    ],
];

echo dot('user.db.config.username', $data); // root

attach('user.db.config.host', $data, 'localhost');
detach('user.db.config.username', $data);
```

## Value objects

Value objects expose their raw value through `yield()`. Transformations return a
new wrapper so chains do not mutate the original.

### DateTime

```php
$start = when();
$end = when('+30 days');
$random = $start->rand($end);

$end->gt($start);   // true
$start->btwn($start, $end);

$start->gte($end);
$start->lte($end);
$start->equals($end);
$start->same(new DateTime());

$start->clone('+1 day');
$start->reset(); // 00:00:00
$start->last();  // 23:59:59

echo $start;
```

`same()` compares calendar dates; `equals()` compares the complete instant.

### Today and date periods

`period()` controls an inclusive date range and can simulate the current day.

```php
$period = period();
$period->create(when('1900-01-01'), when('1963-12-31'));
$period->reset(when('1960-03-23'));

$fakeToday = today();
echo $fakeToday->format('Y-m-d'); // 1960-03-23

$fakeToday->hasPeriod(); // true
\Strukt\Today::withDate(when('1959-01-01'))->isValid();
$period->reset(); // restore the real today
```

### String

```php
$value = str('Strukt Commons');

echo $value->toSnake(); // strukt_commons
echo $value->toCamel(); // StruktCommons

$value->startsWith('Strukt');    // true
$value->endsWith('Commons');     // true
$value->contains('Commons');     // true
$value->empty();                 // false
$value->replace('Commons', 'Library');
$value->replaceFirst('S', 'X');
$value->replaceLast('s', 'z');

str('[commons]')->btwn('[', ']'); // commons
str('log')->pad('.')->right(3);   // log...
str('admin@example.com')->is()->email(); // true
```

Other operations include `prepend()`, `concat()`, `len()`, `count()`,
`split()`, `slice()`, `first()`, `last()`, `part()`, `repeat()`, `toUpper()`,
`toLower()`, `replaceAt()`, and `newline()`.

### Array

`arr()` returns a chainable wrapper implementing `Countable` and
`IteratorAggregate`.

```php
$numbers = arr([1, 2, 2, 3]);

$numbers->sum();     // 8
$numbers->product(); // 12

$numbers->uniq()->values()->yield();   // [1, 2, 3]
$numbers->distinct()->yield();         // [1 => 1, 2 => 2, 3 => 1]
```

Compose transformations:

```php
$rows = arr([
    ['name' => 'Ada', 'score' => 92],
    ['name' => 'Grace', 'score' => 88],
]);

$names = $rows->column('name')->yield();
$ordered = $rows->order()->desc('score')->yield();
$flat = arr(['user' => ['name' => 'Ada']])->level()->yield();

$upper = arr(['ada', 'grace'])
    ->map(fn (int|string $key, string $value): string => strtoupper($value))
    ->yield();
```

Predicates and selectors:

```php
arr(['ada', 'grace'])->isof()->strings(); // true
arr([1, 1])->are()->all(1);               // true
arr(['name' => 'Ada'])->is()->map();      // true
arr([1, 2, 3])->only([1, 3])->yield();    // [1, 3]
```

The wrapper also provides `push()`, `enqueue()`, `prequeue()`, `remove()`,
`merge()`, `enjoin()`, `slice()`, `reverse()`, `flip()`, `rehash()`, `filter()`,
`each()`, `diff()`, `cross()`, `join()`, `contains()`, `has()`, `skip()`,
`jump()`, `stopAt()`, and pointer helpers.

Nested arrays can be flattened with optional key prefixes:

```php
arr([
    'user' => [
        'name' => 'Ada',
        'roles' => ['admin', 'author'],
    ],
])->level()->prefix('payload')->yield();

// ['payload.user.name' => 'Ada', 'payload.user.roles.0' => 'admin', ...]
```

## Token queries

```php
$query = token('page:2|tag:php,composer');

$query->has('page'); // true
$query->get('tag');  // ['php', 'composer']

$query->set('sort', 'name')->remove('page');
echo $query->yield(); // tag:php,composer|sort:name

echo tokenize(['page' => 2, 'tag' => ['php', 'composer']]);
```

## Stack and messages

`stack()` records messages in a bounded process-local stack. The default limit
is ten messages.

```php
\Strukt\Stack::clear();

$message = stack('User was not found');
$message->add('A retry was scheduled');

$messages = \Strukt\Stack::get();
$messages->yield();

$filtered = \Strukt\Stack::get('/retry/');
$filtered->yield();

\Strukt\Stack::limit(20);
```

## JSON

```php
$payload = json(['name' => 'Peter', 'active' => true]);

echo $payload->pp();     // pretty JSON
$encoded = $payload->encode();
$decoded = json($encoded)->decode();
json($encoded)->valid(); // true
```

Encoding and decoding use `JSON_THROW_ON_ERROR`, so malformed data raises
`JsonException`.

## Registry, configuration, and helpers

```php
reg('user.name', 'Ada');
echo reg('user.name');

echo config('app.name');
echo format('date', when('2024-01-01'));
echo plural('story');   // stories
echo singular('stories'); // story
echo uuid();

provider('provider.core');
```

Low-level `alias()`, `is_map()`, `notnull()`, `negate()`, and `raise()` come
from `strukt/base`. `form()` and `form_check()` come from `strukt/form`;
`number()`, `matrix()`, and `pipeline()` come from `strukt/math`.
