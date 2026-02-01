---
name: solana-wallet
description: Create and manage Solana wallets — generate keypairs, check balances, request airdrops, and send SOL/SPL tokens using JavaScript.
metadata: {"openclaw":{"emoji":"🔑","requires":{"bins":["node"]}}}
---

# Solana Wallet Skill

Create and manage Solana wallets using JavaScript and `@solana/web3.js`. No Solana CLI needed — just Node.js.

## Setup

Install the dependency in your workspace:

```bash
npm init -y 2>/dev/null
npm install @solana/web3.js@1 @solana/spl-token bs58
```

## Generate a New Wallet

```javascript
const { Keypair } = require("@solana/web3.js");
const bs58 = require("bs58");
const fs = require("fs");

const keypair = Keypair.generate();
const pubkey = keypair.publicKey.toBase58();
const secret = bs58.encode(keypair.secretKey);

console.log("Public Key:", pubkey);
console.log("Secret Key (bs58):", secret);

// Save keypair to file (JSON array format, compatible with Solana CLI)
fs.writeFileSync("wallet.json", JSON.stringify(Array.from(keypair.secretKey)));
console.log("Saved to wallet.json");
```

## Load an Existing Wallet

```javascript
const { Keypair } = require("@solana/web3.js");
const fs = require("fs");

// From JSON file
const secret = JSON.parse(fs.readFileSync("wallet.json", "utf8"));
const keypair = Keypair.fromSecretKey(Uint8Array.from(secret));
console.log("Loaded:", keypair.publicKey.toBase58());
```

```javascript
// From bs58 private key string
const bs58 = require("bs58");
const { Keypair } = require("@solana/web3.js");

const keypair = Keypair.fromSecretKey(bs58.decode("YOUR_BS58_PRIVATE_KEY"));
console.log("Loaded:", keypair.publicKey.toBase58());
```

## Check Balance

```javascript
const { Connection, PublicKey, LAMPORTS_PER_SOL } = require("@solana/web3.js");

const connection = new Connection("https://api.devnet.solana.com", "confirmed");
// For mainnet: new Connection("https://api.mainnet-beta.solana.com", "confirmed")

const pubkey = new PublicKey("ADDRESS_HERE");
const balance = await connection.getBalance(pubkey);
console.log(`Balance: ${balance / LAMPORTS_PER_SOL} SOL`);
```

## Request Airdrop (Devnet/Testnet Only)

```javascript
const { Connection, PublicKey, LAMPORTS_PER_SOL } = require("@solana/web3.js");

const connection = new Connection("https://api.devnet.solana.com", "confirmed");
const pubkey = new PublicKey("ADDRESS_HERE");

const sig = await connection.requestAirdrop(pubkey, 1 * LAMPORTS_PER_SOL);
await connection.confirmTransaction(sig);
console.log("Airdrop confirmed:", sig);
```

## Send SOL

```javascript
const {
  Connection, Keypair, PublicKey, Transaction,
  SystemProgram, sendAndConfirmTransaction, LAMPORTS_PER_SOL
} = require("@solana/web3.js");
const fs = require("fs");

const connection = new Connection("https://api.devnet.solana.com", "confirmed");

// Load sender wallet
const secret = JSON.parse(fs.readFileSync("wallet.json", "utf8"));
const sender = Keypair.fromSecretKey(Uint8Array.from(secret));

const recipient = new PublicKey("RECIPIENT_ADDRESS");
const amountSol = 0.1;

const tx = new Transaction().add(
  SystemProgram.transfer({
    fromPubkey: sender.publicKey,
    toPubkey: recipient,
    lamports: amountSol * LAMPORTS_PER_SOL,
  })
);

const sig = await sendAndConfirmTransaction(connection, tx, [sender]);
console.log("Sent!", sig);
```

## Check SPL Token Balances

```javascript
const { Connection, PublicKey } = require("@solana/web3.js");
const { getAccount, getAssociatedTokenAddress, getMint } = require("@solana/spl-token");

const connection = new Connection("https://api.devnet.solana.com", "confirmed");
const wallet = new PublicKey("WALLET_ADDRESS");
const mint = new PublicKey("TOKEN_MINT_ADDRESS");

const ata = await getAssociatedTokenAddress(mint, wallet);
const account = await getAccount(connection, ata);
const mintInfo = await getMint(connection, mint);
const balance = Number(account.amount) / Math.pow(10, mintInfo.decimals);
console.log(`Token balance: ${balance}`);
```

## Send SPL Tokens

```javascript
const { Connection, Keypair, PublicKey } = require("@solana/web3.js");
const { getOrCreateAssociatedTokenAccount, transfer } = require("@solana/spl-token");
const fs = require("fs");

const connection = new Connection("https://api.devnet.solana.com", "confirmed");
const secret = JSON.parse(fs.readFileSync("wallet.json", "utf8"));
const sender = Keypair.fromSecretKey(Uint8Array.from(secret));

const mint = new PublicKey("TOKEN_MINT_ADDRESS");
const recipient = new PublicKey("RECIPIENT_ADDRESS");
const amount = 100; // in smallest unit (e.g., if 6 decimals, 100 = 0.0001 tokens)

const senderAta = await getOrCreateAssociatedTokenAccount(connection, sender, mint, sender.publicKey);
const recipientAta = await getOrCreateAssociatedTokenAccount(connection, sender, mint, recipient);

const sig = await transfer(connection, sender, senderAta.address, recipientAta.address, sender, amount);
console.log("Token transfer:", sig);
```

## Get Recent Transactions

```javascript
const { Connection, PublicKey } = require("@solana/web3.js");

const connection = new Connection("https://api.devnet.solana.com", "confirmed");
const pubkey = new PublicKey("ADDRESS_HERE");

const sigs = await connection.getSignaturesForAddress(pubkey, { limit: 10 });
for (const s of sigs) {
  console.log(`${s.signature} | ${s.confirmationStatus} | ${new Date(s.blockTime * 1000).toISOString()}`);
}
```

## Generate Vanity Address

```javascript
const { Keypair } = require("@solana/web3.js");

const prefix = "SOL"; // desired prefix
let attempts = 0;

while (true) {
  const kp = Keypair.generate();
  attempts++;
  if (kp.publicKey.toBase58().startsWith(prefix)) {
    console.log(`Found after ${attempts} attempts!`);
    console.log("Public Key:", kp.publicKey.toBase58());
    console.log("Secret Key:", JSON.stringify(Array.from(kp.secretKey)));
    break;
  }
  if (attempts % 100000 === 0) console.log(`${attempts} attempts...`);
}
```

## Security Notes

- **Never share private keys or wallet.json files**
- Store wallet files with restricted permissions: `chmod 600 wallet.json`
- For mainnet wallets with significant funds, use a hardware wallet
- Devnet/testnet SOL has no real value — safe for testing
- When generating wallets for the user, always tell them where the keypair file is saved

## Explorer Links

- Mainnet: `https://explorer.solana.com/address/<PUBKEY>`
- Devnet: `https://explorer.solana.com/address/<PUBKEY>?cluster=devnet`
- Transaction: `https://explorer.solana.com/tx/<TX_SIG>`
