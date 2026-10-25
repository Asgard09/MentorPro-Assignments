# 03 – Plan

## Approach
Build bottom-up in small, testable steps. Each milestone should run on its own.

## Milestones
| # | Milestone | Deliverable | Done when |
|---|-----------|-------------|-----------|
| 1 | Project skeleton | `server.py` with asyncio TCP listener on 6380 | Server starts, accepts and closes connections |
| 2 | RESP reader | `read_command()` parsing arrays + inline | Prints parsed args for each request |
| 3 | RESP writers | Encoder helpers (`simple`, `error`, `integer`, `bulk`, `null`) | Output bytes match the spec table |
| 4 | Command dispatch | `execute(args)` with `PING`, `ECHO` | `redis-cli PING` returns `PONG` |
| 5 | Storage commands | `SET`, `GET`, `DEL` on a dict | Acceptance tests 2–4 pass |
| 6 | Error handling | Unknown command, arity checks, protocol errors | Acceptance test 5 passes |
| 7 | Verification | Smoke-test script, multi-client check | All 6 acceptance tests pass |

## Design decisions
- **asyncio, single thread:** no locks needed because commands run one at a time, as in real Redis.
- **Bytes everywhere:** avoids encoding bugs; Redis values are binary-safe.
- **Dispatch table (dict of functions):** adding a command later is one line.

