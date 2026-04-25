# RAXC Integration Guide — Initia Hackathon

## Overview

RAXC supports **two payment models** for smart contract security analysis:

1. **Credit Vault** — Users deposit USDC, backend deducts actual cost after analysis
2. **Pay-per-Analysis** — Users pay upfront, API validates payment before processing

---

## Pay-per-Analysis Flow (Recommended for Initia)

Perfect for Initia's Native Features: **auto-signing** or **interwoven-bridge**

### Step 1: Frontend Token Estimation

```typescript
// User pastes their Solidity contract
const contractCode = userInput;

// Count input tokens (use OpenAI tokenizer or estimate ~4 chars = 1 token)
const inputTokens = estimateTokens(contractCode);

// Output tokens fixed at 8000
const outputTokens = 8000;

// Calculate cost using contract's public function
const totalCost = await raxcVault.estimateCost(inputTokens, outputTokens);

console.log(`Analysis will cost: ${totalCost / 1e6} USDC`);
```

### Step 2: User Payment via Initia Native Features

```typescript
// Option A: Auto-signing (gasless transaction)
const paymentTx = await raxcVault.payForAnalysis(inputTokens, outputTokens);
const paymentId = paymentTx.logs[0].topics[1]; // Extract paymentId from event
const txHash = paymentTx.hash;

// Option B: Interwoven-bridge (cross-chain payment)
// User bridges USDC from another chain → pays on Initia
const bridgeTx = await interwovenBridge.bridgeAndPay(
  sourceChain, 
  totalCost, 
  raxcVaultAddress,
  inputTokens,
  outputTokens
);
```

### Step 3: Submit to API with Payment Proof

```typescript
const response = await fetch('https://api.raxc.io/analyze', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    contract: contractCode,
    paymentId: paymentId,
    txHash: txHash,
    user: userAddress
  })
});
```

### Step 4: API Validates Payment

```solidity
// API backend validates payment on-chain
(bool isValid, address user, uint256 amount) = raxcVault.verifyPayment(paymentId);

require(isValid, "Invalid payment");
require(user == requestUser, "User mismatch");

// Mark payment as used
raxcVault.markPaymentUsed(paymentId);

// Now safe to run analysis
runGPT4Analysis(contract);
```

### Step 5: Return Vulnerability Report

```typescript
// API returns analysis results
{
  "paymentId": "0x...",
  "vulnerabilities": [...],
  "riskLevel": "HIGH",
  "exploitReferences": [...],
  "cost": {
    "actualPromptTokens": 15234,
    "actualCompletionTokens": 8000,
    "amountCharged": "0.425000" // USDC
  }
}
```

---

## Smart Contract Functions

### For Users

#### `payForAnalysis(uint256 promptTokens, uint256 completionTokens)`
Pay for a single analysis upfront.

**Returns:** `bytes32 paymentId` — Unique payment identifier

**Usage:**
```solidity
// User approves USDC first
IERC20(usdc).approve(vaultAddress, estimatedCost);

// Pay for analysis
bytes32 paymentId = vault.payForAnalysis(50000, 8000);
```

#### `estimateCost(uint256 promptTokens, uint256 completionTokens)`
Calculate cost before payment.

**Returns:** `uint256 totalCost` — Cost in USDC (6 decimals)

---

### For Backend Operator

#### `verifyPayment(bytes32 paymentId)`
Check if payment is valid and unused.

**Returns:**
- `bool isValid` — True if payment exists and unused
- `address user` — Who made the payment
- `uint256 amount` — Amount paid

#### `markPaymentUsed(bytes32 paymentId)`
Mark payment as consumed after analysis completes.

**Access:** `OPERATOR_ROLE` only

---

### For Admin

#### `withdrawFees(address recipient, uint256 amount)`
Withdraw accumulated platform fees (10% of all costs).

---

## Credit Vault Model (Alternative)

Users can also deposit USDC into the vault for multiple analyses:

```typescript
// User deposits 10 USDC (minimum 1 USDC)
await usdc.approve(vaultAddress, 10_000_000);
await vault.deposit(10_000_000, userAddress);

// Check balance
const balance = await vault.getCreditBalance(userAddress);
console.log(`Credit balance: ${balance / 1e6} USDC`);

// Backend deducts after each analysis
// (no need for user to sign each time)
await vault.deductCost(userAddress, promptTokens, completionTokens);
```

---

## Pricing Formula

```
Actual Cost = (promptTokens × $2.50 / 1M) + (completionTokens × $10.00 / 1M)
User Charge = Actual Cost × 1.10  (10% platform fee)
```

**Examples:**
- 50k prompt + 8k completion = $0.205 → User pays $0.2255 USDC
- 100k prompt + 8k completion = $0.33 → User pays $0.363 USDC
- 200k prompt + 8k completion = $0.58 → User pays $0.638 USDC

---

## Deployment Addresses

### Initia Testnet (Anvil Asia-Southeast)
- **RPC URL:** `https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz`
- **Vault Contract:** `0x...` (TBD)
- **USDC Token:** `0x...` (Initia USDC)

### Avalanche C-Chain
- **Vault Contract:** `0x...` (TBD)
- **USDC Token:** `0xB97EF9Ef8734C71904D8002F8b6Bc66Dd9c48a6E`

---

## Initia Native Features Integration

### Auto-signing
RAXC can integrate with Initia's auto-signing feature so users don't need to manually approve each payment transaction.

```typescript
// Enable auto-signing for RAXC payments
await initiaAutosigning.enableAutoApproval({
  contract: raxcVaultAddress,
  function: 'payForAnalysis',
  maxAmount: '5000000', // Max 5 USDC per auto-approval
  duration: '7d' // Valid for 7 days
});
```

### Interwoven Bridge
Users can pay from other chains without manual bridging:

```typescript
// Pay from Ethereum → Initia in one transaction
await interwovenBridge.crossChainPay({
  sourceChain: 'ethereum',
  targetChain: 'initia',
  targetContract: raxcVaultAddress,
  targetFunction: 'payForAnalysis',
  params: [promptTokens, completionTokens]
});
```

---

## API Endpoints

### POST `/analyze`
Submit contract for analysis with payment proof.

**Request:**
```json
{
  "contract": "pragma solidity ^0.8.0; ...",
  "paymentId": "0xabc123...",
  "txHash": "0xdef456...",
  "user": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb7"
}
```

**Response:**
```json
{
  "paymentId": "0xabc123...",
  "status": "completed",
  "vulnerabilities": [
    {
      "type": "Reentrancy",
      "severity": "CRITICAL",
      "location": "line 45",
      "exploitReference": "Parity Wallet Hack 2017",
      "recommendation": "Use ReentrancyGuard modifier"
    }
  ],
  "riskLevel": "HIGH",
  "cost": {
    "promptTokens": 52341,
    "completionTokens": 8000,
    "totalCharged": "0.231750"
  }
}
```

---

## Security Considerations

1. **Payment Validation:** API MUST verify payment on-chain before processing
2. **Replay Protection:** Each paymentId can only be used once
3. **Operator Security:** Backend operator key should be stored in secure vault (AWS KMS, GCP Secret Manager)
4. **Rate Limiting:** Implement rate limits to prevent spam
5. **Minimum Payment:** While no minimum for pay-per-analysis, deposits require 1 USDC minimum

---

## Testing

Run full test suite:
```bash
forge test --match-contract RaxcCreditVaultTest -vv
```

Test specific payment flow:
```bash
forge test --match-test testPayForAnalysis -vvvv
```

---

## Support

For integration help or questions:
- GitHub Issues: [link]
- Discord: [link]
- Docs: [link]
