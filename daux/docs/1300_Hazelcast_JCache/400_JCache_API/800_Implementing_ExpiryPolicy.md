
In JCache, `javax.cache.expiry.ExpiryPolicy` implementations are used to automatically expire cache entries based on different rules.

Expiry timeouts are defined using `javax.cache.expiry.Duration`, which is a pair of `java.util.concurrent.TimeUnit`, that
describes a time unit and a long, defining the timeout value. The minimum allowed `TimeUnit` is `TimeUnit.MILLISECONDS`.
The long value `durationAmount` must be equal or greater than zero. A value of zero (or `Duration.ZERO`) indicates that the
cache entry expires immediately.

By default, JCache delivers a set of predefined expiry strategies in the standard API.

- `AccessedExpiryPolicy`: Expires after a given set of time measured from creation of the cache entry. The expiry timeout is updated on accessing the key.
- `CreatedExpiryPolicy`: Expires after a given set of time measured from creation of the cache entry. The expiry timeout is never updated.
- `EternalExpiryPolicy`: Never expires. This is the default behavior, similar to `ExpiryPolicy` being set to null.
- `ModifiedExpiryPolicy`: Expires after a given set of time measured from creation of the cache entry. The expiry timeout is updated on updating the key.
- `TouchedExpiryPolicy`: Expires after a given set of time measured from creation of the cache entry. The expiry timeout is updated on accessing or updating the key.

Because `EternalExpiryPolicy` does not expire cache entries, it is still possible to evict values from memory if an underlying
`CacheLoader` is defined.

