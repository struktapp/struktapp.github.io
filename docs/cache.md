Strukt Cache
===

Typed cache storage with a process-local memory driver, JSON filesystem
persistence, and an optional Memcached backend behind one fluent API.

## Install

```sh
composer require strukt/cache
```

Requires PHP 8.2+. The package uses `strukt/base` for helpers and environment
values, `strukt/commons` for aliases and configuration, and `strukt/fs` for
filesystem operations.

## Quick start

The `cache()` helper opens an in-memory namespace by default.

```php
cache('settings.theme', 'dark');
echo cache('settings.theme'); // dark
```

Use the fluent facade for explicit control:

```php
use Strukt\Cache\Cache;
use Strukt\Cache\Driver\Memory;

$cache = Cache::make('settings', Memory::class);
$cache->put('theme', 'dark');

echo $cache->get('theme');
```

Persistent caches must be saved:

```php
use Strukt\Cache\Cache;
use Strukt\Cache\Driver\Fs;

$cache = new Cache('settings', Fs::class);
$cache->put('theme', 'dark')->save();
```

## Drivers

| Driver | Class | Storage |
| --- | --- | --- |
| Memory | `Strukt\Cache\Driver\Memory` | Current PHP process. |
| Filesystem | `Strukt\Cache\Driver\Fs` | One JSON file per namespace. |
| Memcached | `Strukt\Cache\Driver\MemCached` | A Memcached server. |

All drivers implement `Strukt\Contract\CacheDriverInterface`.

Short driver names are `mem`/`memory`, `fs`, and `memc`/`memcached`.

## The cache helper

```php
cache(string $filename, mixed $value = null, string $driver = Memory::class): mixed
```

With no dot in the name, `cache()` returns a `Cache` facade:

```php
$cache = cache('settings');
$cache->put('theme', 'dark');
```

With a dot, the first segment is the namespace and the remainder is a dotted
key:

```php
cache('settings.theme', 'dark', 'fs');
$theme = cache('settings.theme', null, 'fs');
```

A `null` value means read. Use the facade when `null` must be stored
explicitly. `memc('sessions')` opens a Memcached namespace.

## Facade API

```php
use Strukt\Cache\Cache;

$cache = new Cache('settings');          // defaults to the filesystem driver
$cache = Cache::make('settings', 'mem'); // late-static constructor

$cache->put('user.name', 'Ada');
$cache->get('user.name');
$cache->exists('user.name');
$cache->remove('user.name');
$cache->empty();
$cache->driver(); // CacheDriverInterface
$cache->save();
```

Keys must have non-empty segments. Empty keys, leading/trailing dots, and
consecutive dots throw `InvalidArgumentException`. A stored `null` still counts
as existing.

## Memory

Memory values are shared by namespace name within the current process.

```php
use Strukt\Cache\Driver\Memory;

$first = new Memory('runtime');
$first->put('user.id', 42);

$second = new Memory('runtime');
echo $second->get('user.id'); // 42

Memory::reset(); // clear every process-local namespace
```

`save()` is a no-op for memory.

## Filesystem

The filesystem driver stores a complete namespace as pretty JSON and writes
atomically through a temporary file.

```php
use Strukt\Cache\Driver\Fs;

$cache = new Fs('settings');
$cache->put('theme', 'dark')->save();

$cache = new Fs('settings', sys_get_temp_dir().'/my-cache');
$cache->path(); // the JSON file path
```

The default directory is the application's `.cache` folder. Names cannot contain
directory separators. Invalid JSON is rejected when the driver opens the file;
encoding and write failures surface as exceptions.

## Memcached

```php
use Strukt\Cache\Driver\MemCached;

$cache = new MemCached('settings', [
    'host' => '127.0.0.1',
    'port' => 11211,
    'ttl' => 3600,
]);

$cache->put('theme', 'dark')->save();
$cache->isSuccess();
```

Options may instead come from the environment keys `memcached_host`,
`memcached_port`, `memcached_ttl`, `memcached_username`, and
`memcached_password`. The PHP `memcached` extension is only required when this
driver is instantiated.

## Configuration

```php
config('cache.default', 'fs');    // short name or driver class
config('cache.namespace', 'app'); // default namespace
config('cache.disable', true);    // force memory storage
```

Custom drivers can be bound on the shared container:

```php
app()->set('cache.redis', fn () => RedisDriver::class);

$cache = new Cache('sessions', 'redis');
```

The default service is `Memory` with namespace `default`.

## Lifecycle

1. Create or open a namespace.
2. Read and mutate with `exists()`, `get()`, `put()`, and `remove()`.
3. Call `save()` for filesystem or Memcached storage.
4. Reopen the namespace to confirm persisted values.

Memory changes are visible immediately; filesystem and Memcached changes remain
local to the driver until `save()`.
