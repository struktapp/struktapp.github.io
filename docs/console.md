Strukt Console
===

Documentation-first PHP CLI commands. A structured PHPDoc block on a command
class becomes routing, argument and option parsing, validation, `--help` output,
command listing, and generated Markdown documentation.

## Install

```sh
composer require strukt/console
```

Create a `console.php` in the consuming project:

```php
<?php

return [
    'commands' => __DIR__.'/src/Command',
    'docs' => __DIR__.'/docs/commands',
];
```

## Defining a command

Implement `Strukt\Console\Runtime\CommandInterface` and describe the command
with the `@command` DSL.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use Strukt\Console\Runtime\CommandInterface;

/**
 * @command user:create
 * @description Creates a new application user.
 * @usage user:create <email> [options]
 *
 * @argument email
 * @type string
 * @required
 * @description The user's email address.
 *
 * @option name
 * @type string
 * @default Anonymous
 * @description User's display name.
 *
 * @option admin
 * @flag
 * @description Create the user as an administrator.
 *
 * @option password
 * @type string
 * @required
 * @description Initial password.
 */
final class CreateUser implements CommandInterface
{
    public function execute(array $arguments, array $options): int
    {
        $email = $arguments['email'];
        $name = $options['name'] ?? 'Anonymous';
        $isAdmin = $options['admin'] ?? false;

        return 0;
    }
}
```

Run it:

```sh
vendor/bin/console user:create john@example.com --name="John Doe" --admin --password=secret
```

## Command specification

| Tag | Purpose |
| --- | --- |
| `@command` | Required public command name. |
| `@description` | Short command description. |
| `@usage` | Optional explicit usage line. |
| `@argument name` | Declare a positional argument. |
| `@option name` | Declare a value option or flag. |
| `@type` | Argument/option type: `string`, `int`, `integer`, `float`, `bool`, `boolean`. |
| `@required` | The argument or option must be supplied. |
| `@default` | Default value. |
| `@enum` | Comma-separated allowed values. |
| `@flag` | Declare a boolean switch. |
| `@short` | Single-character option alias. |

Use one tag per line and plain spaces. The DSL is explicit: arbitrary prose is
not guessed or executed.

## Runtime contract

```php
interface CommandInterface
{
    public function execute(array $arguments, array $options): int;
}
```

Return `0` for success. A non-zero value becomes the process exit status.

## Help, listing, and docs

```sh
vendor/bin/console
vendor/bin/console list
vendor/bin/console user:create --help
vendor/bin/console docs
```

`docs` writes Markdown generated from the same command definitions. The command
block is the single source for CLI behaviour, help, validation, and generated
documentation.

## Design notes

- The PHPDoc DSL is intentionally small and deterministic.
- Types and required fields are validated before execution.
- Command classes are instantiated through the framework container when used in
  a Strukt application, so constructor dependencies can be declared.
- Future extensions include attributes, shell completion, command groups,
  aliases, interactive prompts, and richer output formatting.
