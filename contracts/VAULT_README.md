# RAXC Credit Vault

ERC4626 vault contract for the RAXC platform's on-chain credit system. Users deposit USDC to receive vault shares representing their credit balance, which is used to pay for AI-powered smart contract analysis.

## Features

✅ **ERC4626 Standard** - Full compatibility with the vault standard  
✅ **Transparent Pricing** - Costs calculated from actual OpenAI token usage  
✅ **Platform Fee** - 10% fee on top of AI costs (user pays = actual_cost × 1.10)  
✅ **Operator Role** - Backend can deduct costs after each analysis  
✅ **Multi-chain Support** - Deployable on Avalanche, Arbitrum, Optimism, Polygon, Base  
✅ **Fee Collection** - Platform fees accumulate in vault for admin withdrawal  

## Cost Model

Based on GPT-4o pricing:
- **Prompt tokens**: $2.50 per 1M tokens
- **Completion tokens**: $10.00 per 1M tokens
- **Platform fee**: +10% on actual cost

**Example**: Analysis using 100k prompt tokens + 50k completion tokens:
```
Actual cost = (100,000 × $2.50/1M) + (50,000 × $10.00/1M)
            = $0.25 + $0.50 = $0.75

Platform fee = $0.75 × 0.10 = $0.075

User pays = $0.75 + $0.075 = $0.825 USDC
```

## Architecture

```
User deposits USDC → Receives vRAXC vault shares
                    ↓
            Credit balance tracked on-chain
                    ↓
       Backend operator deducts costs after each analysis
                    ↓
              Platform fees accumulate
```

## Usage

### Deployment

**Avalanche C-Chain:**
```bash
forge script script/DeployRaxcVault.s.sol:DeployRaxcVault \
  --sig "deployAvalanche()" \
  --rpc-url $AVALANCHE_RPC_URL \
  --private-key $PRIVATE_KEY \
  --broadcast \
  --verify
```

**Custom chain:**
```bash
# Set environment variables
export USDC_ADDRESS=0x...
export OPERATOR_ADDRESS=0x...
export PRIVATE_KEY=0x...

forge script script/DeployRaxcVault.s.sol:DeployRaxcVault \
  --rpc-url $RPC_URL \
  --broadcast \
  --verify
```

### Contract Interactions

**User deposits USDC:**
```solidity
// Approve vault to spend USDC
IERC20(usdcAddress).approve(vaultAddress, amount);

// Deposit USDC and receive vault shares
vault.deposit(amount, userAddress);

// Check credit balance (in USDC)
uint256 balance = vault.getCreditBalance(userAddress);
```

**Backend deducts cost after analysis:**
```solidity
// Operator deducts based on actual token usage
vault.deductCost(
    userAddress,
    promptTokens,      // From OpenAI API response
    completionTokens   // From OpenAI API response
);
```

**User withdraws unused credits:**
```solidity
// Withdraw USDC (burns vault shares)
vault.withdraw(amount, receiver, owner);
```

**Admin withdraws platform fees:**
```solidity
uint256 fees = vault.getAvailableFees();
vault.withdrawFees(recipient, fees);
```

### Estimating Costs

Users can estimate analysis costs before submitting:

```solidity
// Estimate cost for expected token usage
uint256 estimatedPromptTokens = 50000;   // 50k tokens
uint256 estimatedCompletionTokens = 25000; // 25k tokens

uint256 cost = vault.estimateCost(
    estimatedPromptTokens,
    estimatedCompletionTokens
);
// Returns total cost in USDC (including 10% platform fee)
```

## Testing

Run the test suite:
```bash
forge test -vvv
```

Run specific test:
```bash
forge test --match-test testDeductCost -vvv
```

Run with gas reporting:
```bash
forge test --gas-report
```

## Key Functions

### User Functions
- `deposit(uint256 assets, address receiver)` - Deposit USDC for credits
- `withdraw(uint256 assets, address receiver, address owner)` - Withdraw unused credits
- `getCreditBalance(address user)` - Check credit balance in USDC
- `estimateCost(uint256 promptTokens, uint256 completionTokens)` - Estimate analysis cost

### Operator Functions
- `deductCost(address user, uint256 promptTokens, uint256 completionTokens)` - Deduct cost after analysis

### Admin Functions
- `addOperator(address operator)` - Grant operator role to backend service
- `removeOperator(address operator)` - Revoke operator role
- `withdrawFees(address recipient, uint256 amount)` - Withdraw accumulated platform fees
- `getAvailableFees()` - Check available fees for withdrawal

### View Functions
- `calculateCost(uint256 promptTokens, uint256 completionTokens)` - Calculate cost breakdown
- `convertToAssets(uint256 shares)` - Convert vault shares to USDC amount
- `convertToShares(uint256 assets)` - Convert USDC amount to vault shares

## Security Considerations

1. **Operator Trust** - Backend operator can deduct from user balances; operator address must be secure
2. **Role Management** - Admin role controls operator permissions and fee withdrawals
3. **OpenZeppelin Contracts** - Built on audited OpenZeppelin implementations (ERC4626, AccessControl)
4. **Fee Tracking** - Platform fees tracked separately to prevent over-withdrawal
5. **Precision** - Uses USDC's 6 decimals for accurate micro-transaction pricing

## Upgrade Path (Phase 6)

For trustless operation with zkTLS:

1. Backend generates ZK proof of OpenAI API response using Reclaim Protocol
2. Proof includes prompt_tokens, completion_tokens, and timestamp
3. Contract verifies ZK proof before allowing deduction
4. Eliminates trust requirement for operator role

## Integration Example

Backend integration (TypeScript/Node.js):

```typescript
import { ethers } from 'ethers';

// After OpenAI API call completes
const usage = openaiResponse.usage;
const promptTokens = usage.prompt_tokens;
const completionTokens = usage.completion_tokens;

// Deduct cost from user's vault balance
const tx = await vaultContract.deductCost(
  userAddress,
  promptTokens,
  completionTokens
);

await tx.wait();
console.log(`Deducted cost for user ${userAddress}`);
```

## License

MIT

## Support

For issues or questions, please open an issue on the GitHub repository.
