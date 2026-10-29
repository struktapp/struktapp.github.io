Strukt Tasker
===

A small PHP task runner for project-local commands. It loads `tasker.php`,
registers named PHP callables, executes one task from the shell, and provides
helpers for composing tasks, watching files, and running external commands.

<center><img src="tasker.jpg" width="200" height="200"></center>

## Install

```sh
composer require strukt/tasker
```

Requires PHP 8.2+, `proc_open()`, `ext-zip`, and the `strukt/event`,
`strukt/fs`, and `strukt/process` packages.

The `tasker` executable is installed in `vendor/bin`:

```sh
vendor/bin/tasker list
```

When run in a project without `tasker.php`, Tasker creates a starter file with a
`test` task.

## Quick start

Create `tasker.php`:

```php
<?php

/**
 * Say hello to someone.
 */
task('hello', static function (string $name): void {
    writeln("Hello {$name}!");
});

/**
 * Run a shell command.
 */
task('php-version', static function (): void {
    [$output, $error] = run(PHP_BINARY.' --version');
    echo $error === '' ? $output : $error;
});
```

Run tasks with positional arguments:

```sh
vendor/bin/tasker list
vendor/bin/tasker hello Ada
vendor/bin/tasker php-version
```

Tasker passes command-line values to callbacks as strings. Required and maximum
positional counts are checked before invocation.

## Built-in commands

| Command | Description |
| --- | --- |
| `list` | List built-in and project tasks in registration order. |
| `help` | Show usage and the task list. |
| `version` | Print the Tasker version. |

The equivalent switches are `--help`/`-h` and `--version`/`-V`.

| Exit status | Meaning |
| ---: | --- |
| `0` | Task completed successfully. |
| `1` | The task file or task raised an exception. |
| `64` | No task was supplied. |
| `127` | The requested task was not registered. |

## Task-file helpers

Composer loads `src/helpers.php` automatically. Helpers are declared
conditionally so a host application can provide the same name.

### task()

```php
task(string $name, callable $function): Strukt\Task
```

Registers a task. The callback's first PHPDoc description paragraph is shown by
`list`.

```php
/** Remove generated files. */
task('clean', static function (): void {
    // cleanup
});
```

Task names are unique; registering a duplicate throws `LogicException`.

### go()

```php
go(string $name, mixed ...$args): mixed
```

Executes another registered task and returns its result.

```php
task('all', static function (): void {
    go('clean');
    go('build');
});
```

### writeln()

```php
writeln(string|int|float|bool|null $line): void
```

Writes one scalar value followed by `PHP_EOL`.

### watch()

```php
watch(string $path, callable $callback, int $wait = 1): void
```

Polls a file or directory until the process is stopped. The callback receives an
array of added, removed, or content-changed paths. The first scan establishes a
baseline and does not call the callback.

```php
task('watch:js', static function (): void {
    watch('app/js', static function (array $files): void {
        foreach ($files as $file) {
            writeln("Changed: {$file}");
        }
    });
});
```

`watch()` throws `InvalidArgumentException` for an unknown path or negative
interval. `TaskWatcher` adds an iteration limit and an injectable sleeper for
tests.

### run()

```php
run(
    string $command,
    ?callable $callback = null,
    ?string $input = null,
): array{0: string, 1: string}
```

Runs a shell command and returns `[output, error]`. The callback receives
complete standard-output lines, including line endings. Input is written exactly
to standard input, then the input pipe is closed.

```php
[$output, $error] = run('printf "ready\n"');

[$output, $error] = run(
    PHP_BINARY.' -r '.escapeshellarg('echo stream_get_contents(STDIN);'),
    input: "from task\n",
);
```

Use `CommandRunner` directly when the process exit code matters.

## Task API

`Strukt\Task` wraps `Strukt\Event`. It owns the name, description, and parameter
metadata, while the event retains the callback as a Closure and exposes its
reflection.

```php
new Task(string $name, callable $callback, ?string $description = null)

$task->name();
$task->callback();
$task->reflection();
$task->description();
$task->parameters();
$task->invoke(array $arguments = []);
$task->invoke(mixed ...$arguments);
```

`parameters()` returns records with `name`, `type`, `required`, and `variadic`
keys. `invoke()` supports positional arrays and PHP named-argument arrays.
Invalid positional counts throw `InvalidArgumentException`.

## Registry API

`Strukt\TaskRegistry` is an ordered, isolated collection.

```php
$registry = new TaskRegistry();

$registry->add(string $name, callable $callback, ?string $description = null);
$registry->get(string $name);
$registry->has(string $name);
$registry->names();
$registry->all();
$registry->run(string $name, array $arguments = []);
$registry->reset();
$registry->count();
```

The registry implements `IteratorAggregate` and can be iterated as `Task` values
keyed by task name. `add()` rejects duplicate names.

The active registry can be selected explicitly:

```php
$previous = TaskRegistry::activate($registry);

TaskRegistry::current();
TaskRegistry::active();
TaskRegistry::restore($previous);
```

Calling `task()` or `go()` outside an active application throws
`LogicException`.

## Application API

```php
new Tasker(
    ?string $directory = null,
    string $taskFile = 'tasker.php',
    ?string $version = null,
    ?callable $outputWriter = null,
    ?callable $errorWriter = null,
)

$tasker->run(array $argv): int
$tasker->registry(): TaskRegistry
$tasker->taskFile(): string
$tasker->taskList(): string
Tasker::defaultTaskFile(): string
```

`directory` defaults to the current working directory. A relative task-file name
is resolved below that directory. Output and error writers receive one string at
a time, which makes the CLI easy to test or embed.

`bin/tasker` constructs this class with the process argument vector and exits
with the value returned by `run()`.

## Watcher API

```php
new TaskWatcher(
    ?callable $sleeper = null,
    ?callable $fingerprinter = null,
)

$watcher->watch(
    string $path,
    callable $callback,
    int $interval = 1,
    ?int $iterations = null,
): void

$watcher->snapshot(string $path): array<string, string>
```

The default fingerprinter uses SHA-256. Snapshots are path-sorted and compare
added, removed, and changed files. A null iteration count runs until the process
is interrupted.

## Command API

`Strukt\CommandRunner` returns a `Strukt\CommandResult`.

```php
$runner = new CommandRunner();

$result = $runner->run(
    string $command,
    ?callable $callback = null,
    ?string $input = null,
): CommandResult

$result->output();
$result->error();
$result->exitCode();
$result->successful();
$result->toArray();
```

`successful()` is true only for exit code zero. `toArray()` returns the output
and error tuple used by the global `run()` helper.
