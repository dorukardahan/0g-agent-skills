# 0G Key-Value Store

## Metadata

- **Category**: storage
- **SDK**: `@0glabs/0g-ts-sdk` ^0.8.0, `ethers` ^6.13.0
- **Activation Triggers**: "key-value", "KV store", "structured data", "kv read", "kv write"

## Purpose

Read and write structured key-value data using 0G's KV layer. Uses Batcher for efficient writes and
KvClient for reads. Built on top of the Log layer for persistence.

## Prerequisites

- Node.js >= 18
- `@0glabs/0g-ts-sdk` and `ethers` installed
- Funded wallet with 0G tokens (for writes)
- `.env` with `PRIVATE_KEY`, `RPC_URL`, `STORAGE_INDEXER`, `KV_INDEXER`

## Quick Workflow

### Write

1. Create ethers wallet and Indexer
2. Create a `Batcher` with stream ID, Indexer, and wallet
3. Set key-value pairs using `batcher.set(key, value)`
4. Execute batch with `batcher.exec()`

### Read

1. Create a `KvClient` with KV Indexer URL
2. Call `kvClient.getValue(streamId, key)`
3. Decode the returned bytes

## Core Rules

### ALWAYS

- Encode keys and values as `Uint8Array` (use `TextEncoder`)
- Use `Batcher` for write operations (efficient batching)
- Use `KvClient` for read operations
- Decode returned values with `TextDecoder`

### NEVER

- Pass string keys/values directly (must be `Uint8Array`)
- Mix up stream IDs between different data sets
- Forget to call `batcher.exec()` after setting values

## Code Examples

### Write Key-Value Pairs

```typescript
import { Indexer, Batcher } from '@0glabs/0g-ts-sdk';
import { ethers } from 'ethers';
import 'dotenv/config';

async function writeKV(streamId: number, key: string, value: string): Promise<string> {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
  const indexer = new Indexer(process.env.STORAGE_INDEXER!);

  const batcher = new Batcher(streamId, indexer, wallet);

  const encodedKey = new TextEncoder().encode(key);
  const encodedValue = new TextEncoder().encode(value);
  batcher.set(encodedKey, encodedValue);

  const tx = await batcher.exec();
  console.log('Write tx:', tx);
  return tx;
}

// Usage
await writeKV(1, 'user:123', JSON.stringify({ name: 'Alice', score: 100 }));
```

### Read Key-Value Pairs

```typescript
import { KvClient } from '@0glabs/0g-ts-sdk';
import 'dotenv/config';

async function readKV(streamId: string, key: string): Promise<string | null> {
  const kvClient = new KvClient(process.env.KV_INDEXER!);

  const encodedKey = new TextEncoder().encode(key);
  const value = await kvClient.getValue(streamId, encodedKey);

  if (!value) return null;
  return new TextDecoder().decode(value);
}

// Usage
const data = await readKV('0x...', 'user:123');
if (data) {
  const user = JSON.parse(data);
  console.log('User:', user.name);
}
```

### Batch Write Multiple Keys

```typescript
async function batchWrite(
  streamId: number,
  entries: Array<{ key: string; value: string }>,
): Promise<string> {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
  const indexer = new Indexer(process.env.STORAGE_INDEXER!);

  const batcher = new Batcher(streamId, indexer, wallet);

  for (const { key, value } of entries) {
    batcher.set(new TextEncoder().encode(key), new TextEncoder().encode(value));
  }

  const tx = await batcher.exec();
  console.log(`Wrote ${entries.length} entries, tx: ${tx}`);
  return tx;
}

// Usage
await batchWrite(1, [
  { key: 'config:theme', value: 'dark' },
  { key: 'config:lang', value: 'en' },
  { key: 'config:version', value: '2.0' },
]);
```

## Anti-Patterns

```typescript
// BAD: Passing strings directly instead of Uint8Array
batcher.set('my-key', 'my-value'); // TypeError

// BAD: Forgetting to exec the batch
batcher.set(key, value);
// Missing: await batcher.exec(); — nothing is written!

// BAD: Using wrong indexer for reads vs writes
const batcher = new Batcher(1, kvIndexer, wallet); // Wrong! Use storage indexer
```

## Common Errors & Fixes

| Error                   | Cause                                   | Fix                                              |
| ----------------------- | --------------------------------------- | ------------------------------------------------ |
| `TypeError`             | Passing strings instead of `Uint8Array` | Use `new TextEncoder().encode()`                 |
| `key not found`         | Key doesn't exist or wrong stream ID    | Verify stream ID and key name                    |
| `insufficient funds`    | Wallet empty (writes cost gas)          | Fund wallet with 0G                              |
| `indexer not available` | Wrong indexer URL                       | Check `KV_INDEXER` / `STORAGE_INDEXER` in `.env` |

## Related Skills

- [Upload File](../upload-file/SKILL.md) — raw file storage
- [Download File](../download-file/SKILL.md) — retrieve raw files
- [Storage + Chain](../../cross-layer/storage-plus-chain/SKILL.md) — on-chain references

## References

- [Storage Patterns](../../../patterns/STORAGE.md)
- [Network Config](../../../patterns/NETWORK_CONFIG.md)
- [0G Storage SDK Docs](https://docs.0g.ai/build-with-0g/storage-network/sdk)
