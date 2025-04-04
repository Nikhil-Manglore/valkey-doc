﻿---
title: "Hashes"
description: >
    Introduction to Hashes
---

Hashes are record types structured as collections of field-value pairs.
You can use hashes to represent basic objects and to store groupings of counters, among other things.

```
127.0.0.1:6379> HSET bike:1 model Deimos brand Ergonom type 'Enduro bikes' price 4972
(integer) 4
127.0.0.1:6379> HGET bike:1 model
"Deimos"
127.0.0.1:6379> HGET bike:1 price
"4972"
127.0.0.1:6379> HGETALL bike:1
1) "model"
2) "Deimos"
3) "brand"
4) "Ergonom"
5) "type"
6) "Enduro bikes"
7) "price"
8) "4972"
```

While hashes are handy to represent *objects*, actually the number of fields you can
put inside a hash has no practical limits (other than available memory), so you can use
hashes in many different ways inside your application.

The command `HSET` sets multiple fields of the hash, while `HGET` retrieves
a single field. `HMGET` is similar to `HGET` but returns an array of values:

```
127.0.0.1:6379> HMGET bike:1 model price no-such-field
1) "Deimos"
2) "4972"
3) (nil)
```

There are commands that are able to perform operations on individual fields
as well, like `HINCRBY`:

```
127.0.0.1:6379> HINCRBY bike:1 price 100
(integer) 5072
127.0.0.1:6379> HINCRBY bike:1 price -100
(integer) 4972
```

It is worth noting that small hashes (i.e., a few elements with small values) are
encoded in special way in memory that make them very memory efficient.

## Basic commands

* `HSET` sets the value of one or more fields on a hash.
* `HGET` returns the value at a given field.
* `HMGET` returns the values at one or more given fields.
* `HINCRBY` increments the value at a given field by the integer provided.

See the [complete list of hash commands](../commands/#hash).

## Examples

* Store counters for the number of times bike:1 has been ridden, has crashed, or has changed owners:
```
127.0.0.1:6379> HINCRBY bike:1:stats rides 1
(integer) 1
127.0.0.1:6379> HINCRBY bike:1:stats rides 1
(integer) 2
127.0.0.1:6379> HINCRBY bike:1:stats rides 1
(integer) 3
127.0.0.1:6379> HINCRBY bike:1:stats crashes 1
(integer) 1
127.0.0.1:6379> HINCRBY bike:1:stats owners 1
(integer) 1
127.0.0.1:6379> HGET bike:1:stats rides
"3"
127.0.0.1:6379> HMGET bike:1:stats owners crashes
1) "1"
2) "1"
```


## Performance

Most Hash commands are O(1).

A few commands - such as `HKEYS`, `HVALS`, and `HGETALL` - are O(n), where _n_ is the number of field-value pairs.

## Limits

Every hash can store up to 4,294,967,295 (2^32 - 1) field-value pairs.
In practice, your hashes are limited only by the overall memory on the VMs hosting your Valkey deployment.
