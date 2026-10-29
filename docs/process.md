Strukt Process
===

Typed, resource-safe process execution for PHP 8.2 and later. The package
supports sequential command batches, line callbacks, captured output,
interactive processes, supervised worker pools, and a managed supervisor CLI.

## Install

```sh
composer require strukt/process
```

Requires PHP 8.2+ and `strukt/base`.

## Quick start

```php
$result = process('printf "hello\n"');

echo $result->outputs();
```

Stream complete output lines while the command runs:

```php
$result = process(
    'printf "first\nsecond\n"',
    static function (string $line): void {
        echo "received: {$line}";
    },
);
```

`process()` waits for each command, drains standard output and standard error
concurrently, and returns a `Strukt\ProcessResult`.

## Helpers

| Helper | Parameters | Returns |
| --- | --- | --- |
| `process` | command string or batch, optional callback | `ProcessResult` |
| `pprocess` | command string | `PromptableProcessInterface` |
| `pool` | size, command, cwd, env | `ProcessPool` |
| `supervisor` | size, command, restart cap, backoff, cwd, env | `ProcessSupervisor` |
| `switchChannels` | bool | Swaps logical output/error channels for new processes. |

## Running commands

### One command

```php
$result = process('php -r "echo 2 + 2;"');

echo $result->outputs(); // 4
$process = $result->resource();
```

### Multiple commands

Commands run in array order. Each command has its own process resource and
captured output.

```php
$result = process([
    'printf "compile\n"',
    'printf "test\n"',
]);

echo $result->outputs(0); // compile
echo $result->outputs(1); // test

$first = $result->resource(0);
$second = $result->resource(1);
```

An empty list returns an empty result. Every command must be a non-empty string;
invalid commands throw `InvalidArgumentException`.

### Line callbacks

The callback receives complete logical standard-output lines, including line
endings. A final fragment without a newline is delivered after the process
closes its output stream.

```php
$lines = [];

process('printf "one\ntwo\n"', static function (string $line) use (&$lines): void {
    $lines[] = trim($line);
});

// ['one', 'two']
```

Callbacks may declare no parameters. Standard error is drained but is not sent to
the callback.

## ProcessResult

`Strukt\ProcessResult` extends `ArrayIterator` and may be iterated as a list of
`Strukt\Process` objects.

```php
$result->resource(int $idx = 0): Process
$result->outputs(int $idx = 0): ?string
$result->errors(int $idx = 0): ?string
```

`resource()` throws `OutOfBoundsException` for a missing index. `outputs()` and
`errors()` return `null` when no entry exists and an empty string when the
command produced no output on that channel.

## Error output and exit codes

Both channels are retained.

```php
$result = process('sh -c "printf failure >&2; exit 7"');

echo $result->errors();            // failure
echo $result->resource()->error(); // failure
echo $result->resource()->close(); // 7
```

`Process::run()` throws `RuntimeException` when a command cannot be started.
Invalid constructor handles or empty commands throw `InvalidArgumentException`.

## Switching channels

The default mapping is standard output to `output()` and standard error to
`error()`.

```php
switchChannels(true);

$result = process('sh -c "printf output; printf error >&2"');
echo $result->outputs(); // error
echo $result->errors();  // output

switchChannels(false);
```

The setting affects newly created processes and remains until changed.

## Interactive processes

Use `pprocess()` when a command expects input.

```php
$prompt = pprocess('read password; printf "%s\n" "$password"');

$prompt->write('secret');
$prompt->closeInput();

echo $prompt->read(); // secret
```

`write()` appends `PHP_EOL` and returns the bytes written. Close standard input
when the child must receive EOF.

```php
$prompt->read();
$prompt->error();
$prompt->readline();
$prompt->isRunning();
$prompt->terminate();
$prompt->close();
```

Closing an already-closed pipe is safe. Read output before `close()` if it has
not already been captured.

## Process pool

`ProcessPool` supervises a fixed set of concurrent workers.

```php
$pool = new ProcessPool(3);
$pool->start(PHP_BINARY.' '.escapeshellarg($workerScript));

while ($pool->running()) {
    $pool->drain(static function (int $index, string $line): void {
        printf("worker %d: %s", $index, $line);
    });
    usleep(100000);
}

$pool->stop();
```

Or start one with the helper:

```php
$pool = pool(3, [PHP_BINARY, $workerScript, '--prefix=demo']);
```

`start()` spawns the workers with separate pipes; starting twice throws
`RuntimeException`. `drain()` never blocks on a worker and retains standard
error on each worker. `stop()` terminates and reaps every worker, returning exit
codes keyed by spawn index.

Introspection:

| Method | Behavior |
| --- | --- |
| `count` | Configured pool size. |
| `active` | Workers still running. |
| `running` | True while any worker runs. |
| `processes` | Workers in spawn order. |
| `process($index)` | One worker by spawn index. |

## Process supervisor

`ProcessSupervisor` composes a pool with a restart policy. A dead worker is
replaced by a fresh copy of the original command in the same slot.

```php
$supervisor = new ProcessSupervisor(3, restartCap: 5, backoff: 0.5);

$supervisor->onExit(static function (int $index, int $exitCode): void {
    printf("worker %d exited with %d\n", $index, $exitCode);
});

$supervisor->start(PHP_BINARY.' '.escapeshellarg($workerScript));

while (true) {
    $supervisor->supervise(static function (int $index, string $line): void {
        printf("worker %d: %s", $index, $line);
    });
    usleep(100000);
}
```

Or use the helper:

```php
$supervisor = supervisor(3, [PHP_BINARY, $workerScript], restartCap: 5);
```

Policy:

- `restartCap` bounds restarts per slot. `-1` never gives up.
- `backoff` is the minimum delay before restarting the same slot.
- `onExit()` fires whenever a worker is found dead, whether or not it restarts.

`supervise()` drains output first, then revives dead slots. Introspection includes
`restarts($index)`, `totalRestarts`, and `stop()`.

## Managed supervisor

`Strukt\ManagedSupervisor` runs the supervision loop as a detached daemon and
persists observable state.

```php
use Strukt\ManagedSupervisor;

$supervisor = new ManagedSupervisor('mail', size: 3, restartCap: 5);
$supervisor->start(PHP_BINARY.' '.escapeshellarg($workerScript));
```

Artifacts live under `~/.strukt/supervisor` by default, or
`STRUKT_SUPERVISOR_DIR` / the `$dir` argument:

| Artifact | Purpose |
| --- | --- |
| `<name>/supervise.pid` | Daemon pid. |
| `<name>/state.json` | Snapshot written every supervision pass. |
| `<name>/output.log` | Combined worker output and supervisor events. |

```php
ManagedSupervisor::status('mail');
ManagedSupervisor::running('mail');
ManagedSupervisor::ps();
ManagedSupervisor::tail('mail', 50);
ManagedSupervisor::kill('mail', 0);
ManagedSupervisor::stop('mail');
```

## Supervisor CLI

```bash
php vendor/bin/supervise start mail "php worker.php" --size=2 --dir=/var/run/sv
php vendor/bin/supervise ps
php vendor/bin/supervise status mail
php vendor/bin/supervise kill mail 0
php vendor/bin/supervise logs mail --lines=100
php vendor/bin/supervise stop mail
```

Add `--json` for scripting. Run `php vendor/bin/supervise list` for the command
reference.

## Process API

```php
Process::run(array $commands, ?callable $callback = null): ProcessResult
$process->wait(callable $callback): void
$process->output(): string
$process->poll(?callable $callback = null): bool
Process::switchChannels(bool $switch = true): void
```

`wait()` uses non-blocking pipes and `stream_select()` so large error output
cannot block while the parent waits on standard output. `poll()` drains what is
available without blocking and returns true while the process is running.

`Strukt\Pprocess` extends `Strukt\AbstractPromptableProcess` and adds `write()`
through `PromptableProcessInterface`.

## Resource lifecycle

`Process::run()` closes standard input before waiting, drains both output
channels, dispatches pending output, and reaps the child. Captured strings remain
available through the result and process objects after cleanup.

For manual interactive processes:

1. Call `write()` as needed.
2. Call `closeInput()` when no more input will be sent.
3. Read output and error.
4. Call `close()` or `closeAll()`.

All process objects attempt safe cleanup in `__destruct()`.
