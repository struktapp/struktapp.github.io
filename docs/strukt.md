Strukt
===

The application skeleton for the `strukt4` package set. It ships almost nothing
of its own: a `composer.json` that pulls in `strukt/framework` and a
`post-create-project-cmd` hook that scaffolds a runnable application.

Everything under `app/`, `cfg/`, `templates/`, `lib/`, plus `index.php` and
`.env`, is generated and git-ignored.

## Install

```sh
composer create-project strukt/strukt --prefer-dist
```

The install hook prompts for an application name, then runs:

```sh
vendor/bin/strukt app:make
vendor/bin/strukt app:reload
```

`app:make` scaffolds `app/src/<Name>`, configuration, and templates.
`app:reload` generates `lib/App/Loader.php`, `index.php`, and `.env`.

## Generated layout

```text
.
├── app/
│   └── src/
│       └── {{app}}/
│           ├── User.php
│           └── AuthModule/
│               ├── {{app}}AuthModule.php
│               ├── Controller/User.php
│               ├── Form/User.php
│               ├── Router/Index.php
│               └── Tests/UserTest.php
├── cfg/
│   └── app.ini
├── templates/
├── lib/
│   └── App/
│       └── Loader.php
├── index.php
├── .env
├── composer.json
└── vendor/
```

The generated `AuthModule` uses the alias `au` and ships with `welcome` and
`hello` routes.

## Run it

```sh
vendor/bin/strukt app:exec
# Serving on http://127.0.0.1:8080
```

Visit `http://127.0.0.1:8080/` and `http://127.0.0.1:8080/hello/ada`.

`app:exec` runs the PHP built-in server with `index.php` as the router script.
The `serve_*`, `is_dev`, and `rel_*` defaults live in `.env`.

## Controller facet

`Controller/User.php` is a stub and is not routed by the generated app. Routes
come from classes under `Router/`. Controllers are reached through the module
alias:

```php
return $this->get('au.ctr.User')->find($id);
```

The registrar indexes it as `nr.au.ctr.User`, and `core('au.ctr.User')` resolves
it through the facet provider.

## Common commands

| Command | Purpose |
| --- | --- |
| `app:make [<name>]` | Scaffold an application; prompts for the name when omitted. |
| `app:reload` | Regenerate `lib/App/Loader.php`, `index.php`, and `.env`. |
| `app:exec` | Serve the application with the PHP built-in server. |
| `module:make <app> <module> <alias>` | Add a module to an application. |
| `route:make [<module>] [<name>] [<route>]` | Scaffold or extend a router. |
| `route:ls` | List registered routes. |
| `shell:exec` | Open the interactive console. |

```sh
vendor/bin/strukt list
vendor/bin/strukt help module:make
vendor/bin/strukt app:make payroll
vendor/bin/strukt module:make payroll human_resource hr
vendor/bin/strukt app:reload
vendor/bin/strukt app:exec
```

`shell:exec` starts a PsySH session with the application booted.

## Sessions

The generated `lib/App/Provider/Session.php` binds the native `$_SESSION` store.

```php
use Strukt\Contract\Http\SessionInterface;
use Strukt\Http\Session\Native;

router()->session(static fn (): SessionInterface => new Native());
router()->middleware('sess');
```

To share sessions across processes or servers, use the cache-backed store:

```php
use Strukt\Http\Session\Cache;

router()->session(static fn (): SessionInterface => new Cache(memc('sess')));
```

Give each client its own namespace, such as `memc('sess_'.$sid)`, to keep
sessions isolated.

## Working in the monorepo

The skeleton resolves `strukt/framework` and its siblings through Composer
`path` repositories (`../framework`, `../base`, ...), so it installs against the
working copies next to it.

```sh
composer install
vendor/bin/strukt app:make {{app}}
vendor/bin/strukt app:reload
```

`bin/clear` resets the project to its seed state by deleting every generated
artifact.

## Notes

- `app:make` and `app:reload` must run before module and route generators can
  discover the application.
- Generated scaffolding is driven by the project-owned `templates/` directory.
- The application loads without Composer autoload edits; the framework resolves
  `App\` to `lib/` and first-level directories under `app/src` at runtime.
