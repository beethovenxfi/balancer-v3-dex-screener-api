# Balancer V3 DEX Screener Adapter

A comprehensive multi-chain REST API adapter for Balancer V3 pools, providing DEX Screener compatible endpoints across multiple blockchain networks.

## 📡 API Endpoints

### Base URL

```
http://localhost:3001/api
```

### Chain Management

-   `GET /api/chains` - List all supported chains

### Universal Endpoints

-   `GET /api/health` - Health check

### Per-Chain Endpoints

All chain-specific endpoints follow the pattern: `/api/{chain}/...`

| Endpoint                                                | Description        | Example                                        |
| ------------------------------------------------------- | ------------------ | ---------------------------------------------- |
| `GET /api/{chain}/latest-block`                         | Latest block info  | `/api/ethereum/latest-block`                   |
| `GET /api/{chain}/asset?id={address}`                   | Token information  | `/api/arbitrum/asset?id=0x123...`              |
| `GET /api/{chain}/pair?id={pairId}`                     | Pair information   | `/api/polygon/pair?id=0xabc...`                |
| `GET /api/{chain}/events?fromBlock={from}&toBlock={to}` | Block range events | `/api/base/events?fromBlock=1000&toBlock=2000` |

### API Examples

#### Get Latest Block

```bash
curl "http://localhost:3001/api/ethereum/latest-block"
```

```json
{
    "success": true,
    "data": {
        "blockNumber": 19123456,
        "blockTimestamp": 1752762620
    },
    "timestamp": 1752762620190
}
```

#### Get Asset Information

```bash
curl "http://localhost:3001/api/arbitrum/asset?id=0x82af49447d8a07e3bd95bd0d56f35241523fbab1"
```

```json
{
    "success": true,
    "data": {
        "id": "0x82af49447d8a07e3bd95bd0d56f35241523fbab1",
        "name": "Wrapped Ether",
        "symbol": "WETH",
        "decimals": 18,
        "totalSupply": null,
        "maxSupply": null,
        "circulatingSupply": null,
        "info": {
            "address": "0x82af49447d8a07e3bd95bd0d56f35241523fbab1"
        }
    },
    "timestamp": 1752762655998
}
```

#### Get Events

```bash
curl "http://localhost:3001/api/polygon/events?fromBlock=50000000&toBlock=50000100"
```

```json
{
    "success": true,
    "data": [
        {
            "block": {
                "blockNumber": 50000050,
                "blockTimestamp": 1752762610
            },
            "eventType": "swap",
            "txnId": "0x1234567890abcdef...",
            "txnIndex": 0,
            "eventIndex": 1,
            "maker": "0xuser123...",
            "pairId": "0xpool...-0xtoken0...-0xtoken1...",
            "priceNative": "1.0025",
            "asset0In": "1000000000000000000",
            "asset1Out": "1002500000000000000",
            "reserves": {
                "asset0": "1000000000000000000000",
                "asset1": "999000000000000000000"
            }
        }
    ],
    "timestamp": 1752762633599
}
```

## ⚙️ Configuration

### Environment Variables

Create a `.env` file with the following configuration:

### Environment Variable Pattern

For each supported chain, you need two environment variables:

-   `SUBGRAPH_URL_{CHAIN}` - The GraphQL subgraph endpoint
-   `RPC_URL_{CHAIN}` - The JSON-RPC endpoint

Where `{CHAIN}` is the uppercase chain slug (e.g., `ETHEREUM`, `ARBITRUM`, `POLYGON`).

## 🛠️ Installation & Setup

### Prerequisites

-   Node.js 18+ or Bun runtime
-   TypeScript 5.0+
-   Access to Balancer V3 subgraphs

### Installation

1. **Clone the repository:**

```bash
git clone https://github.com/your-repo/balancer-v3-dex-screener-adapter.git
cd balancer-v3-dex-screener-adapter
```

2. **Install dependencies:**

```bash
# Using npm
npm install

# Or using bun
bun install
```

3. **Configure environment:**

```bash
cp .env.example .env
# Edit .env with your API keys and endpoints
```

4. **Build the project:**

```bash
npm run build
```

5. **Start the server:**

```bash
# Development mode
npm run dev

# Production mode
npm start
```

### Development Commands

```bash
# Start development server with hot reload
npm run dev

# Build TypeScript
npm run build

# Run tests
npm test

# Lint code
npm run lint

# Test specific chain
curl http://localhost:3001/api/ethereum/latest-block
```

### Testing

The project includes comprehensive test coverage for all components:

#### Running Tests

```bash

# Run tests in isolation
bun run test:isolated

# Run specific module test
bun run test swap-util
```

**Note**: Due to module mocking conflicts in complex test suites, some tests may fail when run together but pass individually. Use `bun run test:isolated` to run tests in separate processes for guaranteed isolation.

## 🔧 Adding New Chains

### Step 1: Add Environment Variables

```env
SUBGRAPH_URL_NEWCHAIN=https://your-subgraph-url
RPC_URL_NEWCHAIN=https://your-rpc-url
```

### Step 2: Update Chain Configuration

```typescript
// In src/utils/chain-config.ts
import { newchain } from 'viem/chains';

// Add to initializeConfigs()
const newchainSubgraphUrl = process.env.SUBGRAPH_URL_NEWCHAIN;
const newchainRpcUrl = process.env.RPC_URL_NEWCHAIN;

if (newchainSubgraphUrl && newchainRpcUrl) {
    this.configs.set('newchain', {
        apiSlug: 'NEWCHAIN',
        viemChain: newchain,
        subgraphUrl: newchainSubgraphUrl,
        rpcUrl: newchainRpcUrl,
    });
}
```

### Step 3: Update GraphQL Client Mapping

```typescript
// In src/graphql/client.ts
const apiSlugToChainSlug: Record<string, string> = {
    // ... existing mappings
    NEWCHAIN: 'newchain',
};
```

### Step 4: Test the New Chain

```bash
curl http://localhost:3001/api/newchain/latest-block
```

## 📚 Token Mapping

### ERC4626 Support

The adapter automatically handles ERC4626 wrapped tokens by:

1. **Querying Balancer API**: Fetches token metadata per chain
2. **Condition Checking**: Validates `useUnderlyingForAddRemove` and `canUseBufferForSwaps`
3. **Underlying Resolution**: Maps pool tokens to underlying tokens

### Supported Token Types

-   Standard ERC20 tokens
-   ERC4626 wrapped tokens (with underlying mapping)
-   Boosted pool tokens
-   Native token wrappers (WETH, WMATIC, etc.)

## 🚨 Error Handling

### HTTP Status Codes

-   `200` - Success
-   `400` - Bad Request (invalid parameters, unsupported chain)
-   `404` - Not Found (asset/pair not found)
-   `500` - Internal Server Error

### Error Response Format

```json
{
    "success": false,
    "error": "Detailed error message",
    "timestamp": 1752762677324
}
```

### Unsupported Chain Example

```json
{
    "success": false,
    "error": "Unsupported chain: xyz. Supported chains: ethereum, arbitrum, polygon, optimism, base, avalanche, gnosis, sonic, hyperevm",
    "timestamp": 1752762677324
}
```

**Built with ❤️ for the Balancer ecosystem**
