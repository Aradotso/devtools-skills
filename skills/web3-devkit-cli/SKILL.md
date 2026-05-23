---
name: web3-devkit-cli
description: Bootstrap dApps, generate smart contracts, manage wallets, test RPCs, deploy contracts, and monitor on-chain activity for EVM and Solana blockchains.
triggers:
  - scaffold a new web3 project
  - generate an ERC20 token contract
  - check wallet balance on blockchain
  - test RPC connection for ethereum
  - deploy smart contract to base network
  - monitor on-chain events for wallet
  - create new blockchain wallet
  - add wagmi integration to frontend
---

# Web3 DevKit CLI

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

Web3 DevKit is an open-source CLI for EVM and Solana developers. It helps bootstrap projects, generate smart contracts, manage wallets, test RPCs, deploy contracts, and monitor on-chain activity across multiple blockchain networks.

## Installation

```bash
npm install -g web3-devkit-cli
```

Or run locally in a cloned repository:

```bash
git clone https://github.com/jazzband/web3-devkit-cli.git
cd web3-devkit-cli
npm install
npm run build
npm link  # Makes 'web3' command available globally
```

Run without linking:

```bash
npm run web3 -- <command>
```

## Core Commands

### Project Bootstrap

Initialize new blockchain projects with curated templates:

```bash
# Interactive mode
web3 init

# Specify chain type
web3 init evm           # EVM templates (Foundry, Hardhat)
web3 init solana        # Solana templates (Anchor)
web3 init fullstack     # Full-stack with frontend + backend
```

**Available templates:**
- `evm-foundry` - Foundry (Solidity, Forge, Cast)
- `evm-hardhat` - Hardhat + TypeScript
- `solana-anchor` - Anchor program workspace
- `nextjs-wagmi` - Next.js + wagmi (EVM frontend)
- `nextjs-solana-wallet` - Next.js + Solana wallet adapter
- `fullstack-evm` - Foundry contracts + Next.js + API
- `fullstack-solana` - Anchor + Next.js + API

### Contract Generation

Generate smart contracts and programs from boilerplate:

```bash
# Generate ERC20 token
web3 generate token -c evm -v erc20 -n MyToken -y

# Generate ERC721 NFT
web3 generate nft -c evm -v erc721 -n MyNFT -o ./contracts -y

# Generate Solana escrow program
web3 generate vault -c solana -v escrow-anchor -n MyEscrow -y
```

**Generation categories:**
- `token` - ERC20, SPL tokens
- `nft` - ERC721, ERC1155, Metaplex NFTs
- `staking` - Staking contracts/programs
- `vault` - Escrow, vaults, safes
- `prediction-market` - Prediction market logic

**Flags:**
- `-c, --chain` - Chain type: `evm` or `solana`
- `-v, --variant` - Template variant (e.g., `erc20`, `erc721`)
- `-n, --name` - Contract/program name
- `-o, --output` - Output directory (default: `./contracts`)
- `-y, --yes` - Skip confirmation prompts

### Wallet Management

Create wallets, check balances, and inspect tokens:

```bash
# Create new wallet
web3 wallet create --chain evm
web3 wallet create --chain solana

# Check native balance
web3 wallet balance -n base -a 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb

# List token balances
web3 wallet tokens -n ethereum -a 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb
```

**Supported networks:**
- **EVM:** ethereum, base, arbitrum, polygon, bsc, avalanche
- **Solana:** mainnet-beta, devnet, testnet

### RPC & Network Testing

Validate RPC connectivity and check network status:

```bash
# Test RPC endpoint
web3 rpc test -n base
web3 rpc test -n arbitrum
web3 rpc test -n solana

# Check network health
web3 network check -n ethereum
```

### Contract Deployment

Deploy contracts and programs with gas estimation:

```bash
# Estimate deployment gas (EVM)
web3 deploy evm -n base --estimate

# Deploy to network (EVM)
web3 deploy evm -n base -y

# Deploy Solana program
web3 deploy solana -n devnet -y

# View deployment history
web3 deploy history

# Verify contract on explorer
web3 verify -n base -a 0xContractAddress -c MyToken
```

Deployments are stored in `.web3-devkit/deployments/` with network, address, and timestamp.

### Event Monitoring

Monitor on-chain activity in real-time:

```bash
# Monitor contract events
web3 monitor contract -a 0xContractAddress -e Transfer -n base

# Monitor wallet transactions
web3 monitor wallet -a 0xYourWallet -n ethereum

# Monitor specific token for wallet
web3 monitor token -a 0xTokenAddress -w 0xWalletAddress -n base
```

### Frontend Integration

Add wallet providers and chain configurations to Next.js projects:

```bash
# Add wagmi (EVM)
web3 add wagmi -y

# Add RainbowKit
web3 add rainbowkit -y

# Add WalletConnect
web3 add wallet-connect -y

# Add Viem
web3 add viem -y

# Add Solana wallet adapter
web3 add solana-wallet -y
```

### Configuration Management

Persist project defaults to skip repetitive flags:

```bash
# Initialize config
web3 config init

# Get configuration value
web3 config get defaultChain

# Set configuration value
web3 config set rpc.base https://mainnet.base.org
web3 config set defaultChain evm
web3 config set framework foundry
```

Configuration is stored in `.web3-devkit/config.json`:

```json
{
  "defaultChain": "evm",
  "framework": "foundry",
  "rpc": {
    "base": "https://mainnet.base.org",
    "ethereum": "https://eth-mainnet.g.alchemy.com/v2/${ALCHEMY_API_KEY}"
  },
  "walletType": "local"
}
```

## Common Workflows

### Start a New EVM Project

```bash
# Initialize project
web3 init evm-foundry

# Generate ERC20 token
cd my-project
web3 generate token -c evm -v erc20 -n MyToken -y

# Set default network
web3 config set defaultChain evm
web3 config set rpc.base ${BASE_RPC_URL}

# Check RPC connectivity
web3 rpc test -n base

# Deploy contract
web3 deploy evm -n base -y

# Monitor token transfers
web3 monitor contract -a 0xDeployedTokenAddress -e Transfer -n base
```

### Start a Solana Project

```bash
# Initialize Anchor project
web3 init solana-anchor

# Generate escrow program
cd my-solana-project
web3 generate vault -c solana -v escrow-anchor -n MyEscrow -y

# Create Solana wallet
web3 wallet create --chain solana

# Check wallet balance
web3 wallet balance -n devnet -a <SOLANA_WALLET_ADDRESS>

# Deploy program
web3 deploy solana -n devnet -y
```

### Full-Stack dApp with Frontend

```bash
# Initialize full-stack project
web3 init fullstack-evm

# Generate NFT contract
cd my-fullstack-dapp
web3 generate nft -c evm -v erc721 -n MyNFT -y

# Add wagmi integration to frontend
cd frontend
web3 add wagmi -y
web3 add rainbowkit -y

# Configure network
cd ..
web3 config set rpc.base ${BASE_RPC_URL}

# Deploy and verify
web3 deploy evm -n base -y
web3 verify -n base -a 0xContractAddress -c MyNFT
```

## Working with Environment Variables

Always use environment variables for sensitive data:

```bash
# .env file
PRIVATE_KEY=${PRIVATE_KEY}
ALCHEMY_API_KEY=${ALCHEMY_API_KEY}
ETHERSCAN_API_KEY=${ETHERSCAN_API_KEY}
BASE_RPC_URL=https://mainnet.base.org
SOLANA_RPC_URL=https://api.devnet.solana.com
```

Load environment variables before running commands:

```bash
source .env
web3 deploy evm -n base -y
```

Or inline:

```bash
BASE_RPC_URL=${BASE_RPC_URL} web3 rpc test -n base
```

## TypeScript Integration Examples

### Using Generated ERC20 Contract

After generating an ERC20 token with `web3 generate token -c evm -v erc20 -n MyToken -y`:

```typescript
// contracts/MyToken.sol is generated
// Example interaction script

import { createPublicClient, createWalletClient, http } from 'viem';
import { base } from 'viem/chains';
import { privateKeyToAccount } from 'viem/accounts';

const account = privateKeyToAccount(`0x${process.env.PRIVATE_KEY}`);

const publicClient = createPublicClient({
  chain: base,
  transport: http(process.env.BASE_RPC_URL),
});

const walletClient = createWalletClient({
  chain: base,
  transport: http(process.env.BASE_RPC_URL),
  account,
});

// Read token balance
const balance = await publicClient.readContract({
  address: '0xTokenAddress',
  abi: MyTokenABI,
  functionName: 'balanceOf',
  args: [account.address],
});

console.log('Balance:', balance);

// Transfer tokens
const hash = await walletClient.writeContract({
  address: '0xTokenAddress',
  abi: MyTokenABI,
  functionName: 'transfer',
  args: ['0xRecipient', 1000000000000000000n],
});

console.log('Transaction hash:', hash);
```

### Monitoring Events Programmatically

```typescript
import { createPublicClient, http, parseAbiItem } from 'viem';
import { base } from 'viem/chains';

const client = createPublicClient({
  chain: base,
  transport: http(process.env.BASE_RPC_URL),
});

// Watch for Transfer events
const unwatch = client.watchEvent({
  address: '0xTokenAddress',
  event: parseAbiItem('event Transfer(address indexed from, address indexed to, uint256 value)'),
  onLogs: (logs) => {
    logs.forEach((log) => {
      console.log('Transfer detected:', {
        from: log.args.from,
        to: log.args.to,
        value: log.args.value,
      });
    });
  },
});

// Stop watching after 1 hour
setTimeout(() => unwatch(), 3600000);
```

### Custom Deployment Script

```typescript
import { execSync } from 'child_process';
import fs from 'fs';
import path from 'path';

interface DeploymentRecord {
  network: string;
  address: string;
  timestamp: string;
  deployer: string;
}

async function deployContract(network: string): Promise<void> {
  console.log(`Deploying to ${network}...`);
  
  // Run web3-devkit deploy
  const output = execSync(`web3 deploy evm -n ${network} -y`, {
    encoding: 'utf-8',
  });
  
  console.log(output);
  
  // Parse deployment address from output
  const addressMatch = output.match(/Deployed at: (0x[a-fA-F0-9]{40})/);
  if (!addressMatch) {
    throw new Error('Failed to parse deployment address');
  }
  
  const address = addressMatch[1];
  
  // Save deployment record
  const deploymentDir = path.join('.web3-devkit', 'deployments');
  const record: DeploymentRecord = {
    network,
    address,
    timestamp: new Date().toISOString(),
    deployer: process.env.DEPLOYER_ADDRESS || 'unknown',
  };
  
  fs.writeFileSync(
    path.join(deploymentDir, `${network}-latest.json`),
    JSON.stringify(record, null, 2)
  );
  
  console.log(`Deployment saved: ${address}`);
}

deployContract('base').catch(console.error);
```

## Project Structure After Init

After running `web3 init evm-foundry`, you'll get:

```
my-project/
├── contracts/           # Solidity contracts
│   └── Counter.sol
├── script/              # Deployment scripts
│   └── Counter.s.sol
├── test/                # Forge tests
│   └── Counter.t.sol
├── .env.example
├── foundry.toml
├── README.md
└── .web3-devkit/        # DevKit metadata
    ├── config.json
    └── deployments/
```

After running `web3 init fullstack-evm`:

```
my-fullstack-dapp/
├── contracts/           # Smart contracts
├── frontend/            # Next.js dApp
│   ├── components/
│   ├── pages/
│   └── public/
├── backend/
│   └── api/             # API routes
├── .env.example
└── README.md
```

## Troubleshooting

### RPC Connection Fails

**Problem:** `web3 rpc test -n base` times out or fails.

**Solutions:**
1. Check your RPC URL in config: `web3 config get rpc.base`
2. Update RPC URL: `web3 config set rpc.base https://mainnet.base.org`
3. Verify network connectivity: `curl -X POST ${BASE_RPC_URL} -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'`
4. Use environment variable: `BASE_RPC_URL=${ALCHEMY_BASE_URL} web3 rpc test -n base`

### Deployment Gas Estimation Error

**Problem:** `web3 deploy evm -n base --estimate` fails with "insufficient funds".

**Solutions:**
1. Check wallet balance: `web3 wallet balance -n base -a ${YOUR_ADDRESS}`
2. Fund wallet with native token (ETH for Base)
3. Verify private key is set: `echo $PRIVATE_KEY`
4. Use lower gas limits in deployment config

### Generated Contract Won't Compile

**Problem:** `web3 generate token -c evm -v erc20 -n MyToken -y` creates contract that won't compile.

**Solutions:**
1. Ensure Foundry or Hardhat is installed: `forge --version` or `npx hardhat --version`
2. Install dependencies: `forge install` or `npm install`
3. Check Solidity version compatibility in `foundry.toml` or `hardhat.config.ts`
4. Update imports in generated contract if needed

### Monitor Command Shows No Events

**Problem:** `web3 monitor wallet -a 0x... -n ethereum` runs but shows no activity.

**Solutions:**
1. Verify wallet has activity: check on block explorer
2. Ensure correct network: `web3 config get defaultChain`
3. Check RPC connection: `web3 rpc test -n ethereum`
4. Increase polling interval or use WebSocket RPC if available
5. Monitor specific events instead: `web3 monitor contract -a 0x... -e Transfer -n ethereum`

### Frontend Integration Not Working

**Problem:** `web3 add wagmi -y` completes but wallet doesn't connect.

**Solutions:**
1. Ensure you're in a Next.js project directory
2. Install peer dependencies: `npm install wagmi viem @tanstack/react-query`
3. Wrap your app with `WagmiConfig` in `_app.tsx`
4. Configure chains correctly in wagmi config
5. Check browser console for errors

### Command Not Found

**Problem:** `web3` command not recognized after installation.

**Solutions:**
1. Run `npm link` in the cloned repository
2. Use `npm run web3 -- <command>` if not linked
3. Check PATH includes npm global bin: `npm config get prefix`
4. Reinstall globally: `npm install -g web3-devkit-cli`

### Permission Denied on Wallet Creation

**Problem:** `web3 wallet create --chain evm` fails with permission error.

**Solutions:**
1. Check directory permissions: `ls -la .web3-devkit/`
2. Create directory manually: `mkdir -p .web3-devkit/wallets`
3. Run with appropriate permissions
4. Store wallet files in user home directory instead

## Advanced Configuration

### Custom RPC Endpoints per Network

```bash
# Set custom RPC URLs
web3 config set rpc.ethereum https://eth-mainnet.g.alchemy.com/v2/${ALCHEMY_API_KEY}
web3 config set rpc.base https://base-mainnet.g.alchemy.com/v2/${ALCHEMY_API_KEY}
web3 config set rpc.arbitrum https://arb-mainnet.g.alchemy.com/v2/${ALCHEMY_API_KEY}
web3 config set rpc.solana https://api.mainnet-beta.solana.com
```

### Multi-Chain Deployment Strategy

```bash
# Deploy to multiple networks sequentially
for network in base arbitrum polygon; do
  echo "Deploying to $network..."
  web3 deploy evm -n $network -y
  web3 verify -n $network -a $(web3 deploy history | grep $network | awk '{print $2}') -c MyToken
done
```

### Monitoring Multiple Contracts

```bash
# Monitor multiple contracts in parallel
web3 monitor contract -a 0xToken1 -e Transfer -n base &
web3 monitor contract -a 0xToken2 -e Transfer -n base &
web3 monitor wallet -a 0xMyWallet -n base &
wait
```

## Resources

- **Documentation:** Check `docs/` directory in repository
- **Templates:** Browse `templates/` for project scaffolds
- **Generators:** Review `generators/` for contract boilerplates
- **Integrations:** See `integrations/` for frontend wallet adapters

## Requirements

- Node.js 18+
- npm or yarn
- For EVM: Foundry or Hardhat installed
- For Solana: Anchor CLI installed
- Valid RPC endpoints for target networks
