# Raxinia: Smart Contract Security Scanner

<div align="center">

**AI-Powered Security Audits with RAG-Enhanced Vulnerability Detection**

[![Built with Rust](https://img.shields.io/badge/Backend-Rust-orange?style=flat-square)](https://www.rust-lang.org/)
[![Next.js Frontend](https://img.shields.io/badge/Frontend-Next.js-black?style=flat-square)](https://nextjs.org/)
[![Powered by GPT-4o](https://img.shields.io/badge/AI-GPT--4o-green?style=flat-square)](https://openai.com/)
[![Deployed on Initia](https://img.shields.io/badge/Chain-Initia-blue?style=flat-square)](https://initia.xyz/)

</div>

---

## 📖 Overview

Raxinia is a comprehensive smart contract security analysis platform that combines:
- **AI Analysis**: GPT-4o powered vulnerability detection
- **RAG System**: Retrieval-Augmented Generation using 1000+ real exploit patterns from DeFiHackLabs
- **Blockchain Payments**: On-chain payment verification via Initia testnet
- **Modern UI**: Next.js frontend with real-time analysis results

**Traditional audits cost $50K-$200K and take weeks. Raxinia provides instant security analysis for $0.10 per contract.**

---

## 💬 The Problem

DeFi protocols have lost over **$4.1 billion** to smart contract exploits — and the same vulnerability patterns keep repeating year after year.

| Year | Confirmed USD Lost | Trend |
|------|--------------------|-------|
| 2017 | $30,000,000 | Early days |
| 2018 | $140,000,155 | — |
| 2020 | $20,000,000 | — |
| 2021 | $124,365,000 | ↑ DeFi summer |
| 2022 | $205,809,017 | ↑ Bridge attacks |
| 2023 | $443,980,241 | ↑↑ |
| 2024 | $1,386,601,430 | ↑↑↑ |
| 2025 | $1,777,671,071 | ↑↑↑↑ Worst year ever |
| 2026 | $7,655,193 | (Jan–Apr only) |
| **Total** | **$4,136,086,808** | |

**Average loss per exploit: $11.2 million USD**

The losses are accelerating every year. The same vulnerability types — reentrancy, price manipulation, flash loans, access control — appear across hundreds of incidents. **These are preventable if caught before deployment.**

**The root cause:** Developers and auditors lack fast, evidence-based tools to catch vulnerabilities before deployment. Traditional static analysis tools generate too many false positives and miss novel attack patterns. LLMs alone hallucinate and lack grounding in real exploit data.

---

## ✨ The Solution

Raxinia is a **Retrieval-Augmented Generation (RAG) pipeline** that analyzes Solidity smart contracts by comparing them against a database of **626+ real-world DeFi exploits** from DeFiHackLabs.

```
User submits contract
        │
        ▼
 Embed with OpenAI          ← text-embedding-3-small
        │
        ▼
 Semantic Search in Qdrant  ← finds top 5 most similar past exploits
        │
        ▼
 GPT-4o Analysis            ← grounded in real exploit evidence
        │
        ▼
 Structured Security Report ← vulnerability type, risk level, fixed code
```

**The key insight:** When GPT-4o knows that *"this exact pattern was used to drain $439K from CompoundUni in Feb 2024"*, it produces dramatically more accurate and credible reports than generic analysis.

### What Makes Raxinia Different

| Feature | Traditional Tools | Pure GPT-4o | **Raxinia** |
|---------|------------------|------------|-------------|
| Real exploit references | ❌ | ❌ | ✅ |
| Evidence-grounded analysis | ❌ | ❌ | ✅ |
| Similarity scoring | ❌ | ❌ | ✅ |
| Fixed code output | ❌ | Sometimes | ✅ Always |
| Covers novel patterns | ❌ | Sometimes | ✅ |
| Cost per analysis | $$$$ | $ | $ |

---

## 🏗️ Repository Structure

```
Raxinia/
├── contracts/          # Solidity smart contracts (Foundry)
│   ├── src/
│   │   └── RaxcCreditVault.sol    # ERC4626 payment vault
│   ├── script/                     # Deployment scripts
│   └── test/                       # Contract tests
│
├── backend/            # Rust API server (Axum)
│   ├── src/
│   │   ├── main.rs                # CLI tools
│   │   ├── api.rs                 # HTTP server
│   │   └── lib.rs                 # Core analysis logic
│   └── .env                       # Environment variables
│
└── frontend/           # Next.js web application
    ├── app/                       # Pages and layouts
    ├── components/                # React components
    └── lib/                       # Utilities and wallet logic
```

---

## 🔧 Components

### 1. Smart Contracts (`/contracts`)

**Technology**: Foundry (Solidity 0.8.27)

**Main Contract**: `RaxcCreditVault.sol`
- ERC4626-compliant vault for credit management
- Pay-per-use model with USDC
- Role-based access control (operator role)
- Payment verification and tracking

**Deployed Addresses** (Initia Testnet):
- **RaxcCreditVault**: `0x4e91C13158D3E0a8F8fBb7cce91712c6F9886690`
- **MockUSDC**: `0x8A2cCc6764063904745BBA687AF11190f7a50461`

**Setup**:
```bash
cd contracts
forge install
forge build
forge test
```

**Deploy**:
```bash
forge script script/DeployInitia.s.sol --rpc-url $RPC_URL --broadcast
```

---

### 2. Backend (`/backend`)

**Technology**: Rust + Axum + ethers-rs + Qdrant

**Features**:
- HTTP API server for contract analysis
- On-chain payment verification
- Vector database integration (Qdrant)
- RAG-powered GPT-4o analysis
- Markdown report generation

**API Endpoints**:
- `POST /analyze` - Analyze contract with payment verification
- `GET /reports/{file}` - Download markdown reports
- `GET /health` - Health check

**Setup**:
```bash
cd backend

# Install Rust (if needed)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Build
cargo build --release

# Run
cargo run --bin api
```

**Environment Variables** (`.env`):
```bash
OPENAI_API_KEY=sk-proj-...
QDRANT_URL=http://localhost:6333
RPC_URL=https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz
VAULT_ADDRESS=0x4e91C13158D3E0a8F8fBb7cce91712c6F9886690
OPERATOR_PRIVATE_KEY=0x...
```

**Dependencies**:
- Requires Qdrant vector database running locally or remotely
- OpenAI API key for GPT-4o access
- Initia RPC access for on-chain verification

---

### 3. Frontend (`/frontend`)

**Technology**: Next.js 14 + React + ethers.js v6

**Features**:
- Wallet connection (MetaMask/Keplr)
- Contract input with syntax highlighting
- Token estimation (GPT-4o)
- Payment flow (approve → pay → analyze)
- Real-time analysis results
- Markdown report rendering with GitHub Flavored Markdown
- Credit balance tracking (vRaxinia shares)

**Setup**:
```bash
cd frontend

# Install dependencies
npm install

# Development
npm run dev

# Production build
npm run build
npm start
```

**Environment Variables** (`.env.local`):
```bash
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_VAULT_ADDRESS=0x4e91C13158D3E0a8F8fBb7cce91712c6F9886690
NEXT_PUBLIC_USDC_ADDRESS=0x8A2cCc6764063904745BBA687AF11190f7a50461
NEXT_PUBLIC_CHAIN_ID=2124225178762456
NEXT_PUBLIC_RPC_URL=https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz
```

---

## 🚀 Quick Start

### Prerequisites
- Node.js 18+ (frontend)
- Rust 1.95+ (backend)
- Qdrant vector database
- Foundry (contracts)
- MetaMask or Keplr wallet

### Step 1: Deploy Contracts (Optional - already deployed)
```bash
cd contracts
forge script script/DeployInitia.s.sol --rpc-url $RPC_URL --broadcast
```

### Step 2: Start Qdrant
```bash
docker run -p 6333:6333 qdrant/qdrant
```

### Step 3: Start Backend
```bash
cd backend
cp .env.example .env  # Configure your env vars
cargo run --bin api
```

### Step 4: Start Frontend
```bash
cd frontend
cp .env.example .env.local  # Configure your env vars
npm install
npm run dev
```

### Step 5: Access Application
Open http://localhost:3000 in your browser

---

## 💳 Payment Flow

1. **Connect Wallet** - Connect MetaMask/Keplr to Initia testnet
2. **Deposit Credits** (Optional) - Deposit USDC to receive vRaxinia shares
3. **Paste Contract** - Input Solidity code for analysis
4. **Estimate Cost** - See token count and USDC cost (~$0.10)
5. **Approve USDC** - One-time approval for vault contract
6. **Pay for Analysis** - Execute payment transaction
7. **Wait for Results** - Analysis completes in 30-60 seconds
8. **View Report** - Comprehensive markdown report with vulnerability findings

---

## 🔍 How It Works

### Analysis Pipeline

1. **Contract Submission** → User submits Solidity code via frontend
2. **Payment Verification** → Backend verifies on-chain payment transaction
3. **Vector Search** → Query Qdrant for similar exploit patterns (top 5 matches)
4. **RAG Enhancement** → Augment prompt with relevant exploit examples
5. **GPT-4o Analysis** → AI analyzes contract with exploit context
6. **Report Generation** → Markdown report with findings, risk level, and recommendations
7. **Result Delivery** → Frontend displays formatted report with tables

### Technology Stack

**Frontend**:
- Next.js 14 (React framework)
- ethers.js v6 (Web3 library)
- react-markdown + remark-gfm (Report rendering)
- js-tiktoken (Token counting)

**Backend**:
- Axum (HTTP framework)
- ethers-rs (Blockchain integration)
- qdrant-client (Vector database)
- openai-rust (GPT-4o API)
- tokio (Async runtime)

**Blockchain**:
- Initia testnet (EVM-compatible L2)
- Solidity 0.8.27
- Foundry (Development framework)

**AI/ML**:
- OpenAI GPT-4o (Analysis engine)
- Qdrant (Vector database)
- RAG pattern (1000+ exploit examples)

---

## 📊 Smart Contract Details

### RaxcCreditVault (ERC4626)

**Key Features**:
- ERC4626-compliant vault for USDC deposits
- Mints vRaxinia shares representing credit balance
- Dynamic pricing based on GPT-4o token costs
- 10% platform fee on all transactions
- Role-based access control (DEFAULT_ADMIN_ROLE, OPERATOR_ROLE)
- Payment tracking to prevent replay attacks

**Main Functions**:
```solidity
// Deposit USDC and receive vRaxinia shares
function deposit(uint256 assets, address receiver) external returns (uint256 shares)

// Pay for analysis (returns unique payment ID)
function payForAnalysis(uint256 promptTokens, uint256 completionTokens) external returns (bytes32 paymentId)

// Estimate cost before payment
function estimateCost(uint256 promptTokens, uint256 completionTokens) external view returns (uint256)

// Operator marks payment as used (prevents reuse)
function markPaymentUsed(bytes32 paymentId) external onlyRole(OPERATOR_ROLE)
```

**Pricing Model**:
- GPT-4o prompt tokens: $2.50 per 1M tokens
- GPT-4o completion tokens: $10.00 per 1M tokens
- Platform fee: 10%
- Typical analysis cost: ~$0.10 USDC

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Frontend (Next.js)                  │
│  • Wallet connection (MetaMask/Keplr)                   │
│  • Token estimation (js-tiktoken)                       │
│  • Payment flow (approve → pay → analyze)               │
│  • Markdown report rendering (remark-gfm)               │
└───────────────────┬─────────────────────────────────────┘
                    │
                    │ HTTP API (REST)
                    │
┌───────────────────▼─────────────────────────────────────┐
│                  Backend (Rust/Axum)                    │
│  • Payment verification (ethers-rs)                     │
│  • Vector search (Qdrant client)                        │
│  • AI analysis (OpenAI GPT-4o)                          │
│  • Report generation (Markdown)                         │
└──────────┬──────────────────┬───────────────────────────┘
           │                  │
           │                  │
    ┌──────▼──────┐    ┌─────▼──────┐
    │   Qdrant    │    │   Initia   │
    │   Vector    │    │  Testnet   │
    │  Database   │    │  (EVM L2)  │
    └─────────────┘    └────────────┘
```

---

## 🔐 Security Features

1. **On-Chain Payment Verification**
   - Backend verifies transaction receipt
   - Checks payment amount and status
   - Prevents replay attacks via payment tracking

2. **Role-Based Access Control**
   - Operator role can mark payments as used
   - Admin role for vault management
   - Separation of concerns

3. **ERC4626 Standard**
   - Industry-standard vault interface
   - Transparent share pricing
   - Auditable state changes

4. **Rate Limiting & Validation**
   - Payment ID uniqueness
   - Transaction status verification
   - Sender address validation

---

## 🚢 Deployment Guide

### Deploy Smart Contracts

```bash
cd contracts

# Install dependencies
forge install

# Set environment variables
export PRIVATE_KEY=0x...
export RPC_URL=https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz

# Deploy
forge script script/DeployInitia.s.sol --rpc-url $RPC_URL --broadcast --verify

# Grant operator role
cast send $VAULT_ADDRESS \
  "grantRole(bytes32,address)" \
  0x97667070c54ef182b0f5858b034beac1b6f3089aa2d3188bb1e8929f4fa9b929 \
  $OPERATOR_ADDRESS \
  --rpc-url $RPC_URL \
  --private-key $PRIVATE_KEY
```

### Deploy Backend (Fly.io Example)

```bash
cd backend

# Install Fly CLI
curl -L https://fly.io/install.sh | sh

# Login
fly auth login

# Create app
fly launch

# Set secrets
fly secrets set OPENAI_API_KEY=sk-proj-...
fly secrets set QDRANT_URL=https://your-qdrant.cloud
fly secrets set OPERATOR_PRIVATE_KEY=0x...
fly secrets set VAULT_ADDRESS=0x4e91...
fly secrets set RPC_URL=https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz

# Deploy
fly deploy
```

### Deploy Frontend (Vercel)

```bash
cd frontend

# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Set environment variables
vercel env add NEXT_PUBLIC_API_URL
vercel env add NEXT_PUBLIC_VAULT_ADDRESS
vercel env add NEXT_PUBLIC_USDC_ADDRESS
vercel env add NEXT_PUBLIC_CHAIN_ID
vercel env add NEXT_PUBLIC_RPC_URL

# Deploy
vercel --prod
```

---

## 🧪 Testing

### Contract Tests
```bash
cd contracts
forge test -vvv
```

### Backend Tests
```bash
cd backend
cargo test
```

### Frontend Tests
```bash
cd frontend
npm test
```

---

## 📝 Environment Variables Reference

### Backend (`.env`)
```bash
# OpenAI
OPENAI_API_KEY=sk-proj-...

# Qdrant Vector Database
QDRANT_URL=http://localhost:6333

# Blockchain
RPC_URL=https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz
VAULT_ADDRESS=0x4e91C13158D3E0a8F8fBb7cce91712c6F9886690
OPERATOR_PRIVATE_KEY=0x1999a8e0fd49ee0051300a376d91b70438f1eeeb25fd761ba728a8b8a8088291
```

### Frontend (`.env.local`)
```bash
# API
NEXT_PUBLIC_API_URL=http://localhost:8080

# Contracts
NEXT_PUBLIC_VAULT_ADDRESS=0x4e91C13158D3E0a8F8fBb7cce91712c6F9886690
NEXT_PUBLIC_USDC_ADDRESS=0x8A2cCc6764063904745BBA687AF11190f7a50461

# Network
NEXT_PUBLIC_CHAIN_ID=2124225178762456
NEXT_PUBLIC_RPC_URL=https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz
```

---

## 📚 API Reference

### POST /analyze

Analyze a Solidity smart contract with payment verification.

**Request Body**:
```json
{
  "contract": "pragma solidity ^0.8.0; contract Example { ... }",
  "payment_id": "0x2efbc23b022b5038e0a791f269c00b7dc4ee14a52feaec0fafcfa1f3d8af7fe6",
  "tx_hash": "0x3343f1...",
  "user": "0x874624ae..."
}
```

**Response**:
```json
{
  "download_url": "/reports/analysis_1234.md",
  "vulnerability_found": true,
  "risk_level": "High",
  "summary": "3 critical vulnerabilities detected",
  "findings": [...]
}
```

### GET /reports/{filename}

Download markdown analysis report.

**Response**: Raw markdown file

### GET /health

Health check endpoint.

**Response**: `200 OK`

---

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **DeFiHackLabs** - Real exploit pattern database
- **OpenAI** - GPT-4o API
- **Initia** - EVM-compatible L2 testnet
- **Qdrant** - Vector database for RAG

---

## 📞 Contact & Support

- **GitHub Issues**: [Report bugs or request features](https://github.com/kongphop/Raxinia/issues)
- **Documentation**: [View full docs](./docs)
- **Community**: Join our discussions

---

## 🗺️ Roadmap

- [ ] Mainnet deployment (Initia)
- [ ] Support for multiple programming languages
- [ ] Real-time analysis during development
- [ ] Browser extension for inline security hints
- [ ] Integration with popular IDEs (VS Code, IntelliJ)
- [ ] Community-contributed exploit patterns
- [ ] Advanced reporting with visual graphs
- [ ] Automated fix suggestions

---

<div align="center">

**Built with ❤️ for the Web3 security community**

</div>
│   Backend API   │  Rust (axum + tokio + ethers.rs)
│   (Rust)        │  • Contract analysis orchestration
└────────┬────────┘  • Transaction verification (7-step)
         │           • OpenAI GPT-4o integration
         │           • Qdrant vector search (RAG)
         │
         ├─────────────────┬─────────────────┐
         │                 │                 │
┌────────▼────────┐ ┌──────▼──────┐ ┌───────▼────────┐
│  Initia Chain   │ │  OpenAI     │ │  Qdrant VDB    │
│  (EVM Layer)    │ │  GPT-4o     │ │  (RAG Store)   │
└─────────────────┘ └─────────────┘ └────────────────┘
  • RaxcCreditVault     • Security       • DeFiHackLabs
  • MockUSDC (test)       analysis         exploit DB
  • Payment logs        • $2.50/1M        • 1000+ cases
```

---

## 🚀 Deployed Contracts (Initia Testnet)

| Contract | Address | Description |
|----------|---------|-------------|
| **RaxcCreditVault** | `0x4e91C13158D3E0a8F8fBb7cce91712c6F9886690` | ERC4626 vault + payment system |
| **MockUSDC** | `0x8A2cCc6764063904745BBA687AF11190f7a50461` | Test USDC (6 decimals) |

**Network**: Initia Anvil  
**Chain ID**: `2124225178762456`  
**RPC**: `https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz`

---

## 📦 Quick Start

### Prerequisites
- Node.js 18+
- Rust 1.70+
- MetaMask or compatible wallet
- OpenAI API key

### 1. Clone Repository
```bash
git clone https://github.com/yourusername/raxinia.git
cd raxinia
```

### 2. Configure Backend
```bash
cd backend
cp .env.example .env

# Edit .env and add:
# OPENAI_API_KEY=sk-your-key-here
# (Qdrant is pre-configured for cloud - no Docker needed!)
```

### 3. Start Backend API
```bash
cargo run
# Runs on http://localhost:8080
```

### 4. Configure Frontend
```bash
cd frontend
cp .env.example .env.local

# Already configured with deployed contract addresses
```

### 5. Start Frontend
```bash
npm install
npm run dev
# Runs on http://localhost:3000
```

### 6. Get Test USDC
```bash
# Transfer from deployer wallet
cast send 0x8A2cCc6764063904745BBA687AF11190f7a50461 \
  "transfer(address,uint256)" \
  YOUR_WALLET_ADDRESS \
  1000000000 \
  --rpc-url https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz \
  --private-key 0x1999a8e0fd49ee0051300a376d91b70438f1eeeb25fd761ba728a8b8a8088291
```

### 7. Add Initia to MetaMask
- **Network Name**: Initia Anvil
- **RPC URL**: `https://jsonrpc-evm-1.anvil.asia-southeast.initia.xyz`
- **Chain ID**: `2124225178762456`
- **Currency Symbol**: INIT

---

## 🎯 Usage

### Pay-per-Analysis Flow

1. **Connect Wallet** - Click "Connect Wallet" button
2. **Paste Contract** - Enter your Solidity code
3. **See Cost** - Preview token count and cost estimate
4. **Approve & Pay** - Authorize USDC spending and pay for analysis
5. **Get Report** - Receive detailed security analysis with vulnerabilities and recommendations

### Credit Vault Flow

1. **Deposit Credits** - Click "Deposit Credits" button
2. **Enter Amount** - Minimum 1 USDC
3. **Approve & Deposit** - Authorize and deposit to vault
4. **Analyze Freely** - Future analyses auto-deduct from your balance
5. **Track Balance** - View your vRAXC shares (vault tokens)

---

## 💡 Pricing

### Token-Based Pricing (GPT-4o)
- **Prompt Tokens**: $2.50 per 1M tokens
- **Completion Tokens**: $10.00 per 1M tokens
- **Platform Fee**: 10% on top
- **USDC Decimals**: 6 (1 USDC = 1,000,000 units)

### Example Costs
| Contract Size | Estimated Cost |
|--------------|----------------|
| Simple (50 lines) | ~$0.02 |
| Medium (200 lines) | ~$0.08 |
| Complex (500 lines) | ~$0.20 |
| Large (1000+ lines) | ~$0.50 |

*Actual costs vary based on contract complexity and response length*

---

## 🛡️ Security Features

### Smart Contract (RaxcCreditVault.sol)
✅ **ERC4626 Standard** - Battle-tested vault implementation  
✅ **Access Control** - Role-based permissions (OpenZeppelin)  
✅ **Replay Protection** - Each payment used only once  
✅ **Minimum Deposit** - 1 USDC minimum prevents dust attacks  
✅ **18 Passing Tests** - Comprehensive test coverage  

### Backend API (Rust)
✅ **7-Step Verification**:
  1. Parse payment ID + transaction hash + user address
  2. Fetch transaction receipt from blockchain
  3. Verify transaction succeeded (status == 1)
  4. Verify sender matches claimed user
  5. Verify recipient is vault contract
  6. Call `verifyPayment()` on-chain
  7. Mark payment as used via `markPaymentUsed()`

✅ **No Double-Spending** - On-chain payment tracking  
✅ **Type-Safe** - Rust's memory safety guarantees  

### Frontend (Next.js + React)
✅ **Accurate Token Counting** - Uses js-tiktoken (OpenAI's official tokenizer)  
✅ **Real-time Cost Estimates** - No surprises  
✅ **Wallet Integration** - Secure MetaMask connection  
✅ **Transaction Confirmation** - Wait for on-chain finality  

---

## 🧪 Testing

### Smart Contracts
```bash
cd contracts
forge test -vv

# Expected output:
# [PASS] testPayForAnalysis()
# [PASS] testVerifyPayment()
# [PASS] testReplayProtection()
# ... 18 tests passing
```

### Backend API
```bash
cd backend
cargo test
```

---

## 📚 Documentation

- **[DEPLOYMENT.md](DEPLOYMENT.md)** - Full deployment guide and contract addresses
- **[MISSION.md](MISSION.md)** - Our vision and the problem we're solving
- **[RAXC_ROADMAP.md](RAXC_ROADMAP.md)** - Feature roadmap and future plans
- **[contracts/README.md](contracts/README.md)** - Smart contract documentation

---

## 🏆 Built For

**Initia Hackathon 2026** 🚀

RAXC demonstrates the power of combining:
- ✅ Initia's EVM-compatible blockchain
- ✅ OpenAI's GPT-4o for advanced reasoning
- ✅ DeFiHackLabs' real exploit dataset
- ✅ Modern Web3 UX (MetaMask + Next.js)
- ✅ Rust's performance and safety

---

## 🛣️ Roadmap

### Phase 1: Core Platform ✅ (Current)
- [x] Smart contract vault system
- [x] Pay-per-analysis model
- [x] Credit vault (ERC4626)
- [x] GPT-4o integration
- [x] RAG with DeFiHackLabs data
- [x] Frontend with wallet connection
- [x] Initia testnet deployment

### Phase 2: Enhanced Analysis (Q2 2026)
- [ ] Multi-file contract support
- [ ] Dependency analysis
- [ ] Gas optimization suggestions
- [ ] Custom rule engine
- [ ] Historical vulnerability tracking

### Phase 3: Developer Tools (Q3 2026)
- [ ] VS Code extension
- [ ] CI/CD integration (GitHub Actions)
- [ ] Hardhat/Foundry plugins
- [ ] API for programmatic access
- [ ] Webhook notifications

### Phase 4: Enterprise Features (Q4 2026)
- [ ] Team collaboration
- [ ] Custom AI model fine-tuning
- [ ] Priority support
- [ ] SLA guarantees
- [ ] White-label solutions

### Phase 5: Multi-Chain Expansion (2027)
- [ ] Ethereum mainnet
- [ ] Avalanche
- [ ] Polygon
- [ ] Arbitrum
- [ ] Custom EVM chains

---

## 🤝 Contributing

We welcome contributions! Areas we need help:

- 🐛 **Bug Reports** - Found an issue? Open a GitHub issue
- 💡 **Feature Requests** - Have an idea? Let us know
- 🔍 **Security Research** - Help us expand our exploit database
- 📖 **Documentation** - Improve guides and tutorials
- 🎨 **UI/UX** - Make the frontend more intuitive

---

## 📄 License

MIT License - see [LICENSE](LICENSE) for details

---

## 🙏 Acknowledgments

- **OpenAI** - GPT-4o API
- **DeFiHackLabs** - Comprehensive exploit database
- **OpenZeppelin** - Battle-tested smart contract libraries
- **Initia** - Fast, scalable blockchain infrastructure
- **Qdrant** - High-performance vector database

---

## 📞 Contact

- **Twitter**: [@RaxcSecurity](https://twitter.com/raxcsecurity)
- **Discord**: [Join our community](https://discord.gg/raxc)
- **Email**: security@raxc.io

---

<div align="center">

**Making Web3 Safer, One Contract at a Time** 🛡️

[Get Started](#-quick-start) • [View Docs](DEPLOYMENT.md) • [Report Bug](https://github.com/yourusername/raxinia/issues)

</div>
