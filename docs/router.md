Strukt Router
===

A typed HTTP router with parameter matching, a middleware pipeline, reflection
resolved actions, and form validation.

Routes are declared against patterns such as `/user/{id:int}`, matched by a
compiled regex with typed parameter extraction, guarded by middleware, and
dispatched by injecting the request, response, services, or a validated form.

## Install

```sh
composer require strukt/router
```

Requires PHP 8.2+ and the `strukt4` contract, form, cache, and event packages as
needed.

## Quick start

```php
use Strukt\Router\Router;
use Strukt\Contract\Http\RequestInterface;

$router = new Router();

$router->get('/', fn () => 'home');
$router->get('/user/{id:int}', fn (int $id) => "user[{$id}]");

$router->post('/login', fn (
    RequestInterface $request,
    Strukt\Contract\FormInterface $form,
) => $form->value('email'))->form(LoginForm::class);

$response = $router->kernel()->handle($request);
$response->send();
```

An application normally uses the shared instance instead:

```php
router()->get('/', fn () => 'home');
```

## Routing

Use one verb method per route, or `any()` for every common method.

```php
$router->get('/list', fn () => 'list');
$router->post('/save', fn () => 'saved');
$router->put('/item/{id:int}', fn (int $id) => "put {$id}");
$router->patch('/item/{id:int}', fn (int $id) => "patch {$id}");
$router->delete('/item/{id:int}', fn (int $id) => 'deleted');
$router->options('/item', fn () => 'options');
$router->head('/item', fn () => 'head');
$router->any('/hook', fn () => 'any method');
```

### Groups

```php
$router->group('/api/v1', function (Router $router): void {
    $router->get('/users', fn () => 'users');
    $router->get('/users/{id:int}', fn (int $id) => "user[{$id}]");
});
```

### Named routes

```php
$router->get('/user/{id:int}', fn (int $id) => "user[{$id}]")
    ->name('user.show');

$router->path('user.show', ['id' => 7]); // /user/7
```

Route options mutate the registered route fluently: `name()`, `middleware()`,
`permissions()`, `requiresAuth()`, `form()`, and `defaults()`.

## Parameters

| Pattern | Matches | Type |
| --- | --- | --- |
| `{name}` | any segment | string |
| `{id:int}` | digits | int |
| `{price:float}` | decimal | float |
| `{on:bool}` | `true` / `false` | bool |
| `{word:alpha}` | letters | string |
| `{key:alphanum}` | letters, digits, `_`, `-` | string |
| `{slug}` | slug words | string |
| `{on:date}` | `YYYY-MM-DD` | string |
| `{id:uuid}` | UUID | string |
| `{rest:*}` | remaining segments | string |

```php
$router->get('/user/{id:int}', fn (int $id) => "user[{$id}]");
$router->get('/blog/{year:int}/{?month:int}', fn (int $year, ?int $month = null) => '');
$router->get('/files/{rest:*}', fn (string $rest) => $rest);
```

Matched values are cast. An unknown type or missing required placeholder throws.
Reverse generation through `uri()` or `Router::path()` respects the same rules
and route defaults.

## Middleware

Global middleware runs before every route; route middleware runs after that.
Definitions may be a registered name, class, object, or callable.

```php
$router->middleware(TraceMiddleware::class);
$router->get('/admin', fn () => 'admin')->middleware('auth', 'authz');
```

Built-in names:

| Name | Class | Purpose |
| --- | --- | --- |
| `sess` | `Middleware\Session` | Attach the session store to the request. |
| `auth` | `Middleware\Authentication` | Resolve and bind the current user. |
| `authz` | `Middleware\Authorization` | Enforce required permissions. |

Middleware resolves through constructor injection. A middleware class declares
the services it needs, and the router supplies them from the per-request service
map.

## Authentication and permissions

```php
use Strukt\Contract\Http\SessionInterface;
use Strukt\Contract\UserInterface;

$router->session(fn (): SessionInterface => new Strukt\Http\Session\Native());

$router->verify(fn (SessionInterface $session): ?UserInterface =>
    $session->get('email')
        ? new Strukt\User(email: $session->get('email'))
        : null
);

$router->permissions(fn (?UserInterface $user): array =>
    $user?->getEmail() === 'admin@example.com' ? ['admin'] : []
);

$router->get('/dashboard', fn (RequestInterface $request) =>
    $request->user()?->getEmail() ?? 'guest'
)->requiresAuth();

$router->get('/admin', fn () => 'admin')
    ->requiresAuth()
    ->permissions(['admin']);
```

A missing user on an authenticated route stops with 401. A user without the
required permission receives 403.

## Forms

Attach a `strukt/form` class with `form()`. The router submits the request values
and validates before the action runs. A failing validation answers with 422 and
the errors as JSON.

```php
$router->post('/login', fn (
    Strukt\Contract\FormInterface $form,
) => 'hi '.$form->value('email'))->form(LoginForm::class);
```

## Responses

An action may return a string (body), an array (JSON), a `ResponseInterface`, or
`null`.

```php
$router->get('/home', fn () => response()->redirect('/home'));
$router->get('/data', fn () => response()->json(['ok' => true]));
$router->get('/file', fn () => download('/tmp/report.pdf', 'report.pdf'));
```

The kernel maps exceptions to HTTP errors: 404 for an unknown path, 405 with an
`Allow` header for a bad method, 422 for failed form validation, and 401/403 for
the auth gates.

## Helpers

| Helper | Purpose |
| --- | --- |
| `router()` | The shared `Router` instance. |
| `request($input, $method, $uri, $headers)` | Build a `Request`. |
| `response($code, $headers)` | A fluent response builder. |
| `route($path, $method, $input)` | Dispatch and return the response. |
| `matcher($patterns)` | A standalone `UrlMatcher`. |
| `redirect($url, $status = 302)` | A redirect response. |
| `download($path, $filename)` | A file download. |

## HTTP layer

The package ships its own `Strukt\Http` stack implementing the `strukt/contract`
interfaces: `Request` with `createFromGlobals()`, `Response`, `ResponseBuilder`,
`Download`, and `UploadedFile`.

Session stores live under `Strukt\Http\Session`:

```php
$router->session(fn (): SessionInterface => new Strukt\Http\Session\Native());
$router->session(fn (): SessionInterface => new Strukt\Http\Session\Cache(memc('sess')));
```

`Native` persists in `$_SESSION`. `Cache` persists through a `strukt/cache`
driver, so several processes or servers can share a session.

## Module prefixes

In a Strukt application, module route prefixes are resolved in this order:

1. Module class property or method: `public string $prefix = '/api/v1'` or
   `public function prefix(): string`.
2. Router folder `Prefix.php` class: `prefix()` method, `$prefix` property, or
   `getPrefix()`.
3. Class-level annotation: `@Prefix('/api/v1')`.

```php
use Strukt\Framework\Contract\Module;

class PayrollAuthModule implements Module
{
    public string $prefix = '/api/v1';
}
```

Prefix joining follows `Router::group()` semantics, with the trailing slash
preserved for the root.
