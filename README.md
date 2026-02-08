# 0G Agent Skills

AI agent skills for building on the **0G decentralized AI operating system** — Storage, Compute, and
Chain.

Give your AI coding assistant (Claude Code, Cursor, Copilot) complete context for building on 0G.
Ask it to "upload a file to 0G" or "run AI inference on 0G Compute" and get correct, working code.

## What's Inside

**15 skills** across 4 categories:

| Category        | Skills                                                                         | What You Can Build                         |
| --------------- | ------------------------------------------------------------------------------ | ------------------------------------------ |
| **Storage**     | Upload, Download, KV Store, Merkle Verification                                | Decentralized file storage, key-value data |
| **Compute**     | Chat, Image Gen, Speech-to-Text, Fine-Tuning, Provider Discovery, Account Mgmt | AI-powered applications                    |
| **Chain**       | Deploy, Interact, Scaffold                                                     | Smart contracts on 0G Chain                |
| **Cross-Layer** | Storage+Chain, Compute+Storage                                                 | Full-stack dApps                           |

Plus **6 pattern documents** (architecture deep-dives), **3 IDE setup guides**, and full
orchestration via `AGENTS.md`.

## Quick Start

### 1. Install

```bash
git clone https://github.com/0gfoundation/agent-skills-0g .0g-skills
```

### 2. Set Up Your IDE

| IDE             | Setup                                                                                  |
| --------------- | -------------------------------------------------------------------------------------- |
| **Claude Code** | Copy `CLAUDE.md` to project root — auto-detected                                       |
| **Cursor**      | Create `.cursorrules` — see [setup guide](setups/cursor/README.md)                     |
| **Copilot**     | Create `.github/copilot-instructions.md` — see [setup guide](setups/copilot/README.md) |

### 3. Start Building

Ask your AI assistant:

- _"Upload a file to 0G Storage"_
- _"Build a chatbot using 0G Compute"_
- _"Deploy a smart contract to 0G Chain"_
- _"Create an NFT with metadata stored on 0G"_

The assistant will generate correct, working TypeScript code using current SDK versions.

## Repository Structure

```
agent-skills-0g/
├── CLAUDE.md              # Auto-loader for Claude Code
├── AGENTS.md              # Master orchestration (workflows, rules, triggers)
├── skills/
│   ├── storage/           # 4 storage skills
│   ├── compute/           # 6 compute skills
│   ├── chain/             # 3 chain skills
│   └── cross-layer/       # 2 cross-layer skills
├── patterns/              # 6 architecture reference docs
├── setups/                # IDE-specific setup guides
├── INSTALL.md             # Installation guide
└── CONTRIBUTING.md        # Contribution guide
```

## SDKs & Versions

| Package                     | Version | Purpose                          |
| --------------------------- | ------- | -------------------------------- |
| `@0glabs/0g-ts-sdk`         | ^0.8.0  | Storage (upload, download, KV)   |
| `@0glabs/0g-serving-broker` | ^0.6.5  | Compute (inference, fine-tuning) |
| `ethers`                    | ^6.13.0 | Chain interaction (v6 only)      |

## Key Rules

These critical rules are embedded throughout the skills and enforced by `AGENTS.md`:

1. **processResponse()** — Call after every compute inference. Param order:
   `(providerAddress, chatID, usageData)`
2. **ChatID** — Extract from `ZG-Res-Key` header first, body as fallback
3. **evmVersion** — Always `"cancun"` for 0G Chain contracts
4. **ethers v6** — Never use v5 patterns
5. **File handles** — Always close `ZgFile` in `finally` blocks
6. **No hardcoded keys** — Always use `.env`

## Networks

| Network             | RPC Endpoint                   | Chain ID |
| ------------------- | ------------------------------ | -------- |
| Testnet (Galileo)   | `https://evmrpc-testnet.0g.ai` | 16602    |
| Mainnet (Aristotle) | `https://evmrpc.0g.ai`         | 16661    |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on adding new skills.

## License

MIT — see [LICENSE](LICENSE)

## Links

- [0G Documentation](https://docs.0g.ai)
- [0G Storage SDK](https://docs.0g.ai/build-with-0g/storage-network/sdk)
- [0G Compute SDK](https://docs.0g.ai/build-with-0g/compute-network/sdk)
- [0G Chain](https://docs.0g.ai/build-with-0g/0g-chain)
- [Discord](https://discord.gg/0glabs)
