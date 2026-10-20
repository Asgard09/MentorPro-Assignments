
```python
import asyncio

class RedisDataStore:
    def __init__(self):
        self._store = {}

    def get(self, key: str):
        return self._store.get(key)

    def set(self, key: str, value: str):
        self._store[key] = value
        return "OK"

    def delete(self, key: str) -> int:
        if key in self._store:
            del self._store[key]
            return 1
        return 0

    def exists(self, key: str) -> int:
        return 1 if key in self._store else 0


class RESPParser:
    @staticmethod
    def encode_simple_string(val: str) -> bytes:
        return f"+{val}\r\n".encode()

    @staticmethod
    def encode_bulk_string(val: str) -> bytes:
        if val is None:
            return b"$-1\r\n"
        return f"${len(val)}\r\n{val}\r\n".encode()

    @staticmethod
    def encode_integer(val: int) -> bytes:
        return f":{val}\r\n".encode()

    @staticmethod
    def encode_error(msg: str) -> bytes:
        return f"-ERR {msg}\r\n".encode()


class MiniRedisServer:
    def __init__(self, host="127.0.0.1", port=6379):
        self.host = host
        self.port = port
        self.store = RedisDataStore()

    async def handle_client(self, reader: asyncio.StreamReader, writer: asyncio.StreamWriter):
        while True:
            data = await reader.read(1024)
            if not data:
                break

