# 04 – Implementation

Follows `03-plan.md`. Everything lives in one file: `server.py` (Python 3.9+, standard library only).

## server.py

```python
import asyncio

HOST, PORT = "127.0.0.1", 6380
store: dict[bytes, bytes] = {}


# ---------- RESP encoders (milestone 3) ----------
def simple(s: str) -> bytes:
    return f"+{s}\r\n".encode()

def error(s: str) -> bytes:
    return f"-ERR {s}\r\n".encode()

def integer(n: int) -> bytes:
    return f":{n}\r\n".encode()

def bulk(b: bytes) -> bytes:
    return b"$%d\r\n%s\r\n" % (len(b), b)

NULL = b"$-1\r\n"


# ---------- RESP reader (milestone 2) ----------
class ProtocolError(Exception):
    pass

async def read_command(reader: asyncio.StreamReader):
    """Return a list of byte args, [] for a blank line, or None on disconnect."""
    line = await reader.readline()
    if not line:
        return None
    if not line.startswith(b"*"):          # inline command, e.g. from nc
        return line.split()
    try:
        count = int(line[1:])
        args = []
        for _ in range(count):
            header = await reader.readline()
            if not header.startswith(b"$"):
                raise ProtocolError
            length = int(header[1:])
            data = await reader.readexactly(length + 2)   # payload + \r\n
            args.append(data[:-2])
        return args
    except (ValueError, ProtocolError):
        raise ProtocolError


# ---------- Commands (milestones 4-5) ----------
def cmd_ping(args):
    return simple("PONG") if not args else bulk(args[0])

def cmd_echo(args):
    return bulk(args[0])

def cmd_set(args):
    store[args[0]] = args[1]
    return simple("OK")

def cmd_get(args):
    value = store.get(args[0])
    return NULL if value is None else bulk(value)

def cmd_del(args):
    return integer(sum(1 for k in args if store.pop(k, None) is not None))

# name: (handler, min_args, max_args)  max None = unlimited
COMMANDS = {
    b"PING": (cmd_ping, 0, 1),
    b"ECHO": (cmd_echo, 1, 1),
    b"SET":  (cmd_set, 2, 2),
    b"GET":  (cmd_get, 1, 1),
    b"DEL":  (cmd_del, 1, None),
}


# ---------- Dispatch + errors (milestone 6) ----------
def execute(args) -> bytes:
    name = args[0].upper()
    entry = COMMANDS.get(name)
    if entry is None:
        return error(f"unknown command '{args[0].decode(errors='replace')}'")
    handler, lo, hi = entry
    n = len(args) - 1
    if n < lo or (hi is not None and n > hi):
        return error(f"wrong number of arguments for '{name.decode().lower()}' command")
    return handler(args[1:])


# ---------- Connection handling (milestone 1) ----------
async def handle_client(reader, writer):
    try:
        while True:
            args = await read_command(reader)
            if args is None:
                break
            if not args:
                continue
            writer.write(execute(args))
            await writer.drain()
    except ProtocolError:
        writer.write(error("protocol error"))
        await writer.drain()
    except (asyncio.IncompleteReadError, ConnectionError):
        pass
    finally:
        writer.close()


async def main():
    server = await asyncio.start_server(handle_client, HOST, PORT)
    print(f"Listening on {HOST}:{PORT}")
    async with server:
        await server.serve_forever()


if __name__ == "__main__":
    asyncio.run(main())
```

## Run it

```bash
python server.py
```

## Try it

With a real client:
```bash
redis-cli -p 6380
127.0.0.1:6380> PING
PONG
127.0.0.1:6380> SET name claude
OK
127.0.0.1:6380> GET name
"claude"
127.0.0.1:6380> DEL name nothing
(integer) 1
127.0.0.1:6380> GET name
(nil)
```

Without `redis-cli`:
```bash
printf 'PING\r\nSET a 1\r\nGET a\r\n' | nc 127.0.0.1 6380
```

With Python (`pip install redis`):
```python
import redis
r = redis.Redis(port=6380)
r.set("a", "1"); print(r.get("a"))   # b'1'
```

## Smoke test (milestone 7)

Save as `test_server.py` and run while the server is up:

```python
import socket

def send(sock, *parts):
    msg = b"*%d\r\n" % len(parts)
    for p in parts:
        p = p.encode()
        msg += b"$%d\r\n%s\r\n" % (len(p), p)
    sock.sendall(msg)
    return sock.recv(4096)

a = socket.create_connection(("127.0.0.1", 6380))
b = socket.create_connection(("127.0.0.1", 6380))

assert send(a, "PING") == b"+PONG\r\n"
assert send(a, "SET", "a", "1") == b"+OK\r\n"
assert send(b, "GET", "a") == b"$1\r\n1\r\n"          # shared across clients
assert send(a, "GET", "missing") == b"$-1\r\n"
assert send(a, "DEL", "a", "missing") == b":1\r\n"
assert send(a, "FOO").startswith(b"-ERR unknown command")
assert send(a, "GET").startswith(b"-ERR wrong number")
assert send(a, "PING") == b"+PONG\r\n"                # still alive
print("All acceptance tests passed")
```

## Acceptance checklist

- [x] `PING` → `PONG`
- [x] `SET` / `GET` round trip
- [x] `GET` missing key → nil
- [x] `DEL` returns count
- [x] Unknown command returns error, server survives
- [x] Two clients share data

## Known limitations
- Memory only: data is lost on restart.
- Single large pipelined write may need a bigger `recv` buffer in the test script.
- No limits on key/value size.
