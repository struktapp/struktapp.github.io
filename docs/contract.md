Strukt Contract
===

Zero-dependency interfaces for HTTP, authentication, forms, service providers,
caching, and processes. This is the foundational Strukt package: it requires
nothing but PHP, so any other package or application can depend on it without
pulling in a runtime.

## Install

```sh
composer require strukt/contract
```

## Contracts

| Contract | Purpose |
| --- | --- |
| `Strukt\Contract\UserInterface` | An authenticated principal. Nullable throughout; password is write-only. |
| `Strukt\Contract\AuthInterface` | Resolves the current principal. |
| `Strukt\Contract\MiddlewareInterface` | One link in the HTTP pipeline. |
| `Strukt\Contract\FormInterface` | Field definitions, submitted values, and validation state. |
| `Strukt\Contract\ProviderInterface` | Container bindings contributed by a service provider. |
| `Strukt\Contract\Http\RequestInterface` | Inbound method, URI, headers, parameters, files, and user. |
| `Strukt\Contract\Http\ResponseInterface` | Status, headers, body, and send. |
| `Strukt\Contract\Http\SessionInterface` | Per-client state, including flash data. |
| `Strukt\Contract\Http\DownloadInterface` | An attachment or inline payload. |
| `Strukt\Contract\Http\UploadedFileInterface` | A file received through a multipart form. |
| `Strukt\Contract\CacheDriverInterface` | Shared cache driver operations. |
| `Strukt\Contract\ProcessInterface` | Shared process resource operations. |
| `Strukt\Contract\PromptableProcessInterface` | A process that accepts standard input. |

Two rules hold across the HTTP contracts:

- No contract names a concrete class. A middleware returns a
  `ResponseInterface`, never a specific response type.
- Accessors are honest about null. A guest is a valid state.

## Reference implementations

`Strukt\User`, `Strukt\Session`, and `Strukt\Auth` are dependency-free
implementations. They are the default wiring, not the only option.

### User

```php
use Strukt\User;

$user = new User(id: '7', username: 'pitsolu', email: 'pitsolu@gmail.com');

$user->getUsername(); // pitsolu
$user->get('id');     // 7
$user->has('email');  // true
$user->toArray();
```

The password is write-only: it is never returned by a getter, is not addressable
through `get()`, and never appears in `toArray()`.

```php
$user = new User(password: password_hash('secret', PASSWORD_BCRYPT));

$user->hasPassword();            // true
$user->verifyPassword('secret'); // true
$user->verifyPassword('wrong');  // false
$user->get('password', 'leaked'); // leaked, never the real hash
```

### Session

```php
use Strukt\Session;

$session = new Session();

$session->set('theme', 'dark');
$session->get('theme');   // dark
$session->has('theme');   // true

$session->flash('notice', 'saved');
$session->getFlash('notice'); // saved, consumed on read
```

A key holding `null` is still present.

### Auth

`Auth` receives its session by injection rather than resolving a global
container.

```php
use Strukt\Auth;
use Strukt\Session;
use Strukt\User;

$auth = new Auth(new Session());

$auth->guest(); // true
$auth->login(new User(id: '7', username: 'pitsolu'));
$auth->check(); // true
$auth->id();    // 7
$auth->logout();

(new Auth($session))->user()?->getUsername();
```

`login()` rejects a user with neither an id nor a username.

## Middleware

```php
use Strukt\Contract\Http\RequestInterface;
use Strukt\Contract\Http\ResponseInterface;
use Strukt\Contract\MiddlewareInterface;

final class Authenticate implements MiddlewareInterface
{
    public function process(
        RequestInterface $request,
        ResponseInterface $response,
        callable $next,
    ): ResponseInterface {
        if ($request->user() === null) {
            return $response->setStatus(401);
        }

        return $next($request, $response);
    }
}
```

Returning without calling `$next` short-circuits the chain.

## HTTP, cache, and process contracts

The HTTP interfaces are implemented by `strukt/router`'s `Strukt\Http` stack:
`Request`, `Response`, `ResponseBuilder`, `Download`, `UploadedFile`, and the
session stores. The cache contract is implemented by `strukt/cache`; the process
contracts are implemented by `strukt/process`.

Applications and packages should type against the contracts rather than the
concrete classes.
