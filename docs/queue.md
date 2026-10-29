Strukt Queue
===

A Memcached-backed FIFO message queue with replies. It supports the classic
producer/consumer flow: push a job, pop it, answer with a reply, and wait for the
answer.

The implementation is concurrency-safe: `pop()` is CAS-guarded, so parallel
workers cannot receive the same message twice.

## Requirements

- PHP 8.2+
- `memcached` extension and `libmemcached`
- a running Memcached server, default `127.0.0.1:11211`

## Install

```sh
composer require strukt/queue
```

The package depends on `strukt/base`, `strukt/console`, and `strukt/process`.
The `kqueue` binary is installed into `vendor/bin/`.

## Quick start

```php
use Strukt\Queue;

$queue = new Queue(['prefix' => 'acme']);

$id = $queue->push('jobs', ['user' => 7, 'type' => 'export']);

// in a worker
$job = $queue->pop('jobs', $id);
$queue->reply('jobs', $id, 'exported-42.pdf');

// in the caller
$reply = qwait('jobs', $id, 30);
```

## Features

- FIFO delivery with monotonic, never-reused ids.
- Exactly-once handoff under parallel workers.
- Replies for request/response jobs.
- Namespaces through the `prefix` option.
- Loud failures: constructor throws, failed pushes throw, waits time out.
- Rescue of expired or missing slots.
- Crash redelivery through `Strukt\Queue\Supervisor`.

## Queue API

`Strukt\Queue` is the complete surface.

```php
$queue = new Queue(['prefix' => 'acme']);

$queue->push('jobs', $payload, $ttl = null); // id
$queue->pop('jobs', $id);                    // value or false
$queue->reply('jobs', $id, $reply, $ttl = null);
$queue->getReply('jobs', $id);               // reply or null

$queue->is_empty('jobs');
$queue->is_processed('jobs', $id);
$queue->count('jobs');

$queue->flush('jobs');
$queue->flushAll($prefix = null);
$queue->queues();
$queue->all();
$queue->prefixes();
$queue->client(); // Memcached
$queue->prefix();
```

The constructor options are `prefix`, Memcached `host`, `port`, `ttl`,
`username`, and `password`.

## Helpers

```text
qpush($queue, $item, $options?)       Enqueue and return the id.
qpop($queue, &$id, $options?)         Dequeue, or return false when empty.
qreply($queue, $id, $reply, $options?) Store a reply.
qwait($queue, $id, $seconds?, $options?) Block until a reply arrives.
qempty($queue, $options?)             Whether nothing remains.
qprocessed($queue, $id, $options?)    Whether a message was popped.
```

`qwait()` gives up after its timeout instead of hanging forever.

## CLI

```bash
kqueue push jobs 'my payload'
kqueue pop jobs
kqueue status jobs
kqueue queues
kqueue queues --prefix jobs
kqueue flush jobs
kqueue flush --all --prefix jobs
kqueue flush --all
```

`kqueue list` and `kqueue help <command>` work locally.

## Crash redelivery

`Strukt\Queue\Supervisor` keeps a pool of workers alive with restart cap and
backoff. Messages claimed by a dead worker but never finished are returned to the
queue.

## Operational notes

- Use a distinct prefix per application or environment.
- Use bounded TTLs for jobs that may become stale.
- Treat replies as optional; callers that need them should use `qwait()`.
- Flush queues explicitly in tests and local development.

## Credits

The original queue concept was contributed by Maurizio Giunti
(https://www.mauriziogiunti.it) under the MIT license.
