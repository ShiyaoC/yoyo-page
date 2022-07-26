---
layout: default
---

# Redis Commands Notes


## KEYS pattern
`keys` command returns all keys matching pattern.
For example, we can use 
`keys pcp:*` to search for keys that matches the patter `pcp:`, where '*' is a wildcard.

[redis key command documentation](https://redis.io/commands/keys/)

## TYPE key
`type` returns the string representation of the type of the value stored at key. The different types that can be returned are: string, list, set, zset, hash and stream.

We can use the key returns from the previous command to get its type:
```
type pcp:desc:series:5dea8102b229105e073edd6f8c36d56f9ddb97a8
```

[redis type command documentation](https://redis.io/commands/type/)

## SMEMBER key 
`smembers` returns all the members of the set value stored at key.
[redis smembers command documentatio](https://redis.io/commands/smembers/)

## HGETALL key
`hgetall` returns all fields and values of the hash stored at key. In the returned value, every field name is followed by its value, so the length of the reply is twice the size of the hash.

`hgetall pcp:desc:series:5dea8102b229105e073edd6f8c36d56f9ddb97a8` returns

```
 1) "indom"
 2) "none"
 3) "pmid"
 4) "60.1.50"
 5) "semantics"
 6) "instant"
 7) "source"
 8) "\x9c\x10\x89\x18\xd7[\x90c\xb8\x17\x98\r\xf1\x95-\xf2U|\xf1Y"
 9) "type"
10) "u64"
11) "units"
12) "Kbyte"
```

[redis hgetall command documentatio](https://redis.io/commands/hgetall/)

## FLUSHALL [ ASYNC | SYNC]
Delete all the keys of all the existing databases, not just the currently selected one. This command never fails.
[redis flushall command documentatio](https://redis.io/commands/flushall/)

[back](.././)
