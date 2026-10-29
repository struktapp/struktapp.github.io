Strukt Framework
===

Annotation-driven application kernel, facades, and console for the `strukt4`
bundle set. This is the package that unifies the Strukt components under one
application lifecycle.

Most applications use it through the `strukt/strukt` skeleton.

## Install

```sh
composer require strukt/framework
```

Requires PHP 8.2+ and a `strukt4` powered project using base, commons, contract,
router, cache, event, form, rules, fs, generator, and console.

## Quick start

```php
use Strukt\Framework\Application;

$app = Application::make();
$app->register(new AuthModule());
$app->run();
```

`handle()` returns a `ResponseInterface`; `run()` sends it.

## Application scaffolding

The framework publishes `vendor/bin/strukt` and a development-only
`vendor/bin/xhttp` binary.

```sh
vendor/bin/strukt list
vendor/bin/strukt help module:make
vendor/bin/strukt cache:clear
```

```sh
vendor/bin/strukt app:make payroll
vendor/bin/strukt app:reload
vendor/bin/strukt app:exec
# http://127.0.0.1:8080
```

`app:make` scaffolds `app/src/<Name>` and the configuration. `app:reload`
generates `lib/App/Loader.php`, `index.php`, and `.env` without overwriting
existing files. `app:exec` runs the PHP built-in server with `index.php` as the
router script.

Application namespaces resolve through `Strukt\Framework\Autoload`, so no
Composer autoload edits or `composer dump-autoload` are required.

## Modules

A module groups application facets beneath one namespace.

```php
use Strukt\Framework\Contract\Module;

final class AuthModule extends Module
{
    protected string $alias = 'auth';
}
```

Facets are indexed in the `nr` registry as `nr.<alias>.<facet>.<name>` and can
be resolved through `core()`:

```php
core('auth.ctr.Login', ['request' => request()]);
```

## Routers and annotations

Routers carry method annotations that `Application` turns into routes on the
Strukt router.

```php
use Strukt\Framework\Contract\Router;

final class Login extends Router
{
    /**
     * @Route(/auth/login)
     * @Method(POST)
     * @Form(Authentication)
     * @Middleware(sess)
     */
    public function login(\Strukt\Http\Request $request)
    {
        //
    }
}
```

Forms validate through `strukt/form` and `strukt/rules`; a failed form
submission produces a 422 response with field errors.

## Dependency injection

The framework ships a small PSR-11 container, backed by `strukt/base`.

```php
use Strukt\Container\Container;
use Strukt\Framework\Contract\Kernel;

$kernel = container()->get(Kernel::class);
$auth = container()->make(\App\Auth\Login::class);

container()->set(
    \App\Mailer::class,
    fn (Container $c) => new \App\Mailer($c->get(\App\Transport::class)),
);
```

Framework collaborators resolve through the container:

| Identifier | Resolves to |
| --- | --- |
| `Kernel` | `RouterAdapter` over the shared router. |
| `Registrar` | Module discovery, backed by `ModuleStore`. |
| `Configuration` | Cached bundle settings. |
| `ModuleStore` | The shared `reg()` registry. |
| `BundlesRepositoryInterface` | The static `Strukt\Bundle\Bundles`. |

Console commands are created through `ContainerCommandFactory`, so commands may
declare constructor dependencies.

## Configuration

Framework `cfg/*.ini` sections merge into the shared `config` registry, only when
the section is not already set:

| Section | Purpose |
| --- | --- |
| `cmd` | Console command groups. |
| `module` | Module folders (`folders.ctr`, `folders.rtr`, ...). |
| `cache` | Cache flags such as `disable`. |
| `repo` | Installed bundle repositories (`core`). |

Environment values used by the kernel:

| Key | Default |
| --- | --- |
| `root_dir` | Project root. |
| `rel_appsrc` | `app/src` |
| `rel_app_lib` | `lib/App` |
| `rel_mod_ini` | `cfg/module.ini` |
| `rel_loader_sgf` | Built-in loader template override. |
| `serve_host` | `127.0.0.1` |
| `serve_port` | `8080` |
| `serve_ctx` | `.` |
| `is_dev` | `true` |

## Bundles

The framework uses **bundles**, not packages. Bundle metadata is available
through `bundle()` and `repos()`.

```php
repos();              // available bundles
repos('installed');   // installed bundle names
repos('published');   // published bundle names

bundle('core')->get('settings');
bundle('core', 'App:Idx')->get('settings');
bundle('core')->get('requirements');
```

The bundle contract lives under `Strukt\Framework\Contract\Bundle`.

## Console commands

| Command | Description |
| --- | --- |
| `app:make` | Generate application scaffolding. |
| `app:reload` | Generate the application module loader. |
| `app:exec` | Run the application on the PHP built-in server. |
| `cache:clear` | Clear the cache. |
| `cache:make` | Rebuild the application configuration cache. |
| `cache:reset` | Reset the configuration cache. |
| `middleware:make` | Create middleware scaffolding. |
| `provider:make` | Create provider scaffolding. |
| `validator:make` | Create validator scaffolding. |
| `module:make` | Generate an application module. |
| `route:make` | Generate or extend a module router. |
| `route:ls` | List registered routes. |
| `shell:exec` | Start an interactive PsySH session. |
| `sys:ls` | List providers and middlewares. |

Bundle publisher commands:

| Command | Description |
| --- | --- |
| `bundle:make` | Create a bundle scaffold. |
| `bundle:add` | Add files or folders to `bundle.json`. |
| `bundle:export` | Export a bundle. |
| `bundle:ls` | List bundles and their status. |
| `bundle:info` | Show information on a bundle. |

## Helpers

| Helper | Purpose |
| --- | --- |
| `app()` | Resolve from the shared container. |
| `container()` | Return or install the shared container. |
| `core()` | Resolve a module facet by alias. |
| `bundle()` | Read bundle metadata. |
| `repos()` | List configured bundles. |
| `config()` | Read or write configuration values. |
| `router()`, `request()`, `response()` | HTTP helpers from `strukt/router`. |
| `cache()` | Cache facade from `strukt/cache`. |
| `event()`, `cmd()` | Event helpers from `strukt/event`. |

The old package-oriented API (`package()`, `package:*`) has been replaced by
bundles (`bundle()`, `bundle:*`). `repos()` keeps its name for compatibility and
now lists bundles.
