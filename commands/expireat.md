`EXPIREAT` has the same effect and semantic as [`EXPIRE`](expire.md), but instead of
specifying the number of seconds representing the TTL (time to live), it takes
an absolute [Unix timestamp][hewowu] (seconds since January 1, 1970). A
timestamp in the past will delete the key immediately.

[hewowu]: http://en.wikipedia.org/wiki/Unix_time

Please for the specific semantics of the command refer to the documentation of
`EXPIRE`.

## Background

`EXPIREAT` was introduced in order to convert relative timeouts to absolute
timeouts for the AOF persistence mode.
Of course, it can be used directly to specify that a given key should expire at
a given time in the future.

## Options

The `EXPIREAT` command supports a set of options:

* `NX` -- Set expiry only when the key has no expiry
* `XX` -- Set expiry only when the key has an existing expiry
* `GT` -- Set expiry only when the new expiry is greater than current one
* `LT` -- Set expiry only when the new expiry is less than current one

A non-volatile key is treated as an infinite TTL for the purpose of `GT` and `LT`.
The `GT`, `LT` and `NX` options are mutually exclusive.

## Examples

```
127.0.0.1:6379> SET mykey "Hello"
OK
127.0.0.1:6379> EXISTS mykey
(integer) 1
127.0.0.1:6379> EXPIREAT mykey 1293840000
(integer) 1
127.0.0.1:6379> EXISTS mykey
(integer) 0
```
