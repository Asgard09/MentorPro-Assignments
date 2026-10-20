# System Specification: Mini-Redis

## 1. Overview
Mini-Redis is a simplified, in-memory key-value store server designed using basic TCP networking and a subset of the Redis Serialization Protocol (RESP2).

## 2. Supported Commands
* **`PING [message]`**: Responds with `PONG` or the provided message string.
* **`SET key value`**: Stores a string value associated with a key.
* **`GET key`**: Retrieves the string value associated with a key, or `nil` if it does not exist.
* **`DEL key`**: Removes the key and its value from storage. Returns integer `1` if removed, `0` if key didn't exist.
* **`EXISTS key`**: Checks if a key exists. Returns integer `1` if exists, `0` otherwise.

## 3. Data Format & Protocol
* Communications use TCP sockets over standard RESP2 format:
  * Simple Strings: `+OK\r\n`
  * Errors: `-ERR error message\r\n`
  * Integers: `:1\r\n`
  * Bulk Strings: `$5\r\nhello\r\n` or `$-1\r\n` (Null Bulk String)
  * Arrays: `*2\r\n$3\r\nGET\r\n$4\r\nname\r\n`

## 4. Non-Functional Requirements
* Single-threaded async network I/O.
* In-memory storage using a standard dictionary / hash table.
* Clean separation between parser logic, command handler, and storage engine.