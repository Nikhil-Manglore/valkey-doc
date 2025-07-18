Atomically transfer a key from a source Valkey instance to a destination Valkey
instance.
On success the key is deleted from the original instance and is guaranteed to
exist in the target instance.

The command is atomic and blocks the two instances for the time required to
transfer the key, at any given time the key will appear to exist in a given
instance or in the other instance, unless a timeout error occurs. In 3.2 and
above, multiple keys can be pipelined in a single call to `MIGRATE` by passing
the empty string ("") as key and adding the `KEYS` clause.

The command internally uses [`DUMP`](dump.md) to generate the serialized version of the key
value, and [`RESTORE`](restore.md) in order to synthesize the key in the target instance.
The source instance acts as a client for the target instance.
If the target instance returns OK to the `RESTORE` command, the source instance
deletes the key using [`DEL`](del.md).

The timeout specifies the maximum idle time in any moment of the communication
with the destination instance in milliseconds.
This means that the operation does not need to be completed within the specified
amount of milliseconds, but that the transfer should make progress without
blocking for more than the specified amount of milliseconds.

`MIGRATE` operates on the database currently selected in the connection.
Before calling `MIGRATE`, you can switch databases using [`SELECT <dbid>`](select.md),
and the command will migrate keys from that database.

`MIGRATE` needs to perform I/O operations and to honor the specified timeout.
When there is an I/O error during the transfer or if the timeout is reached the
operation is aborted and the special error - `IOERR` returned.
When this happens the following two cases are possible:

* The key may be on both the instances.
* The key may be only in the source instance.

It is not possible for the key to get lost in the event of a timeout, but the
client calling `MIGRATE`, in the event of a timeout error, should check if the
key is _also_ present in the target instance and act accordingly.

When any other error is returned (starting with `ERR`) `MIGRATE` guarantees that
the key is still only present in the originating instance (unless a key with the
same name was also _already_ present on the target instance).

If there are no keys to migrate in the source instance `NOKEY` is returned.
Because missing keys are possible in normal conditions, from expiry for example,
`NOKEY` isn't an error.

## Migrating multiple keys with a single command call

`MIGRATE` supports a bulk-migration mode that
uses pipelining in order to migrate multiple keys between instances without
incurring in the round trip time latency and other overheads that there are
when moving each key with a single `MIGRATE` call.

In order to enable this form, the `KEYS` option is used, and the normal *key*
argument is set to an empty string. The actual key names will be provided
after the `KEYS` argument itself, like in the following example:

    MIGRATE 192.168.1.34 6379 "" 0 5000 KEYS key1 key2 key3

When this form is used the `NOKEY` status code is only returned when none
of the keys is present in the instance, otherwise the command is executed, even if
just a single key exists.

## Options

* `COPY` -- Do not remove the key from the local instance.
* `REPLACE` -- Replace existing key on the remote instance.
* `KEYS` -- If the key argument is an empty string, the command will instead migrate all the keys that follow the `KEYS` option (see the above section for more info).
* `AUTH` -- Authenticate with the given password to the remote instance.
* `AUTH2` -- Authenticate with the given username and password pair.
