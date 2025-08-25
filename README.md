# Solana Integration on the Internet Computer

This project demonstrates how canister smart contracts on the Internet Computer Protocol (ICP) can interact directly with the Solana blockchain. It leverages ICP's HTTPS outcalls for querying Solana JSON-RPC services and threshold Ed25519 cryptography for signing and submitting transactions to Solana.

Key features:
- Query Solana blockchain data (e.g., balances, transactions) via the SOL RPC canister.
- Sign Solana transactions using threshold Schnorr (Ed25519) without exposing private keys.
- Frontend for user interaction, built with modern web tools.
- Backend canister handling Solana interactions.

This integration enables chain fusion between ICP and Solana, allowing canisters to hold SOL assets, sign transactions, and more.

For more on the underlying technologies:
- [Solana Integration on ICP](https://internetcomputer.org/docs/building-apps/chain-fusion/solana/overview)
- [Threshold Schnorr (Ed25519)](https://internetcomputer.org/docs/building-apps/network-features/signatures/t-schnorr)
- [HTTPS Outcalls](https://internetcomputer.org/docs/references/https-outcalls-how-it-works)
- [SOL RPC Canister](https://github.com/dfinity/sol-rpc-canister) (used for JSON-RPC interactions)

## Prerequisites

Before starting, ensure you have:
- **dfx SDK**: Version 0.28.0 or later. Install via `sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"`.
- **Node.js and npm**: For frontend development (v18+ recommended).
- **Rust and Cargo**: For backend canister compilation (install via `rustup`).
- **A cycles wallet**: On ICP mainnet, with sufficient cycles for deployments and HTTPS outcalls.
- **Git**: To clone the repository.
- **ICP Identity**: Set up with `dfx identity new <name>` and fund it if deploying to mainnet.

## Cloning the Project

1. Clone the repository from GitHub:
   ```
   git clone https://github.com/your-username/sol_icp_poc.git
   cd sol_icp_poc
   ```

   (Replace `your-username` with the actual GitHub repository owner if different.)

2. If you're forking or customizing, replace any hardcoded canister IDs in the code (e.g., in `src/sol_icp_poc_backend/src/lib.rs` or frontend env files) with your own after deployment.

## Setting Up Environment Variables

Environment variables are crucial for configuring canister IDs, networks, and Solana clusters. They must be set **before** running `npm install`, `cargo build`, or `dfx deploy --network ic` to avoid build/runtime errors like "Subtyping error: text" (often caused by mismatched Candid types, e.g., sending a string instead of a variant like `RpcSources::Default(variant { Mainnet })`).

Create a `.env` file at the project root (and optionally `.env.production` for production overrides). Add the following variables, replacing placeholders with your values.

### Core Variables
```
DFX_VERSION=0.28.0
DFX_NETWORK=ic  # For mainnet; use 'local' for development
```

### Your Canister IDs (After Deployment)
Run `dfx canister id <name> --network ic` after initial deployment to get these.
```
CANISTER_ID_SOL_ICP_POC_FRONTEND=your-frontend-canister-id
CANISTER_ID_SOL_ICP_POC_BACKEND=your-backend-canister-id
CANISTER_ID=your-backend-canister-id  # Generic alias for backend
CANISTER_CANDID_PATH=/absolute/path/to/src/sol_icp_poc_backend/sol_icp_poc_backend.did  # If needed for Candid generation
```

### System Canisters (ICP Defaults)
```
ICP_LEDGER_CANISTER_ID=ryjl3-tyaaa-aaaaa-aaaba-cai  # ICP Ledger
INTERNET_IDENTITY_CANISTER_ID=rdmx6-jaaaa-aaaaa-aaadq-cai  # Internet Identity
```

### Solana Integration
Use the public SOL RPC canister or deploy your own.
```
SOL_RPC_CANISTER_ID=tghme-zyaaa-aaaar-qarca-cai  # Public mainnet SOL RPC canister
SOLANA_CLUSTER=Mainnet  # Or Devnet/Testnet
```

If deploying your own SOL RPC canister:
- Set `SOL_RPC_CANISTER_ID` to your deployed ID.
- Provide API keys via the canister's `updateApiKeys` method (see SOL RPC README).
- Update `dfx.json` with remote/wasm/candid entries for the SOL RPC canister.

### Frontend Variables (Prefixed for Bundlers like Vite)
Mirror the above for frontend embedding (Vite uses `VITE_` prefix).
```
VITE_DFX_NETWORK=ic
VITE_CANISTER_ID_SOL_ICP_POC_BACKEND=your-backend-canister-id
VITE_CANISTER_ID_SOL_ICP_POC_FRONTEND=your-frontend-canister-id
VITE_INTERNET_IDENTITY_CANISTER_ID=rdmx6-jaaaa-aaaaa-aaadq-cai
VITE_SOL_RPC_CANISTER_ID=tghme-zyaaa-aaaar-qarca-cai
VITE_SOLANA_CLUSTER=Mainnet
VITE_DERIVATION_ORIGIN=https://your-frontend-canister-id.ic0.app  # For II/custom domains
```

### Exporting Variables to Your Shell

After creating `.env`, export them for the current session. This ensures `dfx`, `npm`, and `cargo` can access them.

#### macOS/Linux
```
set -a
source ./.env
source ./.env.production  # If using a production file
set +a
# Verify
env | grep -E 'CANISTER_ID|DFX_|SOL_|VITE_|INTERNET_IDENTITY|LEDGER'
```

#### Windows (PowerShell)
```
Get-Content .env | ForEach-Object {
  if ($_ -match '^\s*#') { } elseif ($_ -match '^\s*([^=]+)=(.*)$') { [Environment]::SetEnvironmentVariable($matches[1], $matches[2], 'Process') }
}
# For .env.production too, if applicable
# Verify
Get-ChildItem Env: | Where-Object { $_.Name -match 'CANISTER_ID|DFX_|SOL_|VITE_|INTERNET_IDENTITY|LEDGER' }
```

#### Windows (Command Prompt)
Use a script or set manually:
```
for /f "tokens=1,2 delims==" %a in (.env) do set %a=%b
```

For persistence across sessions, add to your system/user environment variables via OS settings.

**Tip**: If variables are lost after reboot, add them to your shell profile (e.g., `.bashrc` on Linux/macOS) or use a tool like `direnv`.

## Building and Installing Dependencies

1. Install frontend dependencies:
   ```
   npm install
   ```

2. Build the Rust backend canister:
   ```
   cargo build --release --target wasm32-unknown-unknown
   ```

   (This compiles the backend Wasm module.)

## Deployment

Deploy to ICP mainnet or local replica.

### Local Deployment
1. Start the local replica:
   ```
   dfx start --background
   ```

2. Deploy canisters:
   ```
   dfx deploy
   ```

   Your frontend will be available at `http://localhost:4943?canisterId={frontend_canister_id}`.

### Mainnet Deployment (--network ic)
Ensure environment variables are set (see above).

1. Clean previous builds:
   ```
   rm -rf node_modules .vite .parcel-cache dist
   npm ci
   npm run build  # Builds frontend with baked-in env vars
   ```

2. Deploy:
   ```
   dfx deploy --network ic
   ```

   - If using a custom SOL RPC canister, include it in `dfx.json` as a remote canister and deploy it first.
   - Fund your cycles wallet if needed for HTTPS outcalls.

Your app will be live at `https://{frontend_canister_id}.ic0.app`.

**Note on Errors**: If you see "Subtyping error: text" on calls to SOL RPC, it's likely a Candid mismatch (e.g., sending a string like "Mainnet" instead of `variant { Default = variant { Mainnet } }`). Ensure your code constructs proper RpcSources variants, and `SOLANA_CLUSTER` is set correctly.

## Running the Project Locally

- Start development server (frontend):
  ```
  npm start
  ```
  Access at `http://localhost:8080` (proxies to local replica).

- Generate Candid interface (after backend changes):
  ```
  npm run generate
  ```

## Troubleshooting

- **Cycles Issues**: HTTPS outcalls (e.g., to Solana providers) require cycles. Top up via `dfx wallet send`.
- **Consensus Failures**: For fast-changing Solana data (e.g., slots), use transformations or durable nonces.
- **IPv6 Limitations**: Providers must support IPv6; IPv4 may route through proxies.
- **Debugging Calls**: Use browser devtools to log env vars and Candid shapes.

## Contributing

Contributions welcome! Fork, make changes, and submit a PR.

## License

Apache 2.0 (see LICENSE).