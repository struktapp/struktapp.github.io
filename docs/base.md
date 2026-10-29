Strukt Base
===

Typed runtime foundations for the Strukt package set: process-local aliases and
environment values, cached reflection wrappers, conditional helper registration,
a PSR-11 container, and Symfony-backed debugging helpers.

This is the lowest-level Strukt package. Most applications get it through the
framework, but it can be used on its own.

## Install

```sh
composer require strukt/base
```

Requires PHP 8.2+, `psr/container`, and `symfony/var-dumper`. Composer loads
`src/helpers.php`; the helpers are available as soon as the normal autoloader is
required.

```php
require __DIR__.'/vendor/autoload.php';

alias('user', App\User::class);
env('APP_ENV', 'production');

$user = ref(App\User::class)->makeArgs(['Ada'])->getInstance();
```

## Global helpers

| Helper | Purpose |
| --- | --- |
| `helper()` | Select a helper package or inspect the helper registry. |
| `helper_add()` | Register a global helper name for the selected package. |
| `is_map()` | True for a non-empty associative array. |
| `alias()` / `env()` | Read and write process-local aliases and typed values. |
| `ref()` | Create a cached class, object, method, property, or function reference. |
| `app()` | Resolve from the shared container, or return the container. |
| `raise()` | Throw `Strukt\Raise` and record the error. |
| `timezone()`, `negate()`, `notnull()` | Small runtime helpers. |
| `dd()` / `dump()` | Symfony VarDumper output; `dd()` exits. |

## Aliases

Aliases map short names to classes, commands, or events.

```php
alias('user', App\User::class); // App\User::class
alias('user');                  // App\User::class
alias();                        // all alias => target pairs
alias('user*');                 // aliases matching a prefix

Alias::exists('user');          // true
Alias::ls('user');              // matching alias names
Alias::reset();                 // clear the process-local store
```

Registering an existing alias throws `LogicException`; empty names throw
`InvalidArgumentException`.

## Environment values

`env()` stores typed values in memory. It does not modify PHP's process
environment.

```php
env('APP_ENV', 'production');
env('DEBUG', false);

echo env('APP_ENV'); // production
```

Load a dotenv-style file with `Env::withFile()`:

```php
use Strukt\Env;

Env::withFile();          // .env
Env::withFile('.env-dev');

Env::has('APP_ENV');
Env::get('APP_ENV');
Env::set('APP_ENV', 'local');
Env::reset();
```

Blank lines, `#` and `//` comments, an optional `export` prefix, quoted values,
and values containing `=` are supported. Missing keys throw `Strukt\Raise`.

## Reflection

`ref()` caches `ReflectionClass` instances for the lifetime of the process.

```php
$r = ref(App\User::class);
$r->ref();                    // ReflectionClass
$r->makeArgs(['Ada']);        // construct with an array of arguments
$r->make('Ada');              // construct with variadic arguments
$r->noMake();                 // skip the constructor

$r->prop('name')->get();
$r->prop('name')->set('Ada');
$r->method('getName')->invoke();

ref('array_sum')->invoke([1, 2, 3]); // 6
```

Construction may also come from an existing instance with
`ref($object)` or `Ref::createFrom($object)`. Use `Ref::resetCache()` in
long-running workers when classes are reloaded.

## Helper registry

Helpers are conditionally declared so packages can share one process.

```php
helper();        // package names
helper('base');  // helper names registered by base

helper_add('custom_helper'); // true when the name can be defined

Helper::create('my-package')->register('my_helper');
Helper::reset();
```

## Container

The shared PSR-11 container is exposed through `app()`. It supports shared
bindings, per-resolve factories, prebuilt instances, aliases, autowiring, and
named constructor overrides.

```php
use Strukt\Container\Container;

$container = app(); // or Container::getInstance()

$container->set(Mailer::class, fn (Container $c) => new Mailer($c->get(Transport::class)));
$container->factory(Report::class, fn () => new Report());
$container->instance(Config::class, $config);
$container->alias('mailer', Mailer::class);

app(Mailer::class);                  // shared binding
app(User::class, ['id' => 7]);       // autowired with override
$container->make(User::class);       // autowire directly
```

Missing identifiers throw `Strukt\Container\NotFoundException`; failed
construction throws `Strukt\Container\ContainerException`. Both implement the
PSR-11 exception interfaces.

## Errors and debugging

`Raise` is the runtime exception used by the helpers and records each message.

```php
raise('Something went wrong', 500);

Raise::errors();
Raise::reset();

dd($value); // dump and exit
dump($value);
```
