Strukt Orm
===

Managed entities over `strukt/ar`: an identity map, a unit of work, and result
caching layered on the same `Strukt\Ar\Model` classes and connection.

The objects are the same entities returned by `User::find()`; the manager adds
managed behavior for the objects routed through it.

| | Direct (`strukt/ar`) | Managed (`strukt/orm`) |
| --- | --- | --- |
| Style | Immediate, explicit writes. | Deferred, batched writes. |
| Identity | One object per call. | One object per row. |
| Writes | `save()`/`delete()` run at once. | `persist()`/`remove()` flush on `commit()`. |
| Reads | Query builder per query. | Query builder plus result cache. |
| Use | Low-level control. | Managed entities on a large workload. |

## Install

```sh
composer require strukt/orm
```

Requires PHP 8.2+; `strukt/ar` is pulled in automatically.

## Manager

`orm()` returns the shared manager resolved through the default connection.

```php
use Strukt\Orm\Manager;

Manager::fromEnv('.env', '.env.local');

$sam = orm()->find(User::class, 7);
```

`Manager::for('name')` binds a manager to a named connection.
`Manager::fromEnv()` loads the dotenv files through `Strukt\Ar\Connection` and
then returns the shared manager.

## Identity map and unit of work

`find()` consults the identity map before touching the database, so a row always
loads into the same instance.

```php
$sam = orm()->find(User::class, 7);
$sam->name = 'Samantha'; // nothing written yet

$draft = new User();
$draft->fill(['name' => 'Ada', 'email' => 'ada@strukt.test']);

orm()->persist($draft);
orm()->remove($sam);
orm()->commit(); // one transaction: insert, update, delete
```

An untouched entity is not written: a snapshot taken when it was loaded is
compared at commit time, and only drifted columns are saved.

A failed commit rolls the batch back and rethrows the cause. The manager discards
its tracked work, so reload affected rows with `find()` afterward.

```php
orm()->rollback(); // discard pending work without touching the database
```

Peek at the identity map without a query:

```php
$sam = orm()->find(User::class, 7);

orm()->lookup(User::class, 7);  // same instance, no round trip
orm()->lookup(User::class, 99); // null
```

## Repositories

`query()` returns a `Repository` that proxies the Ar query builder but hydrates
through the identity map, so query results share instances with `find()`.

```php
$admins = orm()->query(User::class)
    ->where('is_admin', 1)
    ->orderBy('name')
    ->get();

$first = orm()->query(User::class)
    ->where('email', $email)
    ->first();
```

Row results are cached by SQL and bindings, so re-running the same query reuses
the rows. The cache lives in the manager and is emptied by `commit()`,
`rollback()`, and `clear()`.

Aggregates such as `count()` and `sum()` always read live.

## Working with both layers

Installing `strukt/orm` brings `strukt/ar` in as a dependency, so both layers use
the same models, tables, and connection. Three behaviours follow:

1. **Ar writes do not invalidate the manager's result cache.** Only `commit()`,
   `rollback()`, and `clear()` flush it. A direct `User::create()` or
   `$user->save()` bypasses the manager, so a repeated query can serve rows that
   predate the write until the next flush. Aggregates still read live.

2. **One row can exist as two objects.** Ar `find()`/`first()` materialize fresh
   instances; only objects routed through `orm()` are identity-mapped. Editing a
   managed entity and then saving it through Ar writes immediately and leaves it
   dirty against its snapshot, so `commit()` issues a redundant update.
   Ar-deleting a tracked entity makes `commit()` try the delete again as a
   harmless zero-row change.

3. **Interleaved Ar writes ride the commit's transaction.** An Ar `save()` between
   `persist()` and `commit()` uses the same connection, so a failed commit rolls
   it back too.

In practice, pick one path per row or workflow. Route managed rows through
`orm()`, use `strukt/ar` directly for one-off or bulk work, and call
`orm()->clear()` after Ar writes that touch rows also tracked by the manager.
Do not `save()` or `delete()` entities you intend to `commit()`.

## Tests

The suite runs against an in-memory SQLite connection using the same schema as
`strukt/ar` and covers the identity map, unit of work, repositories, and result
cache.
