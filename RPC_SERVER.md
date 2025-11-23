# Sultan L1 RPC Server API Documentation

The Sultan L1 blockchain provides a comprehensive RPC interface for interacting with the network. This document describes the available endpoints and how to use them.

## Base URLs

**Mainnet:**
- RPC: `https://rpc.sultan.network`
- REST: `https://api.sultan.network`

**Testnet:**
- RPC: `https://rpc-testnet.sultan.network`
- REST: `https://api-testnet.sultan.network`

**Local Development:**
- RPC: `http://localhost:26657`
- REST: `http://localhost:1317`

## RPC Endpoints (Port 26657)

### 1. Network Status

Get current network status including block height, validator count, and chain ID.

**Endpoint:** `GET /status`

**Response:**
```json
{
  "height": 12345,
  "validator_count": 100,
  "total_accounts": 50000,
  "chain_id": "sultan-1"
}
```

**Example (cURL):**
```bash
curl https://rpc.sultan.network/status
```

**Example (Rust):**
```rust
use sultan_sdk::SultanSDK;

let sdk = SultanSDK::new_mainnet().await?;
let status = sdk.status().await?;
println!("Block height: {}", status.height);
```

**Example (JavaScript/TypeScript):**
```javascript
const response = await fetch('https://rpc.sultan.network/status');
const status = await response.json();
console.log('Block height:', status.height);
```

**Example (Python):**
```python
import requests
response = requests.get('https://rpc.sultan.network/status')
status = response.json()
print(f"Block height: {status['height']}")
```

---

### 2. Get Balance

Query the SLTN balance for a specific address.

**Endpoint:** `GET /balance/:address`

**Parameters:**
- `address` - Bech32 encoded Sultan address (starts with `sultan1`)

**Response:**
```json
{
  "address": "sultan1abcdef...",
  "balance": 1000000000000,
  "nonce": 42
}
```

Note: Balance is in usltn (micro-SLTN). Divide by 1,000,000 to get SLTN amount.

**Example (cURL):**
```bash
curl https://rpc.sultan.network/balance/sultan1abcdef...
```

**Example (Rust):**
```rust
let balance = sdk.get_balance("sultan1abcdef...").await?;
let sltn_amount = sdk.get_balance_sltn("sultan1abcdef...").await?;
println!("Balance: {} SLTN", sltn_amount);
```

**Example (JavaScript/TypeScript):**
```javascript
const address = 'sultan1abcdef...';
const response = await fetch(`https://rpc.sultan.network/balance/${address}`);
const data = await response.json();
const sltn = data.balance / 1_000_000;
console.log(`Balance: ${sltn} SLTN`);
```

**Example (Python):**
```python
address = 'sultan1abcdef...'
response = requests.get(f'https://rpc.sultan.network/balance/{address}')
data = response.json()
sltn = data['balance'] / 1_000_000
print(f"Balance: {sltn} SLTN")
```

---

### 3. Submit Transaction

Broadcast a signed transaction to the network.

**Endpoint:** `POST /tx`

**Request Body:**
```json
{
  "tx": {
    "sender": "sultan1abcdef...",
    "recipient": "sultan1xyz...",
    "amount": 1000000000,
    "nonce": 42,
    "signature": "0x..."
  }
}
```

**Response:**
```json
{
  "hash": "0xabc123..."
}
```

**Example (cURL):**
```bash
curl -X POST https://rpc.sultan.network/tx \
  -H "Content-Type: application/json" \
  -d '{"tx": {...}}'
```

**Example (Rust):**
```rust
let tx = Transaction { /* ... */ };
let hash = sdk.send_transaction(tx).await?;
println!("Transaction hash: {}", hash);
```

**Example (JavaScript/TypeScript):**
```javascript
const tx = { /* transaction data */ };
const response = await fetch('https://rpc.sultan.network/tx', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ tx })
});
const result = await response.json();
console.log('TX hash:', result.hash);
```

---

### 4. Get Block

Query block data by height.

**Endpoint:** `GET /block/:height`

**Parameters:**
- `height` - Block height (integer)

**Response:**
```json
{
  "height": 12345,
  "hash": "0xabc123...",
  "timestamp": 1234567890,
  "transactions": [...]
}
```

**Example (cURL):**
```bash
curl https://rpc.sultan.network/block/12345
```

**Example (JavaScript/TypeScript):**
```javascript
const height = 12345;
const response = await fetch(`https://rpc.sultan.network/block/${height}`);
const block = await response.json();
console.log('Block hash:', block.hash);
```

---

## Cosmos SDK REST API (Port 1317)

Sultan L1 also exposes the standard Cosmos SDK REST API at `https://api.sultan.network` (port 1317). This provides access to all standard Cosmos modules:

- `/cosmos/bank/v1beta1/balances/{address}` - Query account balances
- `/cosmos/staking/v1beta1/validators` - List all validators
- `/cosmos/staking/v1beta1/delegations/{delegatorAddr}` - Query delegations
- `/cosmos/tx/v1beta1/txs` - Submit and query transactions
- `/cosmos/auth/v1beta1/accounts/{address}` - Query account info

For full Cosmos REST API documentation, see: https://docs.cosmos.network/api

---

## Network Information

**Chain ID:** `sultan-1` (mainnet), `sultan-testnet-1` (testnet)

**Bech32 Prefix:** `sultan`

**Coin Type:** `118` (standard Cosmos)

**Token Decimals:** `6` (1 SLTN = 1,000,000 usltn)

**Minimum Validator Stake:** `10,000 SLTN` (10,000,000,000 usltn)

**Validator APY:** `26.67%` (fixed)

**Delegator APY:** `10%`

**Gas Fees:** `$0.00` (zero fees)

---

## CORS and Rate Limiting

**CORS:** Enabled for all origins to support browser-based applications.

**Rate Limiting:** 
- Public endpoints: 100 requests per minute per IP
- Authenticated endpoints: 1000 requests per minute

For higher rate limits, please contact the Sultan team or run your own node.

---

## Running Your Own Node

To run your own Sultan L1 node with RPC server:

```bash
# Clone the repository
git clone https://github.com/your-org/sultan

# Build the node
cargo build --release

# Initialize the node
./target/release/sultand init

# Start the node with RPC server
./target/release/sultand start --rpc.laddr tcp://0.0.0.0:26657
```

The RPC server will be available at `http://localhost:26657`.

---

## Support

For questions, issues, or feature requests:
- GitHub: https://github.com/Wollnbergen/BUILD
- Discord: https://discord.gg/sultan
- Documentation: https://docs.sultan.network

---

## Example: Building a Wallet

Here's a complete example of building a simple wallet using the Sultan SDK:

```rust
use sultan_sdk::{SultanSDK, Transaction};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Initialize SDK
    let sdk = SultanSDK::new_mainnet().await?;
    
    // Check balance
    let address = "sultan1abcdef...";
    let balance = sdk.get_balance_sltn(address).await?;
    println!("Balance: {} SLTN", balance);
    
    // Send transaction
    let tx = Transaction {
        sender: address.to_string(),
        recipient: "sultan1xyz...".to_string(),
        amount: 1_000_000_000, // 1000 SLTN
        nonce: 1,
        signature: vec![], // Sign with your private key
    };
    
    let hash = sdk.send_transaction(tx).await?;
    println!("Transaction submitted: {}", hash);
    
    // Become a validator
    let stake_hash = sdk.stake(
        "MyValidator",
        10_000_000_000_000, // 10,000 SLTN
        0.05 // 5% commission
    ).await?;
    println!("Validator created: {}", stake_hash);
    
    Ok(())
}
```

---

## License

MIT License - See LICENSE file for details.
