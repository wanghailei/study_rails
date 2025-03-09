# Solid

## Solid Cable

Solid Cable is a database-backed Action Cable adapter that keeps messages in a table and continously polls for updates. This makes it possible to drop the common dependency on Redis. The performance of Solid Cable is comparable to Redis in most situations. And in all circumstances, it makes it easier to deploy Rails when Redis is no longer a required dependency for Action Cable functionality.

## Solid Cache

Solid Cache replaces the need for either Redis or Memcached for storing HTML fragment caches in particular.It also always for a vastly larger and cheaper cache thanks to its use of disk storage rather than RAM storage. This means your cache can live longer and cover even more requests out the plank of the 95th or 99th percentile.

## Solid Queue

Solid Queue replaces the need for not just Redis, but also a separate job-running framework, like Resque, Delayed Job, or Sidekiq, for most people.