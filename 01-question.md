# 01 – Question

## Problem statement
How do we build the **simplest possible Redis-style feature**: an in-memory key-value store that a real Redis client (`redis-cli`, `redis-py`) can talk to?

## Core questions
1. What is the smallest set of commands that still feels like Redis? 
4. How do we know it works (what is "done")?

## Out of scope
Persistence (RDB/AOF), TTL/`EXPIRE`, data types beyond strings, pub/sub, transactions, replication.

## Success criteria
- `redis-cli -p 6380 SET name Asgard` returns `OK`, and `GET name` returns `"Asgard"`.
- Bad or unknown commands return an error instead of crashing the server.
