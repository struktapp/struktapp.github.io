Strukt Ar
===

A small query builder and active record ORM for SQL databases. It sits directly
on PDO, parameterizes every statement, and keeps the moving parts small.

`Strukt\Ar` covers both data access and active record. Managed entities with an
identity map and unit of work live in `strukt/orm`.

## Install

```sh
composer require strukt/ar
```

Requires PHP 8.2+ and a PDO driver for your database.

## Connections

`Strukt\Ar\Connection` wraps a PDO instance. Open one through a named constructor
and register it with the manager.

```php
use Strukt\Ar\Connection;

Connection::sqlite(':memory:')->asDefault();
Connection::sqlite('app.db')->as('archive');

db();          // default connection
db('archive'); // named connection
```

`mysql()` and `pgsql()` accept host, credentials, and connection options.
`pdo()` wraps an existing client.

```php
db()->transaction(function (Connection $connection): void {
    $connection->execute(
        'UPDATE accounts SET balance = balance - ? WHERE id = ?',
        [25, 7],
    );
    $connection->execute(
        'UPDATE accounts SET balance = balance + ? WHERE id = ?',
        [25, 9],
    );
});
```

`transaction()` commits on success and rolls back when the callback throws.
Nested calls share the outer transaction.

### Connections from environment files

```env
DB_CONNECTION=sqlite
DB_DATABASE=app.db
```

```php
Connection::fromEnv('.env', '.env.local');
```

Files are read in order, so later files override earlier ones. Keys are read with
`env()`, then the process environment, then defaults: `DB_CONNECTION` (`sqlite`),
`DB_DATABASE` (`database.sqlite`), `DB_HOST` (`127.0.0.1`), `DB_USERNAME`
(`root`), `DB_PASSWORD` (empty), and `DB_PORT` (3306 for MySQL, 5432 for
PostgreSQL).

## Query builder

`table()` starts a fluent, parameterized query against one table.

```php
db()->table('users')
    ->where('active', 1)
    ->where('age', '>', 21)
    ->orderBy('name')
    ->limit(10)
    ->get();
```

`where()` accepts equality shorthand, an explicit operator, or a map of
equalities. Other clauses include `orWhere()`, `whereIn()`, `whereNotIn()`,
`whereNull()`, `whereNotNull()`, `whereBetween()`, `join()`, and `leftJoin()`.

```php
db()->table('posts')
    ->join('users', 'users.id', 'posts.user_id')
    ->select(['users.name', 'posts.title'])
    ->groupBy('users.name')
    ->get();
```

Read rows, values, or aggregates:

```php
db()->table('users')->where('active', 1)->count();
db()->table('users')->sum('balance');
db()->table('users')->pluck('name', 'id');
db()->table('users')->where('id', 7)->value('name');
db()->table('users')->where('email', $email)->exists();
```

`insertGetId()` returns the inserted id. `delete()` refuses to run without a
`where()` clause; `update()`, `increment()`, and `decrement()` touch only
matching rows.

Clauses accumulate on the instance, so clone a base query for variations:

```php
$base = db()->table('users')->where('active', 1);

$admins = (clone $base)->where('admin', 1)->get();
$staff = (clone $base)->where('admin', 0)->get();
```

`toSql()` and `bindings()` expose the compiled statement for debugging.

## Active record

Extend `Strukt\Ar\Model`. One instance represents one row.

```php
use Strukt\Ar\Model;

class User extends Model
{
    protected static array $fillable = ['name', 'email', 'is_admin'];

    public function posts(): array
    {
        return $this->hasMany(Post::class);
    }
}

$sam = User::create(['name' => 'Sam', 'email' => 'sam@strukt.test']);

$sam->name; // Sam
$sam->is_admin = true;
$sam->save();

User::find(1);
User::where('is_admin', 0)->orderBy('name')->get();
User::first();
```

### Table names and keys

The table name is derived from the class short name, pluralized and snake-cased:
`PostComment` maps to `post_comments`.

```php
class User extends Model
{
    protected static ?string $table = 'people';
    protected static string $keyName = 'uuid';
}
```

### Fillable, casts, and timestamps

`fillable` guards mass assignment through `create()`, `fill()`, and `update()`.
Individual columns always accept direct assignment.

`casts` map values entering and leaving the database. Dates become
`DateTimeImmutable` on read and return to `Y-m-d H:i:s` on write; `bool`, `int`,
`float`, `string`, and `array` round-trip their JSON representation.

Timestamps are maintained when `$timestamps` is left at its default `true`.

```php
$sam = User::create([...]);

$sam->created_at;
$sam->updated_at;
```

### Lifecycle

```php
$sam = User::findOrFail(7);

$sam->update(['name' => 'Samantha']);
$sam->refresh();
$sam->delete();   // false when the model does not exist yet

$sam->exists();
$sam->key();
$sam->toArray();
$sam->toJson();
```

`save()` inserts when the model is new and updates otherwise, touching only
changed columns. Reads always go through the query builder, so nothing is cached
at the query layer.

### Relationships

Describe a relationship as a method delegating to `hasMany()`, `hasOne()`, or
`belongsTo()`. Foreign keys are derived from class names and can be overridden.
Results are computed on first call and remembered for the instance.

```php
class Post extends Model
{
    public function author(): ?User
    {
        return $this->belongsTo(User::class);
    }

    public function comments(): array
    {
        return $this->hasMany(Comment::class);
    }
}

$post->author()->name;
$user->posts();
$user->posts()[0]->comments();
```

Model columns are read through `__get`/`__set`. A column that is neither an
attribute nor a declared relationship throws `OutOfBoundsException`, so typos
fail loudly.

## Generating models

`model:generate` introspects a database and writes `Ar\Model` subclasses.

```bash
php bin/dbcli model:generate --database=app.db --driver=sqlite

php bin/dbcli model:generate --database=app.db --driver=sqlite \
    --table=users --namespace=App\Models --dir=src/Models
```

MySQL and PostgreSQL add host and credentials:

```bash
php bin/dbcli model:generate --driver=mysql --database=app \
    --host=127.0.0.1 --user=root --password=secret --dir=src/Models

php bin/dbcli model:generate --driver=pgsql --database=app \
    --host=127.0.0.1 --user=app --dir=src/Models
```

Omit `--database` to read the connection from a dotenv file. `--env` lists files
and defaults to `.env`.

```bash
php bin/dbcli model:generate --dir=src/Models
php bin/dbcli model:generate --env=.env,.env.local
```

The generator derives the class name, pins `$table`, selects a `$keyName` when
the primary key is not `id`, disables `$timestamps` when timestamp columns are
missing, and fills `$casts` and `$fillable`.

When installed as a dependency, the command is available as `vendor/bin/dbcli`.

## Creating the database from models

`db:create` scans a directory for models that declare `$schema` and creates their
tables.

```php
class User extends Model
{
    protected static array $schema = [
        'name' => ['type' => 'string', 'nullable' => false],
        'email' => ['type' => 'string', 'nullable' => false, 'unique' => true],
        'is_admin' => ['type' => 'bool', 'nullable' => false, 'default' => false],
        'birthday' => ['type' => 'datetime', 'nullable' => true],
    ];
}
```

```bash
php bin/dbcli db:create --database=app.db --driver=sqlite --dir=src

php bin/dbcli db:create --database=app.db --driver=sqlite \
    --dir=src --namespace=App\Models
```

Models without `$schema` are skipped. The command is idempotent through
`IF NOT EXISTS`. Types map to SQLite storage classes; MySQL and PostgreSQL use
their native SQL types.

## Managed entities

For an identity map, unit of work, and result caching over these same models and
connection, install `strukt/orm`.

```bash
composer require strukt/orm
```

```php
$sam = orm()->find(User::class, 7);
$sam->name = 'Samantha';
orm()->commit();
```

See the [Orm](Orm.md) page for the full contract.
