# 02 – Specification

## 1. Overview
A TCP server speaking a subset of **RESP2** (Redis Serialization Protocol) that stores string keys and string values in memory.

## 2. Server
| Item | Value |
|------|-------|
| Host | `127.0.0.1` |
| Port | `6380` (avoids clashing with a real Redis on 6379) |
| Concurrency | Single-threaded `asyncio` event loop |
| Storage | Python `dict[bytes, bytes]` |

## 3. Protocol (RESP2 subset)
**Requests** are arrays of bulk strings:
```
*3\r\n$3\r\nSET\r\n$4\r\nname\r\n$6\r\nclaude\r\n
```
Inline commands (`PING\r\n`) are also accepted for `nc`/telnet testing.

**Responses**
| Type | Prefix | Example |
|------|--------|---------|
| Simple string | `+` | `+OK\r\n` |
| Error | `-` | `-ERR unknown command\r\n` |
| Integer | `:` | `:1\r\n` |
| Bulk string | `$` | `$6\r\nclaude\r\n` |
| Null bulk | `$-1` | `$-1\r\n` |

## 4. Commands
| Command | Arguments | Reply | Notes |
|---------|-----------|-------|-------|
| `PING` | `[message]` | `+PONG` or bulk `message` | |
| `ECHO` | `message` | bulk `message` | |
| `SET` | `key value` | `+OK` | Overwrites existing value |
| `GET` | `key` | bulk value or null | Null if key missing |
| `DEL` | `key [key ...]` | integer count deleted | |

Command names are case-insensitive. Keys and values are case-sensitive bytes.

## 5. Errors
| Condition | Reply |
|-----------|-------|
| Unknown command | `-ERR unknown command 'X'` |
| Wrong argument count | `-ERR wrong number of arguments for 'x' command` |
| Malformed RESP | `-ERR protocol error` then close connection |

## 6. Non-functional requirements
- One client's error must never crash the server.
- Client disconnects are handled silently.
- No third-party dependencies.

## 7. Acceptance tests
1. `PING` → `PONG`
2. `SET a 1` → `OK`; `GET a` → `"1"`
3. `GET missing` → `(nil)`
4. `DEL a missing` → `(integer) 1`
5. `FOO` → error, server stays up
6. Two simultaneous `redis-cli` sessions share the same data
