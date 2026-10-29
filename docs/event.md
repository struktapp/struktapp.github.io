Strukt Event
===

Fast callable events and named command queues. This package is the `strukt5`
line and a drop-in match for the `strukt4/event` public API.

Internally it fixes the halt-hook leak in `Loop`, makes `run()` exception-safe,
and stops treating array callables such as `[$object, 'method']` as argument
arrays.

## Install

```sh
composer require strukt/event
```

Requires PHP 8.2+ and `strukt/base`. Composer loads `src/helpers.php` and
installs the global `event()` and `cmd()` helpers.

## Global helpers

```php
event(string $name, ?callable $function = null): ?Strukt\Event
cmd(string $name, ?array $args = null): mixed
```

Register and invoke an event:

```php
$login = event(
    'login',
    static fn (string $user, string $password): bool =>
        $user === 'admin' && $password === 'secret',
);

$accepted = $login?->apply('admin', 'secret')->exec();
```

Calling `event()` again with the same name replaces the callback but preserves
its position in the command order.

Execute a registered command:

```php
cmd('login', ['admin', 'secret']);
cmd('greet', ['name' => 'Ada']);
```

Both return `null` when the command is unknown.

## Event API

`Strukt\Event` retains a Closure and its `ReflectionFunction`.

```php
use Strukt\Event;

$event = Event::create(fn (string $name): string => "Hello {$name}");

$event->apply('Ada')->exec();             // Hello Ada
$event->applyArgs(['name' => 'Ada'])->exec();
$event->clear();

$event(); // shorthand for exec()
```

`apply()` stores positional arguments. `applyArgs()` accepts positional or named
keys; named keys use PHP named-argument invocation. Each call replaces the
arguments stored by the previous one. Calling `exec()` without prepared
arguments invokes the callback with no arguments.

## Command API

`Strukt\Cmd` is the process-local named command registry.

```php
use Strukt\Cmd;

Cmd::add('greet', static fn (string $name): string => "Hello {$name}");
Cmd::add('welcome', ['name' => 'Ada'], static fn (string $name): string => "Hi {$name}");

Cmd::get('greet');   // callable|null
Cmd::exists('greet'); // true
Cmd::ls();            // ['greet', 'welcome']
Cmd::ls('^g');        // ['greet']

Cmd::doc('greet');
Cmd::exec('welcome'); // uses the stored default argument
Cmd::reset();
```

`Ls()` filters with a regular-expression body. `doc()` returns a compact
description from the callback's doc comment, or the command name when none is
available. Executing an unknown command throws `InvalidArgumentException`.

## Loop

`Strukt\Loop` extends `Cmd` with an insertion-ordered queue and a repeatable
halt hook.

```php
use Strukt\Loop;

Loop::add('prepare', static function (): void {
    // work
});

Loop::halt(static function (): void {
    // invoked while paused; call Loop::pause(false) to continue
});

Loop::run();
```

`run()` executes commands in insertion order and removes them after they run.
While paused, the halt callback is re-invoked each iteration, which makes the
loop a read-eval-print loop. Calling `halt()` again replaces the previous
terminal callback instead of leaving a stale command behind.

Options:

```php
Loop::run(['alias' => 'worker']); // filter through the base alias map
Loop::run(['limit' => 1000]);     // cap halt iterations
```

If a command or hook throws, `run()` removes the terminal command before the
exception escapes, so a failed run leaves no dangling halt command.

## Notes

- `Event` converts a callable to a `Closure` and reflection object once.
- `Cmd` stores callbacks, defaults, and insertion order in dedicated arrays.
- `Loop` re-invokes the halt hook while paused, enabling REPL-style loops.
- Array callables are recognized as callbacks, not argument arrays.
