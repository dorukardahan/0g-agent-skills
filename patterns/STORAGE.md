# Storage Patterns

Architecture, SDK reference, and best practices for 0G decentralized storage.

## Architecture Overview

0G Storage is a two-layer system:

1. **Log Layer** — Append-only data availability layer for raw file storage
2. **KV Layer** — Key-value store built on top of the Log layer for structured data

```
┌─────────────────────────────────┐
│          KV Layer               │
│   (Structured key-value data)   │
├─────────────────────────────────┤
│          Log Layer              │
│   (Raw file / blob storage)    │
├─────────────────────────────────┤
│       0G Consensus              │
│   (Data availability proofs)   │
└─────────────────────────────────┘
```

## Log Layer (File Storage)

### Core Concepts

- **ZgFile**: Represents a file in the 0G storage system
- **Merkle Tree**: Every file is split into chunks and organized as a Merkle tree
- **Root Hash**: The Merkle root — unique identifier for the file content
- **Indexer**: Service that locates data across storage nodes

### File Upload Lifecycle

1. Create a `ZgFile` from local file path or buffer
2. Generate the Merkle tree (computes root hash)
3. Upload via the Indexer (distributes chunks to storage nodes)
4. Store the root hash for later retrieval
5. **Close the file handle** (critical — prevents memory leaks)

### Upload Pattern

```typescript
import { ZgFile, Indexer } from '@0glabs/0g-ts-sdk';
import { ethers } from 'ethers';
import 'dotenv/config';

const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);

const indexer = new Indexer(process.env.STORAGE_INDEXER!);
const file = await ZgFile.fromFilePath(filePath);

try {
  const [tree, err] = await file.merkleTree();
  if (err) throw err;

  const rootHash = tree.rootHash();
  console.log('Root hash:', rootHash);

  const tx = await indexer.upload(file, wallet);
  console.log('Upload tx:', tx);

  return rootHash;
} finally {
  await file.close();
}
```

### Download Pattern

```typescript
import { Indexer } from '@0glabs/0g-ts-sdk';

const indexer = new Indexer(process.env.STORAGE_INDEXER!);
await indexer.download(rootHash, outputPath, /* verified */ true);
```

### Upload from Buffer

```typescript
import { ZgFile } from '@0glabs/0g-ts-sdk';
import * as fs from 'fs';
import * as os from 'os';
import * as path from 'path';

// Write buffer to temp file (SDK requires file path)
const tempPath = path.join(os.tmpdir(), `0g-upload-${Date.now()}`);
fs.writeFileSync(tempPath, buffer);

const file = await ZgFile.fromFilePath(tempPath);
try {
  const [tree, err] = await file.merkleTree();
  if (err) throw err;
  const rootHash = tree.rootHash();
  await indexer.upload(file, wallet);
  return rootHash;
} finally {
  await file.close();
  fs.unlinkSync(tempPath); // Clean up temp file
}
```

## KV Layer (Structured Data)

### Core Concepts

- **Stream**: A named data channel (like a database table)
- **StreamKey**: Identifies an entry within a stream
- **Batcher**: Batches multiple write operations for efficiency
- **KvClient**: Reads data from the KV layer

### Write Pattern (Batcher)

```typescript
import { Indexer, KvClient, Batcher } from '@0glabs/0g-ts-sdk';
import { ethers } from 'ethers';

const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
const indexer = new Indexer(process.env.STORAGE_INDEXER!);

// Create batcher for write operations
const batcher = new Batcher(1, indexer, wallet);

// Set key-value pairs
const key = new TextEncoder().encode('my-key');
const value = new TextEncoder().encode(JSON.stringify({ data: 'hello' }));
batcher.set(key, value);

// Execute batch write
const tx = await batcher.exec();
console.log('Batch tx:', tx);
```

### Read Pattern (KvClient)

```typescript
import { KvClient } from '@0glabs/0g-ts-sdk';

const kvClient = new KvClient(process.env.KV_INDEXER!);

const streamId = '0x...'; // Your stream ID
const key = new TextEncoder().encode('my-key');

const value = await kvClient.getValue(streamId, key);
const decoded = new TextDecoder().decode(value);
console.log('Value:', JSON.parse(decoded));
```

## Merkle Verification

### Computing Root Hash

```typescript
import { ZgFile } from '@0glabs/0g-ts-sdk';

const file = await ZgFile.fromFilePath(filePath);
try {
  const [tree, err] = await file.merkleTree();
  if (err) throw err;
  console.log('Root hash:', tree.rootHash());
} finally {
  await file.close();
}
```

### Verified Download

```typescript
// The third parameter enables Merkle proof verification
await indexer.download(rootHash, outputPath, true);
// Throws if data integrity check fails
```

## Critical Rules

### ALWAYS

- Generate the Merkle tree before uploading
- Close file handles after upload (`file.close()`)
- Store root hashes — they are the ONLY way to retrieve files
- Use verified downloads in production (third param = `true`)
- Clean up temp files when uploading from buffers

### NEVER

- Skip Merkle tree generation before upload
- Forget to close `ZgFile` handles (causes memory leaks)
- Lose the root hash (data becomes irretrievable)
- Upload without a funded wallet (transaction will fail)

## Common Errors

| Error                   | Cause                      | Fix                                 |
| ----------------------- | -------------------------- | ----------------------------------- |
| `merkle tree error`     | File is empty or corrupted | Verify file exists and has content  |
| `insufficient funds`    | Wallet has no 0G           | Fund wallet from faucet (testnet)   |
| `indexer not available` | Wrong indexer URL          | Check STORAGE_INDEXER in .env       |
| `file not found`        | Root hash doesn't exist    | Verify root hash is correct         |
| `connection refused`    | RPC endpoint down          | Try alternative RPC or check status |

## References

- [0G Storage SDK Docs](https://docs.0g.ai/build-with-0g/storage-network/sdk)
- [0G Storage Architecture](https://docs.0g.ai/learn/0g-storage)
- See also: [NETWORK_CONFIG.md](./NETWORK_CONFIG.md), [SECURITY.md](./SECURITY.md)
