---
name: solana-wallet
description: Create and manage Solana wallets — generate keypairs, check balances, request airdrops, and send SOL/SPL tokens.
metadata: {"openclaw":{"emoji":"🔑","requires":{"bins":["solana"]},"install":[{"id":"brew","kind":"brew","formula":"solana-cli","bins":["solana"],"label":"Install Solana CLI (brew)"}]}}
---

# Solana Wallet Skill

Create and manage Solana wallets using the Solana CLI. This skill covers keypair generation, balance checks, airdrops (devnet/testnet), and SOL/SPL token transfers.

## Prerequisites

- `solana` CLI installed (`brew install solana-cli` or [install guide](https://docs.solanalabs.com/cli/install))
- For mainnet operations, the user must configure their own RPC and fund the wallet

## Quick Reference

### Generate a New Wallet

```bash
# Generate a new keypair (saves to default location ~/.config/solana/id.json)
solana-keygen new

# Generate with a specific output file
solana-keygen new --outfile ~/my-wallet.json

# Generate with no passphrase (for automation)
solana-keygen new --no-bip39-passphrase --outfile ~/my-wallet.json

# Show the public key of a keypair
solana-keygen pubkey ~/my-wallet.json
```

### Recover from Seed Phrase

```bash
# Recover a keypair from a seed phrase
solana-keygen recover --outfile ~/recovered-wallet.json
```

### Generate a Vanity Address

```bash
# Generate a keypair with a specific prefix
solana-keygen grind --starts-with ABC:1
```

### Configure the CLI

```bash
# Set the network (devnet for testing, mainnet-beta for real)
solana config set --url https://api.devnet.solana.com
solana config set --url https://api.mainnet-beta.solana.com

# Set the default keypair
solana config set --keypair ~/my-wallet.json

# Show current config
solana config get
```

### Check Balance

```bash
# Check balance of default wallet
solana balance

# Check balance of a specific address
solana balance <PUBKEY>

# Check balance on a specific network
solana balance --url https://api.devnet.solana.com
```

### Request Airdrop (Devnet/Testnet Only)

```bash
# Request 1 SOL airdrop on devnet
solana airdrop 1 --url https://api.devnet.solana.com

# Request airdrop to a specific address
solana airdrop 1 <PUBKEY> --url https://api.devnet.solana.com
```

### Send SOL

```bash
# Send SOL to an address
solana transfer <RECIPIENT_PUBKEY> <AMOUNT_SOL>

# Send with memo
solana transfer <RECIPIENT_PUBKEY> <AMOUNT_SOL> --with-memo "payment for services"

# Send all SOL (minus fees)
solana transfer <RECIPIENT_PUBKEY> ALL
```

### SPL Tokens

```bash
# Create a token account for a specific mint
spl-token create-account <MINT_ADDRESS>

# Check token balance
spl-token balance <MINT_ADDRESS>

# List all token accounts
spl-token accounts

# Transfer SPL tokens
spl-token transfer <MINT_ADDRESS> <AMOUNT> <RECIPIENT_PUBKEY>
```

### Account Info

```bash
# Show account details
solana account <PUBKEY>

# Show recent transactions
solana transaction-history <PUBKEY> --limit 10

# Confirm a transaction
solana confirm <TX_SIGNATURE>
```

## Security Notes

- **Never share private keys or keypair JSON files**
- Store keypair files with restricted permissions: `chmod 600 ~/my-wallet.json`
- For mainnet wallets with significant funds, use a hardware wallet
- Devnet/testnet SOL has no real value — safe for testing
- When generating wallets for the user, always tell them where the keypair file is saved

## Common Workflows

### New Developer Setup
1. Generate a keypair: `solana-keygen new`
2. Set to devnet: `solana config set --url https://api.devnet.solana.com`
3. Airdrop test SOL: `solana airdrop 2`
4. Verify: `solana balance`

### Check a Wallet
1. `solana balance <PUBKEY>`
2. `solana transaction-history <PUBKEY> --limit 5`

### Send a Payment
1. Verify balance: `solana balance`
2. Send: `solana transfer <RECIPIENT> <AMOUNT>`
3. Confirm: `solana confirm <TX_SIG>`

## Explorer Links

- Mainnet: `https://explorer.solana.com/address/<PUBKEY>`
- Devnet: `https://explorer.solana.com/address/<PUBKEY>?cluster=devnet`
- Transaction: `https://explorer.solana.com/tx/<TX_SIG>`
