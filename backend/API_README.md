# RAXC Backend API

Rust backend API server for RAXC smart contract security scanner with on-chain payment verification.

## Features

✅ **On-chain payment verification** — Validates payment before processing  
✅ **Replay attack protection** — Marks payments as used  
✅ **RAG-powered analysis** — Retrieves similar exploits from Qdrant  
✅ **GPT-4o analysis** — Deep vulnerability detection  
✅ **In-memory reports** — Fast report generation and download  

---

## Setup

### 1. Install Dependencies

```bash
cargo build --release
```

### 2. Configure Environment

Copy `.env.example` to `.env` and fill in:

```bash
cp .env.example .env
```

Required variables:
- `OPENAI_API_KEY` — Your OpenAI API key
- `QDRANT_URL` — Qdrant vector database URL
- `QDRANT_COLLECTION` — Collection name (defihacklabs)
- `RPC_URL` — Blockchain RPC endpoint (Initia/Avalanche)
- `VAULT_ADDRESS` — Deployed RaxcCreditVault contract address
- `OPERATOR_PRIVATE_KEY` — Private key with OPERATOR_ROLE

### 3. Run Server

```bash
cargo run --bin api --release
```

Server starts on `http://0.0.0.0:8080`

---

## API Endpoints

### POST `/analyze`

Analyze a smart contract with payment verification.

**Request:**
```json
{
  "contract": "pragma solidity ^0.8.0; contract MyContract { ... }",
  "payment_id": "0xabc123...",
  "tx_hash": "0xdef456...",
  "user": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7",
  "name": "MyContract"
}
```

**Fields:**
- `contract` (required) — Solidity source code
- `payment_id` (required) — Payment ID from `payForAnalysis()` transaction
- `tx_hash` (required) — Transaction hash for logging
- `user` (required) — User address who made the payment
- `name` (optional) — Contract name for report filename

**Response:**
```json
{
  "download_url": "/reports/RAXC_MyContract_2026-04-25_123456.md",
  "vulnerability_found": "Yes",
  "risk_level": "High",
  "vulnerability_type": "Reentrancy",
  "confidence": "High"
}
```

**Errors:**
- `400` — Invalid payment_id format
- `401` — Payment invalid or already used
- `403` — Payment user mismatch
- `500` — Internal server error

### GET `/reports/{filename}`

Download generated markdown report.

**Response:**
```markdown
Content-Type: text/markdown
Content-Disposition: attachment; filename="..."

# RAXC Security Analysis Report
...
```

### GET `/health`

Health check endpoint.

**Response:**
```json
{
  "status": "ok"
}
```

---

## Payment Verification Flow

1. **Frontend** → User calls `vault.payForAnalysis()` → Gets `paymentId`
2. **Frontend** → Sends request to `/analyze` with payment proof
3. **Backend** → Calls `vault.verifyPayment(paymentId)` on-chain
4. **Backend** → Validates payment is unused and user matches
5. **Backend** → Calls `vault.markPaymentUsed(paymentId)` (prevents replay)
6. **Backend** → Runs GPT-4o analysis
7. **Backend** → Returns vulnerability report

**Security:**
- Payment marked as used BEFORE analysis to prevent replay attacks
- User address validated against on-chain payment record
- Operator wallet must have `OPERATOR_ROLE` in vault contract

---

## Deployment

### Docker

```dockerfile
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release --bin api

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates libssl3
COPY --from=builder /app/target/release/api /usr/local/bin/api
CMD ["api"]
```

Build and run:
```bash
docker build -t raxc-api .
docker run -p 8080:8080 --env-file .env raxc-api
```

### Environment Variables in Production

Use secure secret management:
- AWS Secrets Manager
- Google Secret Manager
- HashiCorp Vault

**Never commit `.env` to git!**

---

## Testing

### Test Payment Verification

```bash
# 1. User pays on-chain
cast send $VAULT_ADDRESS \
  "payForAnalysis(uint256,uint256)" \
  50000 8000 \
  --private-key $USER_KEY \
  --rpc-url $RPC_URL

# 2. Get payment ID from logs
cast logs --address $VAULT_ADDRESS --rpc-url $RPC_URL

# 3. Call API
curl -X POST http://localhost:8080/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "contract": "contract Test { ... }",
    "payment_id": "0x...",
    "tx_hash": "0x...",
    "user": "0x..."
  }'
```

---

## Contract ABI

The API uses these contract functions:

```solidity
interface IRaxcVault {
  function verifyPayment(bytes32 paymentId) 
    external view 
    returns (bool isValid, address user, uint256 amount);
  
  function markPaymentUsed(bytes32 paymentId) 
    external;  // OPERATOR_ROLE only
}
```

---

## Monitoring

Logs include:
- Payment verification status
- User addresses and amounts
- Transaction hashes
- Analysis completion times

Example logs:
```
[*] Connected to chain ID: 12345
[*] Operator address: 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7
[*] Vault contract: 0xVaultAddress...
[*] RAXC API server → http://0.0.0.0:8080
[*] Payment verified: 0.2255 USDC from 0xUserAddress...
[*] Payment marked as used: 0xPaymentId...
```

---

## Security Considerations

1. **Operator Key Security** — Store private key in secure vault
2. **RPC Endpoint** — Use reliable provider (Infura, Alchemy, QuickNode)
3. **Rate Limiting** — Add rate limits to prevent abuse
4. **Error Handling** — Don't expose internal errors to clients
5. **HTTPS** — Use reverse proxy (nginx) with SSL in production

---

## Troubleshooting

### "Failed to verify payment on-chain"
- Check RPC_URL is accessible (Initia: `https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz`)
- Verify VAULT_ADDRESS is correct
- Ensure contract is deployed on the chain

### "Transaction failed"
- Operator wallet needs ETH/AVAX/INIT for gas
- Operator must have OPERATOR_ROLE in contract
- Check operator private key is valid

### "Payment user mismatch"
- User address in request must match on-chain payment
- Addresses are case-insensitive but must be checksummed

---

## License

MIT
