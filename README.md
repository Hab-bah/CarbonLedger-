# CarbonLedger-
CarbonLedger tokenizes verified carbon credits as Stellar assets and trades them on Stellar's built-in decentralized exchange (DEX). Every credit has an on-chain provenance trail — project, methodology, verifier, vintage year — making it impossible to double-count, misrepresent, or mysteriously retire. Transparent carbon markets start here.

Table of Contents

Problem Statement
Solution
How It Works
Architecture
Smart Contract Design
Credit Tokenization Standard
Tech Stack
Getting Started
Project Structure
Roadmap
Contributing
License


Problem Statement
The voluntary carbon market (VCM) is broken. A 2023 investigation found that over 90% of Verra's rainforest offset credits were worthless — phantom credits that did not represent real emissions reductions. Yet they were sold, traded, and "retired" as if they were legitimate.
The core failure is structural: carbon credits move through opaque, centralized registries (Verra, Gold Standard, ACR) with no interoperability, no public audit trail, and no technical barrier to double-counting across registries. Corporations claiming "carbon neutral" status cannot prove their credits are real. Legitimate project developers — reforestation farmers, solar co-ops in emerging markets — cannot command premium prices because buyers cannot verify their claims.
A transparent, open-source infrastructure layer would solve this. CarbonLedger is that layer.

Solution
CarbonLedger does three things:

Tokenizes credits: Each verified carbon credit is minted as a Stellar Asset (1 token = 1 tonne CO₂e), with full provenance metadata stored on-chain via Soroban
Trades credits: Buyers and sellers trade on Stellar's native DEX — no custom AMM, no liquidity fragmentation, instant settlement at near-zero fees
Retires credits immutably: When a buyer retires a credit (claims the offset), the token is burned on-chain with an immutable retirement certificate. The same credit cannot be retired twice, ever

CarbonLedger does not replace registries — it creates an open settlement layer that registries, project developers, brokers, and corporations can all plug into.

How It Works
Credit Lifecycle
1. PROJECT REGISTRATION
   Developer registers a carbon project:
   - Project ID, type (Reforestation | Solar | Biochar | etc.)
   - Methodology (Verra VM0007, Gold Standard TPDDTEC, etc.)
   - Verifier address (approved third-party verifier)
   - Estimated annual credits (tCO₂e)

2. VERIFICATION
   Approved verifier reviews project documentation
   → Signs verification transaction on-chain
   → Project status changes to Verified

3. ISSUANCE
   Verifier issues credits for a specific vintage year:
   - Each credit = 1 token of the project's Stellar Asset
   - Token includes: project_id, vintage_year, methodology, verifier
   → Tokens sent to developer's Stellar account

4. TRADING
   Developer lists credits on Stellar DEX:
   - Create offer: N credits at X USDC per credit
   - Buyers find offers via DEX, pay USDC, receive credit tokens
   - Price discovery happens naturally on the open orderbook

5. RETIREMENT
   Corporate buyer retires credits to claim offset:
   → Calls retire() on Soroban contract
   → Credit tokens burned, immutable retirement record created
   → Certificate generated: who retired, how many, for what entity, when
   → Cannot be undone. Cannot be reused.

6. REPORTING
   Any company, regulator, or journalist can:
   → Query all credits from a project
   → Verify retirement certificates on-chain
   → Confirm no double-counting across retirements
Preventing Double-Counting
The double-counting problem arises when the same credit is counted in two places simultaneously. CarbonLedger prevents this at the protocol level:

A credit token can only exist once (Stellar's asset model enforces this)
Retirement burns the token — it ceases to exist in any account
Retirement is on-chain and irreversible — no central registry can "un-retire" it
All retirements are public and queryable — any party can verify


Architecture
┌────────────────────────────────────────────────────────────────┐
│                    CarbonLedger Platform                       │
│                                                                │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐  │
│  │  Developer      │  │  Verifier       │  │  Buyer /     │  │
│  │  Portal         │  │  Dashboard      │  │  Corporate   │  │
│  │                 │  │                 │  │  Portal      │  │
│  │  - Register     │  │  - Review docs  │  │              │  │
│  │    project      │  │  - Issue credits│  │  - Browse    │  │
│  │  - View credit  │  │  - Flag issues  │  │    projects  │  │
│  │    inventory    │  │                 │  │  - Buy on    │  │
│  │  - List on DEX  │  │                 │  │    DEX       │  │
│  │  - View sales   │  │                 │  │  - Retire    │  │
│  └────────┬────────┘  └────────┬────────┘  └──────┬───────┘  │
│           │                    │                   │          │
│           └──────────────┬─────┘                   │          │
│                          │ Stellar JS SDK            │          │
└──────────────────────────┼────────────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────────────┐
│                   Stellar / Soroban Layer                      │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   Soroban Contracts                     │  │
│  │                                                         │  │
│  │  registry.rs       Project + verifier registry          │  │
│  │  issuance.rs       Credit token minting logic           │  │
│  │  retirement.rs     Irreversible credit burning          │  │
│  │  certificate.rs    On-chain retirement certificates     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                │
│  ┌──────────────────────┐  ┌──────────────────────────────┐   │
│  │  Stellar Assets      │  │   Stellar DEX                │   │
│  │                      │  │                              │   │
│  │  One asset per       │  │  Credit ↔ USDC orderbooks    │   │
│  │  project vintage:    │  │  Instant settlement          │   │
│  │  PROJ_001_2024       │  │  Native to protocol          │   │
│  │  PROJ_001_2023       │  │  No AMM, no slippage risk    │   │
│  │  PROJ_002_2024       │  └──────────────────────────────┘   │
│  └──────────────────────┘                                      │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  Public Indexer (off-chain)                              │ │
│  │  Horizon API → PostgreSQL                                │ │
│  │  Project search, credit inventory, retirement registry   │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
Why Stellar's Native DEX?
Competing carbon tokenization platforms (Toucan on Polygon, Moss on Ethereum) built or use custom AMM pools. This introduces slippage, impermanent loss, and liquidity fragmentation. Stellar's native DEX uses an orderbook model — identical to traditional commodity exchanges — which is precisely the right model for carbon credits:

Price discovery via limit orders, not AMM curves
No liquidity providers needed
Settlement in 3–5 seconds at a fraction of a cent
USDC denominated — familiar to corporate treasury teams


Smart Contract Design
Project Registry (registry.rs)
rust#[contracttype]
pub enum ProjectType {
    Reforestation,
    SoilCarbon,
    Biochar,
    Solar,
    Wind,
    CookingStoves,
    MangroveRestoration,
    BlueCarbonSeagrass,
    Other(String),
}

#[contracttype]
pub enum ProjectStatus {
    Pending,      // Awaiting verifier review
    Verified,     // Approved, credits can be issued
    Suspended,    // Under investigation
    Archived,     // No more credits will be issued
}

#[contracttype]
pub struct CarbonProject {
    pub project_id: String,
    pub developer: Address,
    pub verifier: Address,
    pub project_type: ProjectType,
    pub methodology: String,          // e.g., "Verra VM0007"
    pub country: String,              // ISO 3166
    pub gps_bounds: Vec<GpsCoord>,    // Project boundary polygon
    pub status: ProjectStatus,
    pub registered_at: u64,
    pub document_cids: Vec<String>,   // IPFS CIDs for PDFs
}

pub fn register_project(
    env: Env,
    developer: Address,
    project_type: ProjectType,
    methodology: String,
    country: String,
    gps_bounds: Vec<GpsCoord>,
    document_cids: Vec<String>,
) -> String   // Returns project_id

pub fn verify_project(
    env: Env,
    verifier: Address,
    project_id: String,
)

pub fn suspend_project(
    env: Env,
    verifier: Address,
    project_id: String,
    reason: String,
)
Credit Issuance (issuance.rs)
rust#[contracttype]
pub struct CreditIssuance {
    pub issuance_id: String,
    pub project_id: String,
    pub vintage_year: u32,           // Year the CO₂ was sequestered
    pub quantity: i128,              // Number of credits (tCO₂e)
    pub verifier: Address,
    pub issued_at: u64,
    pub methodology_version: String,
    pub monitoring_report_cid: String,  // IPFS CID of monitoring report
}

// Mint credit tokens and record issuance on-chain
pub fn issue_credits(
    env: Env,
    verifier: Address,
    project_id: String,
    vintage_year: u32,
    quantity: i128,
    monitoring_report_cid: String,
) -> String   // Returns issuance_id

// Query total credits issued for a project
pub fn get_total_issued(
    env: Env,
    project_id: String,
    vintage_year: Option<u32>,
) -> i128
Retirement (retirement.rs)
rust#[contracttype]
pub struct RetirementRecord {
    pub retirement_id: String,
    pub retiree: Address,           // Account retiring the credits
    pub beneficiary_name: String,   // Entity claiming the offset (e.g., "Acme Corp 2024 GHG Report")
    pub project_id: String,
    pub vintage_year: u32,
    pub quantity: i128,
    pub reason: String,             // e.g., "Q4 2024 Scope 3 offset"
    pub retired_at: u64,
    pub tx_hash: String,            // For external verification
}

// Burn credit tokens and create immutable retirement record
pub fn retire_credits(
    env: Env,
    retiree: Address,
    project_id: String,
    vintage_year: u32,
    quantity: i128,
    beneficiary_name: String,
    reason: String,
) -> String   // Returns retirement_id

// Query retirements — public, queryable by anyone
pub fn get_retirements(
    env: Env,
    project_id: Option<String>,
    retiree: Option<Address>,
    vintage_year: Option<u32>,
) -> Vec<RetirementRecord>

// Verify a specific retirement certificate
pub fn verify_retirement(
    env: Env,
    retirement_id: String,
) -> Option<RetirementRecord>

Credit Tokenization Standard
Each project-vintage combination is represented as a unique Stellar Asset:
Asset Code Format: CL{PROJECT_ID_SHORT}{VINTAGE_YY}
Example:          CL001R24  (Project 001, Reforestation, Vintage 2024)

Asset Issuer:     CarbonLedger issuance contract address
Token metadata (not stored in Stellar asset itself, stored in Soroban):
json{
  "project_id": "PROJ_001",
  "vintage_year": 2024,
  "project_type": "Reforestation",
  "methodology": "Verra VM0007",
  "country": "Kenya",
  "verifier": "GABCD...XYZ",
  "issuance_id": "ISS_20240315_001",
  "monitoring_report": "ipfs://bafybeig..."
}
This metadata is queryable via the Soroban contract and surfaced in the DEX orderbook UI, so buyers know exactly what they are purchasing before placing an order.

Tech Stack
LayerTechnologySmart contractsRust + Soroban SDKCredit assetsStellar Asset Contracts (SAC)TradingStellar native DEX (orderbook)FrontendNext.js 14, TypeScript, Tailwind CSSProject document storageIPFS via web3.storageOff-chain indexerNode.js + PostgreSQL, Horizon APIRetirement certificate APIREST + JSON, publicly accessibleWalletFreighter (developers, buyers, verifiers)Settlement currencyUSDC on Stellar

Getting Started
Prerequisites

Node.js 18+
Rust + cargo
Stellar CLI (stellar)
Docker (for local indexer PostgreSQL)

1. Clone the repository
bashgit clone https://github.com/your-org/carbonledger
cd carbonledger
2. Build contracts
bashcd contracts
cargo build --target wasm32-unknown-unknown --release
3. Deploy to Testnet
bash# Deploy registry
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/registry.wasm \
  --source YOUR_SECRET_KEY \
  --network testnet

# Deploy issuance contract
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/issuance.wasm \
  --source YOUR_SECRET_KEY \
  --network testnet

# Deploy retirement contract
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/retirement.wasm \
  --source YOUR_SECRET_KEY \
  --network testnet
4. Start the indexer
bashcd indexer
cp .env.example .env
# Set HORIZON_URL, DATABASE_URL, and contract IDs
docker-compose up -d postgres
npm install && npm run dev
5. Start the portal
bashcd portal
cp .env.example .env.local
npm install && npm run dev
Visit http://localhost:3000

Project Structure
carbonledger/
├── contracts/
│   ├── registry/
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── registry.rs
│   │       └── types.rs
│   ├── issuance/
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── issuance.rs
│   │       └── assets.rs           # Stellar Asset Contract integration
│   ├── retirement/
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── retirement.rs
│   │       └── certificate.rs
│   └── tests/
│       ├── project_lifecycle.rs
│       ├── issuance_flow.rs
│       ├── retirement_immutability.rs
│       └── double_retirement_prevention.rs
│
├── portal/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   │   ├── ProjectCard/
│   │   │   ├── CreditInventory/
│   │   │   ├── DEXOrderbook/        # Live Stellar DEX orderbook view
│   │   │   ├── RetireModal/
│   │   │   └── CertificateViewer/
│   │   └── lib/
│   │       ├── stellar.ts
│   │       ├── dex.ts               # DEX offer creation helpers
│   │       └── contracts.ts
│   └── package.json
│
├── indexer/
│   ├── src/
│   │   ├── subscriber.ts
│   │   ├── models/
│   │   │   ├── Project.ts
│   │   │   ├── Issuance.ts
│   │   │   └── Retirement.ts
│   │   └── api/
│   │       ├── projects.ts
│   │       ├── credits.ts
│   │       └── retirements.ts      # Public retirement certificate API
│   ├── docker-compose.yml
│   └── package.json
│
├── docs/
│   ├── tokenization-standard.md
│   ├── verifier-onboarding.md
│   └── retirement-certificate-api.md
│
└── README.md

Roadmap
Phase 1 — Core Protocol (Current)

 Project registry contract with verifier approval flow
 Credit issuance: mint Stellar assets with on-chain provenance
 Retirement contract: burn tokens, create immutable certificate
 Basic portal: register project, issue credits, retire credits
 Public retirement certificate REST API

Phase 2 — Market Features

 DEX orderbook UI integrated into portal
 Project discovery marketplace (browse by type, country, vintage)
 Retirement certificate PDF generator (for corporate ESG reports)
 Off-chain indexer: full credit inventory and retirement history
 IPFS document upload for monitoring reports

Phase 3 — Ecosystem

 Multi-registry bridge: import existing credits from Verra/Gold Standard via verifier attestation
 Real-time credit price feed (aggregated from Stellar DEX)
 Corporate dashboard: track all retirements, generate annual offset reports
 API for registry integrations (webhook on new project verification)
 Explorer site: fully public, no wallet needed, full market transparency


Contributing
CarbonLedger is part of the Stellar Wave Program on Drips. All issues are open for community contributors during active Wave cycles.

Browse open issues at drips.network/wave/stellar/issues
Fork the repository
Create a branch: git checkout -b feat/your-feature
Add tests for any contract changes
Open a Pull Request

Good First Issues

Unit tests for double-retirement prevention (core invariant to test rigorously)
Implement the retirement certificate PDF generator (jsPDF or Puppeteer)
Add project filtering to the discovery marketplace (by country, type, vintage)
Localise portal to Spanish and French (carbon markets are global)
Build the DEX orderbook component using Stellar Horizon offers API

See CONTRIBUTING.md for code style, testing, and PR requirements.

Framing Note for Reviewers
CarbonLedger is infrastructure for transparent asset trading — not advocacy for any particular climate policy. The Stellar DEX is a technically superior settlement layer for carbon credits compared to Ethereum-based alternatives because of its native orderbook, near-zero fees, and USDC integration. The project is agnostic to which registry or methodology a credit uses; it provides the settlement and transparency layer that sits above existing standards.

License
MIT License — see LICENSE for details.

Carbon markets fail when they're opaque. CarbonLedger makes transparency the default, not the exception.
