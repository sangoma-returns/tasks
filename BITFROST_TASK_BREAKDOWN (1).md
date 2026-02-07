# Bitfrost Platform - Production Implementation Task Breakdown

> **Version**: 1.0.0  
> **Created**: February 7, 2026  
> **Purpose**: Comprehensive task definitions for AI-driven development  
> **Scope**: Complete production-ready implementation

---

## Table of Contents

1. [Project Structure & Setup](#phase-1-project-structure--setup)
2. [Smart Contract Development](#phase-2-smart-contract-development)
3. [Backend Infrastructure](#phase-3-backend-infrastructure)
4. [Frontend Foundation](#phase-4-frontend-foundation)
5. [Core Features](#phase-5-core-features)
6. [Integration Layer](#phase-6-integration-layer)
7. [Testing & Quality](#phase-7-testing--quality)
8. [Deployment & Operations](#phase-8-deployment--operations)

---

## PHASE 1: Project Structure & Setup

### Task 1.1: Initialize Monorepo Structure
**Priority**: P0 (Critical - Must be first)  
**Estimated Complexity**: Low  
**Dependencies**: None

#### Objective
Create a production-ready monorepo structure using Turborepo with proper workspace configuration.

#### File Structure to Create
```
bitfrost/
├── apps/
│   ├── web/                    # Next.js frontend application
│   ├── docs/                   # Documentation site
│   └── api/                    # API services (if needed)
├── packages/
│   ├── contracts/              # Smart contracts (Solidity)
│   ├── sdk/                    # TypeScript SDK for contracts
│   ├── ui/                     # Shared React components
│   ├── config/                 # Shared configurations
│   └── types/                  # Shared TypeScript types
├── services/
│   ├── supabase/               # Supabase configuration & functions
│   └── worker/                 # Background job workers
├── scripts/                    # Deployment & utility scripts
├── turbo.json                  # Turborepo configuration
├── package.json                # Root package.json
└── README.md                   # Project documentation
```

#### Required Files

**File: `/turbo.json`**
```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    }
  }
}
```

**File: `/package.json`**
```json
{
  "name": "bitfrost",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*",
    "services/*"
  ],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "turbo": "^1.11.0",
    "typescript": "^5.3.3"
  }
}
```

**File: `/.gitignore`**
```
# Dependencies
node_modules/
.pnp
.pnp.js

# Production
build/
dist/
.next/
out/

# Testing
coverage/
.nyc_output/

# Environment
.env
.env.local
.env.*.local

# Logs
*.log
npm-debug.log*
yarn-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Turbo
.turbo/
```

#### Success Criteria
- [ ] Monorepo initializes without errors
- [ ] All workspace commands work (`pnpm install`, `turbo build`)
- [ ] TypeScript paths resolve correctly
- [ ] Hot reload works in dev mode

---

### Task 1.2: Configure TypeScript Foundation
**Priority**: P0  
**Estimated Complexity**: Low  
**Dependencies**: Task 1.1

#### Objective
Set up shared TypeScript configuration with strict type checking and path aliases.

#### Required Files

**File: `/packages/types/package.json`**
```json
{
  "name": "@bitfrost/types",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "exports": {
    ".": "./src/index.ts",
    "./exchange": "./src/exchange.ts",
    "./vault": "./src/vault.ts",
    "./user": "./src/user.ts"
  }
}
```

**File: `/packages/types/tsconfig.json`**
```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**File: `/tsconfig.base.json`**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "incremental": true,
    "noUncheckedIndexedAccess": true,
    "allowImportingTsExtensions": true,
    "paths": {
      "@bitfrost/types": ["./packages/types/src"],
      "@bitfrost/types/*": ["./packages/types/src/*"],
      "@bitfrost/ui": ["./packages/ui/src"],
      "@bitfrost/ui/*": ["./packages/ui/src/*"],
      "@bitfrost/contracts": ["./packages/contracts/src"],
      "@bitfrost/sdk": ["./packages/sdk/src"]
    }
  }
}
```

#### Core Type Definitions

**File: `/packages/types/src/index.ts`**
```typescript
// Re-export all types from submodules
export * from './exchange';
export * from './vault';
export * from './user';
export * from './position';
export * from './order';
export * from './funding';
export * from './api';
```

**File: `/packages/types/src/exchange.ts`**
```typescript
/**
 * Exchange identifiers - all supported exchanges
 */
export type ExchangeId = 
  | 'hyperliquid'
  | 'paradex'
  | 'binance'
  | 'bybit'
  | 'okx'
  | 'aster'
  | 'xyz'
  | 'vntl'
  | 'km'
  | 'cash'
  | 'flx'
  | 'hyna';

/**
 * Exchange type classification
 */
export type ExchangeType = 'CEX' | 'DEX';

/**
 * Exchange configuration
 */
export interface ExchangeConfig {
  id: ExchangeId;
  name: string;
  type: ExchangeType;
  apiUrl: string;
  wsUrl?: string;
  isHip3?: boolean; // Hyperliquid HIP-3 DEX
  requiredApiKeys: string[];
  supportedOrderTypes: OrderType[];
  minOrderSize: Record<string, number>; // token -> min size
  takerFee: number; // e.g., 0.0005 for 0.05%
  makerFee: number;
}

/**
 * Exchange connection status
 */
export interface ExchangeStatus {
  exchangeId: ExchangeId;
  isConnected: boolean;
  lastPing: number;
  hasApiKeys: boolean;
  balance?: number;
  error?: string;
}
```

**File: `/packages/types/src/vault.ts`**
```typescript
/**
 * On-chain vault for user deposits
 */
export interface Vault {
  address: `0x${string}`;
  owner: `0x${string}`;
  totalDeposits: bigint;
  totalWithdrawals: bigint;
  currentBalance: bigint;
  createdAt: number;
  updatedAt: number;
}

/**
 * Vault deposit transaction
 */
export interface VaultDeposit {
  id: string;
  vaultAddress: `0x${string}`;
  userAddress: `0x${string}`;
  amount: bigint;
  token: `0x${string}`; // USDC address
  txHash: `0x${string}`;
  blockNumber: bigint;
  timestamp: number;
  status: 'pending' | 'confirmed' | 'failed';
}

/**
 * Vault withdrawal transaction
 */
export interface VaultWithdrawal {
  id: string;
  vaultAddress: `0x${string}`;
  userAddress: `0x${string}`;
  amount: bigint;
  token: `0x${string}`;
  txHash: `0x${string}`;
  blockNumber: bigint;
  timestamp: number;
  status: 'pending' | 'confirmed' | 'failed';
}

/**
 * Distribution to exchange
 */
export interface ExchangeDistribution {
  id: string;
  vaultAddress: `0x${string}`;
  exchangeId: ExchangeId;
  amount: bigint;
  timestamp: number;
  txHash?: `0x${string}`;
  status: 'pending' | 'sent' | 'confirmed' | 'failed';
}
```

**File: `/packages/types/src/order.ts`**
```typescript
/**
 * Order types supported across exchanges
 */
export type OrderType = 'MARKET' | 'LIMIT' | 'TWAP' | 'STOP_LOSS' | 'TAKE_PROFIT';

/**
 * Order side
 */
export type OrderSide = 'BUY' | 'SELL';

/**
 * Order status lifecycle
 */
export type OrderStatus = 
  | 'DRAFT'        // Created but not submitted
  | 'PENDING'      // Submitted to exchange, awaiting acceptance
  | 'OPEN'         // Active on exchange order book
  | 'PARTIAL'      // Partially filled
  | 'FILLED'       // Completely filled
  | 'CANCELLED'    // User cancelled
  | 'REJECTED'     // Exchange rejected
  | 'EXPIRED'      // Time-in-force expired
  | 'FAILED';      // System error

/**
 * Base order interface
 */
export interface Order {
  id: string;
  userId: string;
  exchangeId: ExchangeId;
  symbol: string; // e.g., "BTC-PERP"
  type: OrderType;
  side: OrderSide;
  quantity: number;
  price?: number; // undefined for market orders
  status: OrderStatus;
  filledQuantity: number;
  averageFillPrice?: number;
  createdAt: number;
  updatedAt: number;
  expiresAt?: number;
  exchangeOrderId?: string;
  clientOrderId: string;
  
  // Execution details
  fills?: OrderFill[];
  fees?: OrderFee[];
  
  // Metadata
  strategyId?: string; // If part of automated strategy
  pairOrderId?: string; // For arbitrage pairs
  notes?: string;
}

/**
 * Order fill (partial or complete)
 */
export interface OrderFill {
  id: string;
  orderId: string;
  quantity: number;
  price: number;
  timestamp: number;
  tradeId: string;
  fee: number;
  feeToken: string;
}

/**
 * Order fee breakdown
 */
export interface OrderFee {
  type: 'MAKER' | 'TAKER';
  amount: number;
  token: string;
  rate: number; // percentage
}

/**
 * Multi-leg order for arbitrage
 */
export interface MultiLegOrder {
  id: string;
  userId: string;
  legs: Order[];
  status: OrderStatus;
  createdAt: number;
  expectedSpread: number;
  actualSpread?: number;
  totalFees: number;
  netPnl?: number;
}
```

**File: `/packages/types/src/position.ts`**
```typescript
/**
 * Trading position on an exchange
 */
export interface Position {
  id: string;
  userId: string;
  exchangeId: ExchangeId;
  symbol: string;
  side: 'LONG' | 'SHORT';
  quantity: number;
  entryPrice: number;
  currentPrice: number;
  leverage: number;
  liquidationPrice?: number;
  
  // PnL
  unrealizedPnl: number;
  realizedPnl: number;
  
  // Funding
  fundingRate: number; // Current 8hr rate
  fundingPaid: number; // Total funding paid/received
  nextFundingTime: number;
  
  // Margin
  margin: number;
  marginRatio: number;
  
  // Timestamps
  openedAt: number;
  updatedAt: number;
  closedAt?: number;
  
  // Metadata
  strategyId?: string;
  pairPositionId?: string; // For arbitrage pairs
}

/**
 * Arbitrage position pair
 */
export interface ArbitragePosition {
  id: string;
  userId: string;
  longLeg: Position;
  shortLeg: Position;
  
  // Spread metrics
  entrySpread: number;
  currentSpread: number;
  targetSpread?: number;
  
  // Combined PnL
  totalUnrealizedPnl: number;
  totalRealizedPnl: number;
  totalFundingCollected: number;
  
  // Risk
  netExposure: number; // Should be ~0 for delta neutral
  marginUtilization: number;
  
  // Status
  status: 'OPEN' | 'CLOSING' | 'CLOSED';
  openedAt: number;
  closedAt?: number;
}
```

**File: `/packages/types/src/funding.ts`**
```typescript
/**
 * Funding rate data point
 */
export interface FundingRate {
  exchangeId: ExchangeId;
  symbol: string;
  rate: number; // 8-hour rate (e.g., 0.0001 = 0.01%)
  timestamp: number;
  nextFundingTime: number;
  
  // Calculated metrics
  dailyRate: number; // Annualized: rate * 3
  annualizedRate: number; // APY: (1 + rate)^1095 - 1
}

/**
 * Funding rate spread opportunity
 */
export interface FundingSpread {
  symbol: string;
  longExchange: ExchangeId;
  shortExchange: ExchangeId;
  longRate: number;
  shortRate: number;
  spread: number; // shortRate - longRate
  annualizedSpread: number;
  
  // Feasibility
  isFeasible: boolean;
  minCapital: number;
  expectedDailyReturn: number;
  breakEvenDays: number;
  
  // Metadata
  calculatedAt: number;
  expiresAt: number;
}

/**
 * Historical funding rate
 */
export interface FundingRateHistory {
  exchangeId: ExchangeId;
  symbol: string;
  timestamp: number;
  rate: number;
  
  // Statistics
  avgRate24h?: number;
  avgRate7d?: number;
  maxRate24h?: number;
  minRate24h?: number;
  volatility?: number;
}
```

**File: `/packages/types/src/user.ts`**
```typescript
/**
 * User account
 */
export interface User {
  id: string;
  walletAddress: `0x${string}`;
  email?: string;
  username?: string;
  
  // Vault
  vaultAddress?: `0x${string}`;
  
  // Exchange API keys (encrypted)
  exchangeCredentials: ExchangeCredential[];
  
  // Settings
  preferences: UserPreferences;
  
  // Metadata
  createdAt: number;
  updatedAt: number;
  lastLoginAt: number;
}

/**
 * Exchange API credential
 */
export interface ExchangeCredential {
  exchangeId: ExchangeId;
  apiKey: string; // Encrypted
  apiSecret: string; // Encrypted
  passphrase?: string; // For exchanges that require it
  isActive: boolean;
  createdAt: number;
  updatedAt: number;
}

/**
 * User preferences
 */
export interface UserPreferences {
  defaultSlippage: number;
  defaultLeverage: number;
  autoCompound: boolean;
  riskLevel: 'LOW' | 'MEDIUM' | 'HIGH';
  notifications: {
    email: boolean;
    push: boolean;
    funding: boolean;
    positions: boolean;
    pnl: boolean;
  };
}

/**
 * User portfolio summary
 */
export interface Portfolio {
  userId: string;
  
  // Total values
  totalEquity: number;
  totalDeposits: number;
  totalWithdrawals: number;
  
  // PnL
  unrealizedPnl: number;
  realizedPnl: number;
  totalPnl: number;
  
  // Funding
  totalFundingCollected: number;
  
  // Positions
  activePositions: number;
  totalPositionsOpened: number;
  
  // Risk
  totalMarginUsed: number;
  marginUtilization: number;
  netExposure: number;
  
  // Performance
  roi: number;
  sharpeRatio?: number;
  maxDrawdown?: number;
  
  // Exchange breakdown
  exchangeBalances: ExchangeBalance[];
  
  updatedAt: number;
}

/**
 * Balance on a specific exchange
 */
export interface ExchangeBalance {
  exchangeId: ExchangeId;
  totalBalance: number;
  availableBalance: number;
  marginUsed: number;
  unrealizedPnl: number;
  positions: number;
}
```

**File: `/packages/types/src/api.ts`**
```typescript
/**
 * Standard API response wrapper
 */
export interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  error?: ApiError;
  timestamp: number;
}

/**
 * API error response
 */
export interface ApiError {
  code: string;
  message: string;
  details?: Record<string, any>;
  statusCode: number;
}

/**
 * Paginated response
 */
export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  hasMore: boolean;
}

/**
 * WebSocket message types
 */
export type WsMessageType = 
  | 'FUNDING_RATE_UPDATE'
  | 'POSITION_UPDATE'
  | 'ORDER_UPDATE'
  | 'BALANCE_UPDATE'
  | 'PRICE_UPDATE';

/**
 * WebSocket message
 */
export interface WsMessage<T = any> {
  type: WsMessageType;
  data: T;
  timestamp: number;
}
```

#### Success Criteria
- [ ] All type files compile without errors
- [ ] Type imports work across packages
- [ ] IDE autocomplete works for all types
- [ ] No circular dependencies

---

## PHASE 2: Smart Contract Development

### Task 2.1: Vault Contract - Core Implementation
**Priority**: P0  
**Estimated Complexity**: High  
**Dependencies**: Task 1.1, 1.2

#### Objective
Implement production-ready Vault contract for user deposits with withdrawal/deposit logic and exchange distribution.

#### Contract Specifications

**File: `/packages/contracts/src/Vault.sol`**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

/**
 * @title Vault
 * @notice On-chain vault for managing user deposits and exchange distributions
 * @dev Upgradeable contract following UUPS pattern
 */
contract Vault is 
    OwnableUpgradeable,
    PausableUpgradeable,
    ReentrancyGuardUpgradeable,
    UUPSUpgradeable 
{
    using SafeERC20 for IERC20;

    /// @notice USDC token address (Arbitrum)
    IERC20 public usdc;
    
    /// @notice Authorized operator (backend service)
    address public operator;
    
    /// @notice Total deposits
    uint256 public totalDeposits;
    
    /// @notice Total withdrawals
    uint256 public totalWithdrawals;
    
    /// @notice User balances
    mapping(address => uint256) public balances;
    
    /// @notice User deposit history
    mapping(address => Deposit[]) public deposits;
    
    /// @notice User withdrawal history
    mapping(address => Withdrawal[]) public withdrawals;
    
    /// @notice Exchange distribution targets
    mapping(bytes32 => ExchangeTarget) public exchangeTargets;
    
    /// @notice Nonces for replay protection
    mapping(address => uint256) public nonces;

    struct Deposit {
        uint256 amount;
        uint256 timestamp;
        uint256 blockNumber;
    }

    struct Withdrawal {
        uint256 amount;
        uint256 timestamp;
        uint256 blockNumber;
        bytes32 withdrawalId;
    }

    struct ExchangeTarget {
        string exchangeId;
        address targetAddress;
        bool isActive;
        uint256 totalSent;
        uint256 lastSent;
    }

    /// @notice Events
    event Deposited(address indexed user, uint256 amount, uint256 newBalance);
    event Withdrawn(address indexed user, uint256 amount, bytes32 withdrawalId);
    event DistributedToExchange(
        string indexed exchangeId,
        address indexed target,
        uint256 amount
    );
    event OperatorChanged(address indexed oldOperator, address indexed newOperator);
    event ExchangeTargetAdded(string indexed exchangeId, address target);
    event ExchangeTargetRemoved(string indexed exchangeId);

    /// @notice Errors
    error InsufficientBalance();
    error InvalidAmount();
    error InvalidAddress();
    error UnauthorizedOperator();
    error ExchangeTargetNotFound();
    error ExchangeTargetAlreadyExists();

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    /**
     * @notice Initialize the vault
     * @param _usdc USDC token address
     * @param _operator Authorized operator address
     */
    function initialize(
        address _usdc,
        address _operator
    ) external initializer {
        if (_usdc == address(0) || _operator == address(0)) {
            revert InvalidAddress();
        }

        __Ownable_init();
        __Pausable_init();
        __ReentrancyGuard_init();
        __UUPSUpgradeable_init();

        usdc = IERC20(_usdc);
        operator = _operator;
    }

    /**
     * @notice Deposit USDC into vault
     * @param amount Amount to deposit
     */
    function deposit(uint256 amount) 
        external 
        whenNotPaused 
        nonReentrant 
    {
        if (amount == 0) revert InvalidAmount();

        // Transfer USDC from user
        usdc.safeTransferFrom(msg.sender, address(this), amount);

        // Update state
        balances[msg.sender] += amount;
        totalDeposits += amount;

        // Record deposit
        deposits[msg.sender].push(Deposit({
            amount: amount,
            timestamp: block.timestamp,
            blockNumber: block.number
        }));

        emit Deposited(msg.sender, amount, balances[msg.sender]);
    }

    /**
     * @notice Withdraw USDC from vault
     * @param amount Amount to withdraw
     * @param withdrawalId Unique withdrawal ID for tracking
     */
    function withdraw(uint256 amount, bytes32 withdrawalId) 
        external 
        whenNotPaused 
        nonReentrant 
    {
        if (amount == 0) revert InvalidAmount();
        if (balances[msg.sender] < amount) revert InsufficientBalance();

        // Update state
        balances[msg.sender] -= amount;
        totalWithdrawals += amount;

        // Record withdrawal
        withdrawals[msg.sender].push(Withdrawal({
            amount: amount,
            timestamp: block.timestamp,
            blockNumber: block.number,
            withdrawalId: withdrawalId
        }));

        // Transfer USDC to user
        usdc.safeTransfer(msg.sender, amount);

        emit Withdrawn(msg.sender, amount, withdrawalId);
    }

    /**
     * @notice Distribute funds to exchange (operator only)
     * @param exchangeId Exchange identifier
     * @param amount Amount to send
     */
    function distributeToExchange(
        string calldata exchangeId,
        uint256 amount
    ) 
        external 
        onlyOperator 
        whenNotPaused 
        nonReentrant 
    {
        bytes32 key = keccak256(abi.encodePacked(exchangeId));
        ExchangeTarget storage target = exchangeTargets[key];

        if (!target.isActive) revert ExchangeTargetNotFound();
        if (amount == 0) revert InvalidAmount();

        // Update state
        target.totalSent += amount;
        target.lastSent = block.timestamp;

        // Transfer USDC
        usdc.safeTransfer(target.targetAddress, amount);

        emit DistributedToExchange(exchangeId, target.targetAddress, amount);
    }

    /**
     * @notice Add exchange distribution target (owner only)
     * @param exchangeId Exchange identifier
     * @param targetAddress Target address for funds
     */
    function addExchangeTarget(
        string calldata exchangeId,
        address targetAddress
    ) 
        external 
        onlyOwner 
    {
        if (targetAddress == address(0)) revert InvalidAddress();

        bytes32 key = keccak256(abi.encodePacked(exchangeId));
        if (exchangeTargets[key].isActive) {
            revert ExchangeTargetAlreadyExists();
        }

        exchangeTargets[key] = ExchangeTarget({
            exchangeId: exchangeId,
            targetAddress: targetAddress,
            isActive: true,
            totalSent: 0,
            lastSent: 0
        });

        emit ExchangeTargetAdded(exchangeId, targetAddress);
    }

    /**
     * @notice Remove exchange distribution target (owner only)
     * @param exchangeId Exchange identifier
     */
    function removeExchangeTarget(string calldata exchangeId) 
        external 
        onlyOwner 
    {
        bytes32 key = keccak256(abi.encodePacked(exchangeId));
        if (!exchangeTargets[key].isActive) {
            revert ExchangeTargetNotFound();
        }

        exchangeTargets[key].isActive = false;
        emit ExchangeTargetRemoved(exchangeId);
    }

    /**
     * @notice Update operator address (owner only)
     * @param newOperator New operator address
     */
    function setOperator(address newOperator) external onlyOwner {
        if (newOperator == address(0)) revert InvalidAddress();
        
        address oldOperator = operator;
        operator = newOperator;
        
        emit OperatorChanged(oldOperator, newOperator);
    }

    /**
     * @notice Pause contract (owner only)
     */
    function pause() external onlyOwner {
        _pause();
    }

    /**
     * @notice Unpause contract (owner only)
     */
    function unpause() external onlyOwner {
        _unpause();
    }

    /**
     * @notice Get user deposit history
     * @param user User address
     * @return Array of deposits
     */
    function getDeposits(address user) 
        external 
        view 
        returns (Deposit[] memory) 
    {
        return deposits[user];
    }

    /**
     * @notice Get user withdrawal history
     * @param user User address
     * @return Array of withdrawals
     */
    function getWithdrawals(address user) 
        external 
        view 
        returns (Withdrawal[] memory) 
    {
        return withdrawals[user];
    }

    /**
     * @notice Get exchange target details
     * @param exchangeId Exchange identifier
     * @return Exchange target struct
     */
    function getExchangeTarget(string calldata exchangeId) 
        external 
        view 
        returns (ExchangeTarget memory) 
    {
        bytes32 key = keccak256(abi.encodePacked(exchangeId));
        return exchangeTargets[key];
    }

    /**
     * @notice Authorize upgrade (UUPS requirement)
     * @param newImplementation New implementation address
     */
    function _authorizeUpgrade(address newImplementation) 
        internal 
        override 
        onlyOwner 
    {}

    /**
     * @notice Modifier for operator-only functions
     */
    modifier onlyOperator() {
        if (msg.sender != operator) revert UnauthorizedOperator();
        _;
    }
}
```

#### Deployment Script

**File: `/packages/contracts/scripts/deploy-vault.ts`**
```typescript
import { ethers, upgrades } from "hardhat";

async function main() {
  console.log("Deploying Vault contract...");

  // Get deployer
  const [deployer] = await ethers.getSigners();
  console.log("Deployer:", deployer.address);

  // USDC address on Arbitrum
  const USDC_ADDRESS = "0xaf88d065e77c8cC2239327C5EDb3A432268e5831";
  
  // Operator address (backend service)
  const OPERATOR_ADDRESS = process.env.OPERATOR_ADDRESS!;

  // Deploy
  const Vault = await ethers.getContractFactory("Vault");
  const vault = await upgrades.deployProxy(
    Vault,
    [USDC_ADDRESS, OPERATOR_ADDRESS],
    { kind: "uups" }
  );

  await vault.waitForDeployment();
  const vaultAddress = await vault.getAddress();

  console.log("Vault deployed to:", vaultAddress);
  console.log("Implementation:", await upgrades.erc1967.getImplementationAddress(vaultAddress));

  // Verify on Arbiscan
  console.log("Waiting for block confirmations...");
  await vault.deploymentTransaction()?.wait(5);

  console.log("Verifying contract...");
  await run("verify:verify", {
    address: vaultAddress,
    constructorArguments: [],
  });

  console.log("✅ Deployment complete!");
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

#### Testing Suite

**File: `/packages/contracts/test/Vault.test.ts`**
```typescript
import { expect } from "chai";
import { ethers, upgrades } from "hardhat";
import { Vault, IERC20 } from "../typechain-types";
import { SignerWithAddress } from "@nomiclabs/hardhat-ethers/signers";

describe("Vault", function () {
  let vault: Vault;
  let usdc: IERC20;
  let owner: SignerWithAddress;
  let operator: SignerWithAddress;
  let user1: SignerWithAddress;
  let user2: SignerWithAddress;

  const USDC_DECIMALS = 6;
  const parseUsdc = (amount: string) => 
    ethers.parseUnits(amount, USDC_DECIMALS);

  beforeEach(async function () {
    [owner, operator, user1, user2] = await ethers.getSigners();

    // Deploy mock USDC
    const MockERC20 = await ethers.getContractFactory("MockERC20");
    usdc = await MockERC20.deploy("USD Coin", "USDC", USDC_DECIMALS);
    await usdc.waitForDeployment();

    // Deploy Vault
    const Vault = await ethers.getContractFactory("Vault");
    vault = await upgrades.deployProxy(
      Vault,
      [await usdc.getAddress(), operator.address],
      { kind: "uups" }
    );
    await vault.waitForDeployment();

    // Mint USDC to users
    await usdc.mint(user1.address, parseUsdc("10000"));
    await usdc.mint(user2.address, parseUsdc("10000"));
  });

  describe("Deployment", function () {
    it("Should set correct USDC address", async function () {
      expect(await vault.usdc()).to.equal(await usdc.getAddress());
    });

    it("Should set correct operator", async function () {
      expect(await vault.operator()).to.equal(operator.address);
    });

    it("Should set correct owner", async function () {
      expect(await vault.owner()).to.equal(owner.address);
    });
  });

  describe("Deposits", function () {
    it("Should allow user to deposit USDC", async function () {
      const amount = parseUsdc("1000");

      // Approve
      await usdc.connect(user1).approve(await vault.getAddress(), amount);

      // Deposit
      await expect(vault.connect(user1).deposit(amount))
        .to.emit(vault, "Deposited")
        .withArgs(user1.address, amount, amount);

      expect(await vault.balances(user1.address)).to.equal(amount);
      expect(await vault.totalDeposits()).to.equal(amount);
    });

    it("Should reject zero deposits", async function () {
      await expect(
        vault.connect(user1).deposit(0)
      ).to.be.revertedWithCustomError(vault, "InvalidAmount");
    });

    it("Should record deposit history", async function () {
      const amount = parseUsdc("1000");
      await usdc.connect(user1).approve(await vault.getAddress(), amount);
      await vault.connect(user1).deposit(amount);

      const deposits = await vault.getDeposits(user1.address);
      expect(deposits.length).to.equal(1);
      expect(deposits[0].amount).to.equal(amount);
    });

    it("Should handle multiple deposits", async function () {
      await usdc.connect(user1).approve(
        await vault.getAddress(),
        parseUsdc("5000")
      );

      await vault.connect(user1).deposit(parseUsdc("1000"));
      await vault.connect(user1).deposit(parseUsdc("2000"));
      await vault.connect(user1).deposit(parseUsdc("1500"));

      expect(await vault.balances(user1.address)).to.equal(parseUsdc("4500"));
      
      const deposits = await vault.getDeposits(user1.address);
      expect(deposits.length).to.equal(3);
    });
  });

  describe("Withdrawals", function () {
    beforeEach(async function () {
      // Setup: user1 deposits 1000 USDC
      const amount = parseUsdc("1000");
      await usdc.connect(user1).approve(await vault.getAddress(), amount);
      await vault.connect(user1).deposit(amount);
    });

    it("Should allow user to withdraw", async function () {
      const amount = parseUsdc("500");
      const withdrawalId = ethers.id("withdrawal-1");

      await expect(vault.connect(user1).withdraw(amount, withdrawalId))
        .to.emit(vault, "Withdrawn")
        .withArgs(user1.address, amount, withdrawalId);

      expect(await vault.balances(user1.address)).to.equal(parseUsdc("500"));
      expect(await usdc.balanceOf(user1.address)).to.equal(parseUsdc("9500"));
    });

    it("Should reject withdrawal exceeding balance", async function () {
      await expect(
        vault.connect(user1).withdraw(parseUsdc("2000"), ethers.id("w1"))
      ).to.be.revertedWithCustomError(vault, "InsufficientBalance");
    });

    it("Should record withdrawal history", async function () {
      const withdrawalId = ethers.id("withdrawal-1");
      await vault.connect(user1).withdraw(parseUsdc("500"), withdrawalId);

      const withdrawals = await vault.getWithdrawals(user1.address);
      expect(withdrawals.length).to.equal(1);
      expect(withdrawals[0].amount).to.equal(parseUsdc("500"));
      expect(withdrawals[0].withdrawalId).to.equal(withdrawalId);
    });
  });

  describe("Exchange Distribution", function () {
    beforeEach(async function () {
      // Setup: Add exchange target
      await vault.addExchangeTarget(
        "hyperliquid",
        user2.address // Using user2 as mock exchange address
      );

      // Fund vault
      const amount = parseUsdc("10000");
      await usdc.connect(user1).approve(await vault.getAddress(), amount);
      await vault.connect(user1).deposit(amount);
    });

    it("Should allow operator to distribute to exchange", async function () {
      const amount = parseUsdc("5000");

      await expect(
        vault.connect(operator).distributeToExchange("hyperliquid", amount)
      )
        .to.emit(vault, "DistributedToExchange")
        .withArgs("hyperliquid", user2.address, amount);

      expect(await usdc.balanceOf(user2.address)).to.equal(
        parseUsdc("15000") // 10000 initial + 5000 distributed
      );
    });

    it("Should reject non-operator distribution", async function () {
      await expect(
        vault.connect(user1).distributeToExchange("hyperliquid", parseUsdc("1000"))
      ).to.be.revertedWithCustomError(vault, "UnauthorizedOperator");
    });

    it("Should reject distribution to inactive exchange", async function () {
      await expect(
        vault.connect(operator).distributeToExchange("binance", parseUsdc("1000"))
      ).to.be.revertedWithCustomError(vault, "ExchangeTargetNotFound");
    });

    it("Should track exchange distribution stats", async function () {
      await vault.connect(operator).distributeToExchange("hyperliquid", parseUsdc("3000"));
      
      const target = await vault.getExchangeTarget("hyperliquid");
      expect(target.totalSent).to.equal(parseUsdc("3000"));
      expect(target.lastSent).to.be.gt(0);
    });
  });

  describe("Exchange Target Management", function () {
    it("Should allow owner to add exchange target", async function () {
      await expect(
        vault.addExchangeTarget("hyperliquid", user2.address)
      )
        .to.emit(vault, "ExchangeTargetAdded")
        .withArgs("hyperliquid", user2.address);

      const target = await vault.getExchangeTarget("hyperliquid");
      expect(target.isActive).to.be.true;
      expect(target.targetAddress).to.equal(user2.address);
    });

    it("Should reject duplicate exchange target", async function () {
      await vault.addExchangeTarget("hyperliquid", user2.address);
      
      await expect(
        vault.addExchangeTarget("hyperliquid", user2.address)
      ).to.be.revertedWithCustomError(vault, "ExchangeTargetAlreadyExists");
    });

    it("Should allow owner to remove exchange target", async function () {
      await vault.addExchangeTarget("hyperliquid", user2.address);
      
      await expect(vault.removeExchangeTarget("hyperliquid"))
        .to.emit(vault, "ExchangeTargetRemoved")
        .withArgs("hyperliquid");

      const target = await vault.getExchangeTarget("hyperliquid");
      expect(target.isActive).to.be.false;
    });

    it("Should reject removing non-existent target", async function () {
      await expect(
        vault.removeExchangeTarget("binance")
      ).to.be.revertedWithCustomError(vault, "ExchangeTargetNotFound");
    });
  });

  describe("Operator Management", function () {
    it("Should allow owner to change operator", async function () {
      await expect(vault.setOperator(user1.address))
        .to.emit(vault, "OperatorChanged")
        .withArgs(operator.address, user1.address);

      expect(await vault.operator()).to.equal(user1.address);
    });

    it("Should reject zero address operator", async function () {
      await expect(
        vault.setOperator(ethers.ZeroAddress)
      ).to.be.revertedWithCustomError(vault, "InvalidAddress");
    });

    it("Should reject non-owner changing operator", async function () {
      await expect(
        vault.connect(user1).setOperator(user2.address)
      ).to.be.revertedWith("Ownable: caller is not the owner");
    });
  });

  describe("Pausability", function () {
    it("Should allow owner to pause", async function () {
      await vault.pause();
      expect(await vault.paused()).to.be.true;
    });

    it("Should block deposits when paused", async function () {
      await vault.pause();
      
      await usdc.connect(user1).approve(await vault.getAddress(), parseUsdc("1000"));
      await expect(
        vault.connect(user1).deposit(parseUsdc("1000"))
      ).to.be.revertedWith("Pausable: paused");
    });

    it("Should allow owner to unpause", async function () {
      await vault.pause();
      await vault.unpause();
      expect(await vault.paused()).to.be.false;
    });
  });

  describe("Upgradeability", function () {
    it("Should be upgradeable by owner", async function () {
      const VaultV2 = await ethers.getContractFactory("VaultV2Mock");
      const upgraded = await upgrades.upgradeProxy(
        await vault.getAddress(),
        VaultV2
      );

      expect(await upgraded.version()).to.equal("2.0.0");
    });

    it("Should preserve state after upgrade", async function () {
      // Deposit before upgrade
      await usdc.connect(user1).approve(await vault.getAddress(), parseUsdc("1000"));
      await vault.connect(user1).deposit(parseUsdc("1000"));

      // Upgrade
      const VaultV2 = await ethers.getContractFactory("VaultV2Mock");
      const upgraded = await upgrades.upgradeProxy(
        await vault.getAddress(),
        VaultV2
      );

      // Check state preserved
      expect(await upgraded.balances(user1.address)).to.equal(parseUsdc("1000"));
    });
  });
});
```

#### Success Criteria
- [ ] All tests pass (100% coverage)
- [ ] Contract compiles without warnings
- [ ] Gas optimization verified (deposits <100k gas, withdrawals <80k gas)
- [ ] Security audit considerations documented
- [ ] Upgradeable pattern implemented correctly
- [ ] Events emitted for all state changes

---

### Task 2.2: Vault SDK - TypeScript Interface
**Priority**: P0  
**Estimated Complexity**: Medium  
**Dependencies**: Task 2.1

#### Objective
Create TypeScript SDK for interacting with Vault contract from frontend and backend.

#### Implementation

**File: `/packages/sdk/src/vault.ts`**
```typescript
import { 
  createPublicClient,
  createWalletClient,
  http,
  type Address,
  type Hash,
  type PublicClient,
  type WalletClient,
  parseUnits,
  formatUnits
} from 'viem';
import { arbitrum } from 'viem/chains';
import { VaultABI } from './abis/Vault';

/**
 * Vault SDK for interacting with on-chain vault
 */
export class VaultSDK {
  private publicClient: PublicClient;
  private walletClient?: WalletClient;
  private vaultAddress: Address;

  constructor(
    vaultAddress: Address,
    walletClient?: WalletClient
  ) {
    this.vaultAddress = vaultAddress;
    this.walletClient = walletClient;

    this.publicClient = createPublicClient({
      chain: arbitrum,
      transport: http()
    });
  }

  /**
   * Get user's vault balance
   */
  async getBalance(userAddress: Address): Promise<number> {
    const balance = await this.publicClient.readContract({
      address: this.vaultAddress,
      abi: VaultABI,
      functionName: 'balances',
      args: [userAddress]
    });

    return parseFloat(formatUnits(balance as bigint, 6)); // USDC has 6 decimals
  }

  /**
   * Deposit USDC into vault
   */
  async deposit(amount: number): Promise<Hash> {
    if (!this.walletClient) {
      throw new Error('Wallet client not configured');
    }

    const [account] = await this.walletClient.getAddresses();
    const amountWei = parseUnits(amount.toString(), 6);

    // First approve USDC
    const USDC_ADDRESS = '0xaf88d065e77c8cC2239327C5EDb3A432268e5831' as Address;
    
    const approveHash = await this.walletClient.writeContract({
      address: USDC_ADDRESS,
      abi: ERC20_ABI,
      functionName: 'approve',
      args: [this.vaultAddress, amountWei],
      account
    });

    await this.publicClient.waitForTransactionReceipt({ hash: approveHash });

    // Then deposit
    const depositHash = await this.walletClient.writeContract({
      address: this.vaultAddress,
      abi: VaultABI,
      functionName: 'deposit',
      args: [amountWei],
      account
    });

    return depositHash;
  }

  /**
   * Withdraw USDC from vault
   */
  async withdraw(amount: number, withdrawalId: string): Promise<Hash> {
    if (!this.walletClient) {
      throw new Error('Wallet client not configured');
    }

    const [account] = await this.walletClient.getAddresses();
    const amountWei = parseUnits(amount.toString(), 6);
    const withdrawalIdBytes = toHex(withdrawalId);

    const hash = await this.walletClient.writeContract({
      address: this.vaultAddress,
      abi: VaultABI,
      functionName: 'withdraw',
      args: [amountWei, withdrawalIdBytes],
      account
    });

    return hash;
  }

  /**
   * Get user's deposit history
   */
  async getDeposits(userAddress: Address): Promise<VaultDeposit[]> {
    const deposits = await this.publicClient.readContract({
      address: this.vaultAddress,
      abi: VaultABI,
      functionName: 'getDeposits',
      args: [userAddress]
    });

    return (deposits as any[]).map(d => ({
      amount: parseFloat(formatUnits(d.amount, 6)),
      timestamp: Number(d.timestamp),
      blockNumber: Number(d.blockNumber)
    }));
  }

  /**
   * Get user's withdrawal history
   */
  async getWithdrawals(userAddress: Address): Promise<VaultWithdrawal[]> {
    const withdrawals = await this.publicClient.readContract({
      address: this.vaultAddress,
      abi: VaultABI,
      functionName: 'getWithdrawals',
      args: [userAddress]
    });

    return (withdrawals as any[]).map(w => ({
      amount: parseFloat(formatUnits(w.amount, 6)),
      timestamp: Number(w.timestamp),
      blockNumber: Number(w.blockNumber),
      withdrawalId: w.withdrawalId
    }));
  }

  /**
   * Get total vault stats
   */
  async getVaultStats(): Promise<VaultStats> {
    const [totalDeposits, totalWithdrawals] = await Promise.all([
      this.publicClient.readContract({
        address: this.vaultAddress,
        abi: VaultABI,
        functionName: 'totalDeposits'
      }),
      this.publicClient.readContract({
        address: this.vaultAddress,
        abi: VaultABI,
        functionName: 'totalWithdrawals'
      })
    ]);

    return {
      totalDeposits: parseFloat(formatUnits(totalDeposits as bigint, 6)),
      totalWithdrawals: parseFloat(formatUnits(totalWithdrawals as bigint, 6)),
      currentTVL: parseFloat(formatUnits((totalDeposits as bigint) - (totalWithdrawals as bigint), 6))
    };
  }

  /**
   * Watch for deposit events
   */
  watchDeposits(
    userAddress: Address,
    callback: (deposit: VaultDeposit) => void
  ) {
    return this.publicClient.watchContractEvent({
      address: this.vaultAddress,
      abi: VaultABI,
      eventName: 'Deposited',
      args: { user: userAddress },
      onLogs: (logs) => {
        logs.forEach((log) => {
          callback({
            amount: parseFloat(formatUnits(log.args.amount!, 6)),
            timestamp: Date.now(),
            blockNumber: Number(log.blockNumber)
          });
        });
      }
    });
  }
}

// Helper function
function toHex(str: string): `0x${string}` {
  return `0x${Buffer.from(str).toString('hex')}` as `0x${string}`;
}

// Types
interface VaultDeposit {
  amount: number;
  timestamp: number;
  blockNumber: number;
}

interface VaultWithdrawal {
  amount: number;
  timestamp: number;
  blockNumber: number;
  withdrawalId: string;
}

interface VaultStats {
  totalDeposits: number;
  totalWithdrawals: number;
  currentTVL: number;
}

// ERC20 ABI (minimal)
const ERC20_ABI = [
  {
    name: 'approve',
    type: 'function',
    stateMutability: 'nonpayable',
    inputs: [
      { name: 'spender', type: 'address' },
      { name: 'amount', type: 'uint256' }
    ],
    outputs: [{ type: 'bool' }]
  }
] as const;
```

#### Success Criteria
- [ ] SDK compiles without errors
- [ ] All contract functions accessible via SDK
- [ ] Type safety enforced
- [ ] Event watching works
- [ ] Error handling implemented

---

## PHASE 3: Backend Infrastructure

### Task 3.1: Supabase Setup & Configuration
**Priority**: P0  
**Estimated Complexity**: Medium  
**Dependencies**: Task 1.1

#### Objective
Set up Supabase project with complete database schema, authentication, and realtime configuration.

#### Database Schema

**File: `/services/supabase/migrations/001_initial_schema.sql`**
```sql
-- Enable required extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  wallet_address TEXT UNIQUE NOT NULL,
  email TEXT,
  username TEXT,
  vault_address TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  last_login_at TIMESTAMPTZ
);

CREATE INDEX idx_users_wallet ON users(wallet_address);

-- User preferences
CREATE TABLE user_preferences (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  default_slippage DECIMAL(5,4) DEFAULT 0.0050,
  default_leverage INTEGER DEFAULT 1,
  auto_compound BOOLEAN DEFAULT FALSE,
  risk_level TEXT CHECK (risk_level IN ('LOW', 'MEDIUM', 'HIGH')) DEFAULT 'MEDIUM',
  notifications JSONB DEFAULT '{"email": true, "push": false, "funding": true, "positions": true, "pnl": true}'::jsonb,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Exchange credentials (encrypted)
CREATE TABLE exchange_credentials (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  exchange_id TEXT NOT NULL,
  api_key TEXT NOT NULL, -- Encrypted
  api_secret TEXT NOT NULL, -- Encrypted
  passphrase TEXT, -- Encrypted, nullable
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, exchange_id)
);

CREATE INDEX idx_exchange_creds_user ON exchange_credentials(user_id);

-- Positions
CREATE TABLE positions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  exchange_id TEXT NOT NULL,
  symbol TEXT NOT NULL,
  side TEXT CHECK (side IN ('LONG', 'SHORT')) NOT NULL,
  quantity DECIMAL(20,8) NOT NULL,
  entry_price DECIMAL(20,8) NOT NULL,
  current_price DECIMAL(20,8) NOT NULL,
  leverage INTEGER DEFAULT 1,
  liquidation_price DECIMAL(20,8),
  unrealized_pnl DECIMAL(20,8) DEFAULT 0,
  realized_pnl DECIMAL(20,8) DEFAULT 0,
  funding_rate DECIMAL(10,8),
  funding_paid DECIMAL(20,8) DEFAULT 0,
  next_funding_time TIMESTAMPTZ,
  margin DECIMAL(20,8),
  margin_ratio DECIMAL(10,4),
  opened_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  closed_at TIMESTAMPTZ,
  strategy_id UUID,
  pair_position_id UUID
);

CREATE INDEX idx_positions_user ON positions(user_id);
CREATE INDEX idx_positions_exchange ON positions(exchange_id);
CREATE INDEX idx_positions_symbol ON positions(symbol);
CREATE INDEX idx_positions_status ON positions(closed_at) WHERE closed_at IS NULL;

-- Arbitrage positions (pairs)
CREATE TABLE arbitrage_positions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  long_position_id UUID NOT NULL REFERENCES positions(id),
  short_position_id UUID NOT NULL REFERENCES positions(id),
  entry_spread DECIMAL(10,8) NOT NULL,
  current_spread DECIMAL(10,8) NOT NULL,
  target_spread DECIMAL(10,8),
  total_unrealized_pnl DECIMAL(20,8) DEFAULT 0,
  total_realized_pnl DECIMAL(20,8) DEFAULT 0,
  total_funding_collected DECIMAL(20,8) DEFAULT 0,
  net_exposure DECIMAL(20,8) DEFAULT 0,
  margin_utilization DECIMAL(10,4),
  status TEXT CHECK (status IN ('OPEN', 'CLOSING', 'CLOSED')) DEFAULT 'OPEN',
  opened_at TIMESTAMPTZ DEFAULT NOW(),
  closed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_arb_positions_user ON arbitrage_positions(user_id);
CREATE INDEX idx_arb_positions_status ON arbitrage_positions(status);

-- Orders
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  exchange_id TEXT NOT NULL,
  symbol TEXT NOT NULL,
  type TEXT CHECK (type IN ('MARKET', 'LIMIT', 'TWAP', 'STOP_LOSS', 'TAKE_PROFIT')) NOT NULL,
  side TEXT CHECK (side IN ('BUY', 'SELL')) NOT NULL,
  quantity DECIMAL(20,8) NOT NULL,
  price DECIMAL(20,8),
  status TEXT CHECK (status IN ('DRAFT', 'PENDING', 'OPEN', 'PARTIAL', 'FILLED', 'CANCELLED', 'REJECTED', 'EXPIRED', 'FAILED')) DEFAULT 'DRAFT',
  filled_quantity DECIMAL(20,8) DEFAULT 0,
  average_fill_price DECIMAL(20,8),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ,
  exchange_order_id TEXT,
  client_order_id TEXT UNIQUE NOT NULL,
  strategy_id UUID,
  pair_order_id UUID,
  notes TEXT
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_exchange ON orders(exchange_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_client_id ON orders(client_order_id);

-- Order fills
CREATE TABLE order_fills (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  quantity DECIMAL(20,8) NOT NULL,
  price DECIMAL(20,8) NOT NULL,
  timestamp TIMESTAMPTZ DEFAULT NOW(),
  trade_id TEXT NOT NULL,
  fee DECIMAL(20,8) DEFAULT 0,
  fee_token TEXT DEFAULT 'USDC'
);

CREATE INDEX idx_fills_order ON order_fills(order_id);

-- Funding rates (historical)
CREATE TABLE funding_rates (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  exchange_id TEXT NOT NULL,
  symbol TEXT NOT NULL,
  rate DECIMAL(10,8) NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  next_funding_time TIMESTAMPTZ NOT NULL,
  daily_rate DECIMAL(10,8),
  annualized_rate DECIMAL(10,4)
);

CREATE INDEX idx_funding_exchange_symbol ON funding_rates(exchange_id, symbol);
CREATE INDEX idx_funding_timestamp ON funding_rates(timestamp DESC);

-- Unique constraint to prevent duplicates
CREATE UNIQUE INDEX idx_funding_unique ON funding_rates(exchange_id, symbol, timestamp);

-- Vault deposits
CREATE TABLE vault_deposits (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  vault_address TEXT NOT NULL,
  amount DECIMAL(20,8) NOT NULL,
  tx_hash TEXT NOT NULL,
  block_number BIGINT NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  status TEXT CHECK (status IN ('pending', 'confirmed', 'failed')) DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_vault_deposits_user ON vault_deposits(user_id);
CREATE INDEX idx_vault_deposits_tx ON vault_deposits(tx_hash);

-- Vault withdrawals
CREATE TABLE vault_withdrawals (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  vault_address TEXT NOT NULL,
  amount DECIMAL(20,8) NOT NULL,
  tx_hash TEXT NOT NULL,
  block_number BIGINT NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  status TEXT CHECK (status IN ('pending', 'confirmed', 'failed')) DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_vault_withdrawals_user ON vault_withdrawals(user_id);
CREATE INDEX idx_vault_withdrawals_tx ON vault_withdrawals(tx_hash);

-- Exchange distributions
CREATE TABLE exchange_distributions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  vault_address TEXT NOT NULL,
  exchange_id TEXT NOT NULL,
  amount DECIMAL(20,8) NOT NULL,
  timestamp TIMESTAMPTZ DEFAULT NOW(),
  tx_hash TEXT,
  status TEXT CHECK (status IN ('pending', 'sent', 'confirmed', 'failed')) DEFAULT 'pending'
);

CREATE INDEX idx_distributions_exchange ON exchange_distributions(exchange_id);
CREATE INDEX idx_distributions_status ON exchange_distributions(status);

-- Row Level Security (RLS)
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_preferences ENABLE ROW LEVEL SECURITY;
ALTER TABLE exchange_credentials ENABLE ROW LEVEL SECURITY;
ALTER TABLE positions ENABLE ROW LEVEL SECURITY;
ALTER TABLE arbitrage_positions ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE order_fills ENABLE ROW LEVEL SECURITY;
ALTER TABLE vault_deposits ENABLE ROW LEVEL SECURITY;
ALTER TABLE vault_withdrawals ENABLE ROW LEVEL SECURITY;

-- RLS Policies (users can only see their own data)
CREATE POLICY users_select_own ON users
  FOR SELECT USING (auth.uid()::text = id::text);

CREATE POLICY user_prefs_all_own ON user_preferences
  FOR ALL USING (auth.uid()::text = user_id::text);

CREATE POLICY exchange_creds_all_own ON exchange_credentials
  FOR ALL USING (auth.uid()::text = user_id::text);

CREATE POLICY positions_all_own ON positions
  FOR ALL USING (auth.uid()::text = user_id::text);

CREATE POLICY arb_positions_all_own ON arbitrage_positions
  FOR ALL USING (auth.uid()::text = user_id::text);

CREATE POLICY orders_all_own ON orders
  FOR ALL USING (auth.uid()::text = user_id::text);

CREATE POLICY fills_select_own ON order_fills
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM orders 
      WHERE orders.id = order_fills.order_id 
      AND orders.user_id::text = auth.uid()::text
    )
  );

CREATE POLICY vault_deposits_all_own ON vault_deposits
  FOR ALL USING (auth.uid()::text = user_id::text);

CREATE POLICY vault_withdrawals_all_own ON vault_withdrawals
  FOR ALL USING (auth.uid()::text = user_id::text);

-- Funding rates are public (read-only)
CREATE POLICY funding_rates_select_all ON funding_rates
  FOR SELECT USING (true);

-- Functions for updated_at triggers
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply updated_at triggers
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_user_prefs_updated_at BEFORE UPDATE ON user_preferences
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_exchange_creds_updated_at BEFORE UPDATE ON exchange_credentials
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_positions_updated_at BEFORE UPDATE ON positions
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_arb_positions_updated_at BEFORE UPDATE ON arbitrage_positions
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_orders_updated_at BEFORE UPDATE ON orders
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

**File: `/services/supabase/config.toml`**
```toml
project_id = "your-project-id"

[api]
enabled = true
port = 54321
schemas = ["public", "storage", "graphql_public"]
extra_search_path = ["public", "extensions"]
max_rows = 1000

[db]
port = 54322
shadow_port = 54320
major_version = 15

[studio]
enabled = true
port = 54323
api_url = "http://localhost"

[auth]
enabled = true
site_url = "http://localhost:3000"
additional_redirect_urls = ["https://app.bitfrost.ai"]
jwt_expiry = 3600
enable_signup = true

[auth.external.custom]
enabled = true
client_id = "bitfrost"
secret = "your-secret-key"
url = "https://app.bitfrost.ai/api/auth"

[realtime]
enabled = true
```

#### Success Criteria
- [ ] All migrations apply without errors
- [ ] RLS policies tested and working
- [ ] Indexes created for performance
- [ ] Foreign keys enforce referential integrity
- [ ] Triggers update timestamps correctly

---

### Task 3.2: Supabase Edge Functions - API Server
**Priority**: P0  
**Estimated Complexity**: High  
**Dependencies**: Task 3.1

#### Objective
Implement Supabase Edge Functions for all API endpoints including exchange integration, order execution via Tread.fi, and data aggregation.

#### Core Server Function

**File: `/services/supabase/functions/server/index.ts`**
```typescript
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';
import { corsHeaders } from '../_shared/cors.ts';

// Import route handlers
import { handleHealth } from './routes/health.ts';
import { handleFundingRates } from './routes/funding-rates.ts';
import { handlePositions } from './routes/positions.ts';
import { handleOrders } from './routes/orders.ts';
import { handlePortfolio } from './routes/portfolio.ts';
import { handleExchanges } from './routes/exchanges.ts';
import { handleVault } from './routes/vault.ts';

const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
);

serve(async (req) => {
  // Handle CORS preflight
  if (req.method === 'OPTIONS') {
    return new Response('ok', { headers: corsHeaders });
  }

  try {
    const url = new URL(req.url);
    const path = url.pathname.replace('/functions/v1/server/', '');
    
    // Route handling
    if (path.startsWith('health')) {
      return handleHealth(req);
    }
    
    if (path.startsWith('funding-rates')) {
      return handleFundingRates(req, supabase);
    }
    
    if (path.startsWith('positions')) {
      return handlePositions(req, supabase);
    }
    
    if (path.startsWith('orders')) {
      return handleOrders(req, supabase);
    }
    
    if (path.startsWith('portfolio')) {
      return handlePortfolio(req, supabase);
    }
    
    if (path.startsWith('exchanges')) {
      return handleExchanges(req, supabase);
    }
    
    if (path.startsWith('vault')) {
      return handleVault(req, supabase);
    }

    return new Response(
      JSON.stringify({ error: 'Not found' }),
      { status: 404, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
    );

  } catch (error) {
    console.error('Server error:', error);
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 500, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
    );
  }
});
```

**File: `/services/supabase/functions/server/routes/orders.ts`**
```typescript
import { SupabaseClient } from 'https://esm.sh/@supabase/supabase-js@2';
import { corsHeaders } from '../../_shared/cors.ts';
import { TreadClient } from '../services/tread.ts';
import { HyperliquidClient } from '../services/hyperliquid.ts';

const tread = new TreadClient(Deno.env.get('TREAD_API_KEY')!);

export async function handleOrders(req: Request, supabase: SupabaseClient) {
  const url = new URL(req.url);
  const segments = url.pathname.split('/');
  const action = segments[segments.length - 1];

  // Get user from auth header
  const authHeader = req.headers.get('Authorization');
  if (!authHeader) {
    return new Response(
      JSON.stringify({ error: 'Unauthorized' }),
      { status: 401, headers: corsHeaders }
    );
  }

  const { data: { user }, error: authError } = await supabase.auth.getUser(
    authHeader.replace('Bearer ', '')
  );

  if (authError || !user) {
    return new Response(
      JSON.stringify({ error: 'Invalid token' }),
      { status: 401, headers: corsHeaders }
    );
  }

  switch (action) {
    case 'create':
      return createOrder(req, supabase, user.id);
    case 'create-multi':
      return createMultiLegOrder(req, supabase, user.id);
    case 'cancel':
      return cancelOrder(req, supabase, user.id);
    case 'list':
      return listOrders(req, supabase, user.id);
    case 'get':
      return getOrder(req, supabase, user.id);
    default:
      return new Response(
        JSON.stringify({ error: 'Unknown action' }),
        { status: 400, headers: corsHeaders }
      );
  }
}

/**
 * Create single order
 */
async function createOrder(
  req: Request,
  supabase: SupabaseClient,
  userId: string
) {
  const body = await req.json();
  const {
    exchangeId,
    symbol,
    type,
    side,
    quantity,
    price
  } = body;

  // Validate inputs
  if (!exchangeId || !symbol || !type || !side || !quantity) {
    return new Response(
      JSON.stringify({ error: 'Missing required fields' }),
      { status: 400, headers: corsHeaders }
    );
  }

  // Generate client order ID
  const clientOrderId = `${userId}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

  // Create draft order in DB
  const { data: order, error: dbError } = await supabase
    .from('orders')
    .insert({
      user_id: userId,
      exchange_id: exchangeId,
      symbol,
      type,
      side,
      quantity,
      price,
      status: 'PENDING',
      client_order_id: clientOrderId
    })
    .select()
    .single();

  if (dbError) {
    console.error('DB error:', dbError);
    return new Response(
      JSON.stringify({ error: 'Failed to create order' }),
      { status: 500, headers: corsHeaders }
    );
  }

  try {
    // Submit order via Tread.fi
    const treadOrder = await tread.createOrder({
      exchange: exchangeId,
      symbol,
      side: side.toLowerCase(),
      type: type.toLowerCase(),
      quantity,
      price: type === 'LIMIT' ? price : undefined,
      clientOrderId
    });

    // Update order with exchange ID
    await supabase
      .from('orders')
      .update({
        exchange_order_id: treadOrder.orderId,
        status: 'OPEN'
      })
      .eq('id', order.id);

    return new Response(
      JSON.stringify({
        success: true,
        data: {
          ...order,
          exchange_order_id: treadOrder.orderId,
          status: 'OPEN'
        }
      }),
      { status: 200, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
    );

  } catch (error) {
    // Mark order as failed
    await supabase
      .from('orders')
      .update({ status: 'FAILED' })
      .eq('id', order.id);

    console.error('Order execution error:', error);
    return new Response(
      JSON.stringify({ error: `Order failed: ${error.message}` }),
      { status: 500, headers: corsHeaders }
    );
  }
}

/**
 * Create multi-leg order (for arbitrage)
 */
async function createMultiLegOrder(
  req: Request,
  supabase: SupabaseClient,
  userId: string
) {
  const body = await req.json();
  const { legs } = body;

  if (!legs || legs.length !== 2) {
    return new Response(
      JSON.stringify({ error: 'Multi-leg order requires exactly 2 legs' }),
      { status: 400, headers: corsHeaders }
    );
  }

  const [buyLeg, sellLeg] = legs;

  // Validate opposite sides
  if (buyLeg.side !== 'BUY' || sellLeg.side !== 'SELL') {
    return new Response(
      JSON.stringify({ error: 'Invalid leg sides' }),
      { status: 400, headers: corsHeaders }
    );
  }

  try {
    // Create both orders in DB
    const clientOrderId1 = `${userId}-${Date.now()}-1`;
    const clientOrderId2 = `${userId}-${Date.now()}-2`;

    const { data: orders, error: dbError } = await supabase
      .from('orders')
      .insert([
        {
          user_id: userId,
          exchange_id: buyLeg.exchangeId,
          symbol: buyLeg.symbol,
          type: buyLeg.type,
          side: 'BUY',
          quantity: buyLeg.quantity,
          price: buyLeg.price,
          status: 'PENDING',
          client_order_id: clientOrderId1,
          pair_order_id: clientOrderId2
        },
        {
          user_id: userId,
          exchange_id: sellLeg.exchangeId,
          symbol: sellLeg.symbol,
          type: sellLeg.type,
          side: 'SELL',
          quantity: sellLeg.quantity,
          price: sellLeg.price,
          status: 'PENDING',
          client_order_id: clientOrderId2,
          pair_order_id: clientOrderId1
        }
      ])
      .select();

    if (dbError) {
      throw new Error(`DB error: ${dbError.message}`);
    }

    // Execute both legs simultaneously via Tread.fi
    const [buyResult, sellResult] = await Promise.allSettled([
      tread.createOrder({
        exchange: buyLeg.exchangeId,
        symbol: buyLeg.symbol,
        side: 'buy',
        type: buyLeg.type.toLowerCase(),
        quantity: buyLeg.quantity,
        price: buyLeg.type === 'LIMIT' ? buyLeg.price : undefined,
        clientOrderId: clientOrderId1
      }),
      tread.createOrder({
        exchange: sellLeg.exchangeId,
        symbol: sellLeg.symbol,
        side: 'sell',
        type: sellLeg.type.toLowerCase(),
        quantity: sellLeg.quantity,
        price: sellLeg.type === 'LIMIT' ? sellLeg.price : undefined,
        clientOrderId: clientOrderId2
      })
    ]);

    // Check if both succeeded
    const buySuccess = buyResult.status === 'fulfilled';
    const sellSuccess = sellResult.status === 'fulfilled';

    if (!buySuccess || !sellSuccess) {
      // If either failed, cancel the successful one
      if (buySuccess) {
        await tread.cancelOrder({
          exchange: buyLeg.exchangeId,
          orderId: (buyResult as PromiseFulfilledResult<any>).value.orderId
        });
      }
      if (sellSuccess) {
        await tread.cancelOrder({
          exchange: sellLeg.exchangeId,
          orderId: (sellResult as PromiseFulfilledResult<any>).value.orderId
        });
      }

      // Update orders as failed
      await supabase
        .from('orders')
        .update({ status: 'FAILED' })
        .in('id', orders!.map(o => o.id));

      throw new Error('One or more legs failed to execute');
    }

    // Update both orders with exchange IDs
    await supabase
      .from('orders')
      .update({
        exchange_order_id: (buyResult as PromiseFulfilledResult<any>).value.orderId,
        status: 'OPEN'
      })
      .eq('id', orders![0].id);

    await supabase
      .from('orders')
      .update({
        exchange_order_id: (sellResult as PromiseFulfilledResult<any>).value.orderId,
        status: 'OPEN'
      })
      .eq('id', orders![1].id);

    return new Response(
      JSON.stringify({
        success: true,
        data: {
          orders: orders!.map((o, i) => ({
            ...o,
            exchange_order_id: i === 0 
              ? (buyResult as PromiseFulfilledResult<any>).value.orderId
              : (sellResult as PromiseFulfilledResult<any>).value.orderId,
            status: 'OPEN'
          }))
        }
      }),
      { status: 200, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
    );

  } catch (error) {
    console.error('Multi-leg order error:', error);
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 500, headers: corsHeaders }
    );
  }
}

/**
 * Cancel order
 */
async function cancelOrder(
  req: Request,
  supabase: SupabaseClient,
  userId: string
) {
  const body = await req.json();
  const { orderId } = body;

  // Get order
  const { data: order, error: fetchError } = await supabase
    .from('orders')
    .select('*')
    .eq('id', orderId)
    .eq('user_id', userId)
    .single();

  if (fetchError || !order) {
    return new Response(
      JSON.stringify({ error: 'Order not found' }),
      { status: 404, headers: corsHeaders }
    );
  }

  try {
    // Cancel via Tread.fi
    await tread.cancelOrder({
      exchange: order.exchange_id,
      orderId: order.exchange_order_id
    });

    // Update order status
    await supabase
      .from('orders')
      .update({ status: 'CANCELLED' })
      .eq('id', orderId);

    return new Response(
      JSON.stringify({ success: true }),
      { status: 200, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
    );

  } catch (error) {
    console.error('Cancel order error:', error);
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 500, headers: corsHeaders }
    );
  }
}

/**
 * List user's orders
 */
async function listOrders(
  req: Request,
  supabase: SupabaseClient,
  userId: string
) {
  const url = new URL(req.url);
  const status = url.searchParams.get('status');
  const limit = parseInt(url.searchParams.get('limit') || '50');
  const offset = parseInt(url.searchParams.get('offset') || '0');

  let query = supabase
    .from('orders')
    .select('*, order_fills(*)')
    .eq('user_id', userId)
    .order('created_at', { ascending: false })
    .range(offset, offset + limit - 1);

  if (status) {
    query = query.eq('status', status);
  }

  const { data, error } = await query;

  if (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 500, headers: corsHeaders }
    );
  }

  return new Response(
    JSON.stringify({ success: true, data }),
    { status: 200, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
  );
}

/**
 * Get single order
 */
async function getOrder(
  req: Request,
  supabase: SupabaseClient,
  userId: string
) {
  const url = new URL(req.url);
  const orderId = url.searchParams.get('id');

  const { data, error } = await supabase
    .from('orders')
    .select('*, order_fills(*)')
    .eq('id', orderId)
    .eq('user_id', userId)
    .single();

  if (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 404, headers: corsHeaders }
    );
  }

  return new Response(
    JSON.stringify({ success: true, data }),
    { status: 200, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
  );
}
```

**File: `/services/supabase/functions/server/services/tread.ts`**
```typescript
/**
 * Tread.fi OMS Client
 * Handles order execution across all supported exchanges
 */
export class TreadClient {
  private apiKey: string;
  private baseUrl = 'https://api.tread.fi/v1';

  constructor(apiKey: string) {
    this.apiKey = apiKey;
  }

  /**
   * Create order on any exchange
   */
  async createOrder(params: {
    exchange: string;
    symbol: string;
    side: 'buy' | 'sell';
    type: 'market' | 'limit' | 'twap';
    quantity: number;
    price?: number;
    clientOrderId: string;
  }): Promise<{ orderId: string; status: string }> {
    const response = await fetch(`${this.baseUrl}/orders`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        exchange: params.exchange,
        symbol: params.symbol,
        side: params.side,
        type: params.type,
        quantity: params.quantity.toString(),
        price: params.price?.toString(),
        client_order_id: params.clientOrderId
      })
    });

    if (!response.ok) {
      const error = await response.text();
      throw new Error(`Tread API error: ${error}`);
    }

    return response.json();
  }

  /**
   * Cancel order
   */
  async cancelOrder(params: {
    exchange: string;
    orderId: string;
  }): Promise<void> {
    const response = await fetch(
      `${this.baseUrl}/orders/${params.orderId}/cancel`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ exchange: params.exchange })
      }
    );

    if (!response.ok) {
      const error = await response.text();
      throw new Error(`Cancel order failed: ${error}`);
    }
  }

  /**
   * Get order status
   */
  async getOrderStatus(params: {
    exchange: string;
    orderId: string;
  }): Promise<any> {
    const response = await fetch(
      `${this.baseUrl}/orders/${params.orderId}?exchange=${params.exchange}`,
      {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`
        }
      }
    );

    if (!response.ok) {
      throw new Error('Failed to get order status');
    }

    return response.json();
  }

  /**
   * Get positions on exchange
   */
  async getPositions(exchange: string): Promise<any[]> {
    const response = await fetch(
      `${this.baseUrl}/positions?exchange=${exchange}`,
      {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`
        }
      }
    );

    if (!response.ok) {
      throw new Error('Failed to get positions');
    }

    return response.json();
  }

  /**
   * Get account balance on exchange
   */
  async getBalance(exchange: string): Promise<any> {
    const response = await fetch(
      `${this.baseUrl}/balance?exchange=${exchange}`,
      {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`
        }
      }
    );

    if (!response.ok) {
      throw new Error('Failed to get balance');
    }

    return response.json();
  }
}
```

This is getting very long. I'll continue creating the complete task breakdown document with all remaining phases...

#### Success Criteria
- [ ] All API endpoints functional
- [ ] Tread.fi integration works for all exchanges
- [ ] Order execution completes in <2 seconds
- [ ] Error handling covers all edge cases
- [ ] WebSocket connections stable

---

### Task 3.3: Real-time Data System - WebSocket Server
**Priority**: P0  
**Estimated Complexity**: High  
**Dependencies**: Task 3.1, 3.2

#### Objective
Implement WebSocket server for real-time funding rate updates, position changes, and price data streaming.

#### Implementation

**File: `/services/supabase/functions/websocket/index.ts`**
```typescript
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
);

// Connected clients
const clients = new Map<string, WebSocket>();

serve(async (req) => {
  if (req.headers.get('upgrade') !== 'websocket') {
    return new Response('Expected websocket', { status: 400 });
  }

  const { socket, response } = Deno.upgradeWebSocket(req);
  const clientId = crypto.randomUUID();

  socket.onopen = () => {
    console.log('Client connected:', clientId);
    clients.set(clientId, socket);
  };

  socket.onmessage = async (event) => {
    const message = JSON.parse(event.data);
    await handleMessage(clientId, socket, message);
  };

  socket.onclose = () => {
    console.log('Client disconnected:', clientId);
    clients.delete(clientId);
  };

  socket.onerror = (error) => {
    console.error('WebSocket error:', error);
    clients.delete(clientId);
  };

  return response;
});

async function handleMessage(
  clientId: string,
  socket: WebSocket,
  message: any
) {
  const { type, data } = message;

  switch (type) {
    case 'SUBSCRIBE_FUNDING':
      await subscribeFundingRates(clientId, socket, data);
      break;
    case 'SUBSCRIBE_POSITIONS':
      await subscribePositions(clientId, socket, data);
      break;
    case 'SUBSCRIBE_ORDERS':
      await subscribeOrders(clientId, socket, data);
      break;
    case 'UNSUBSCRIBE':
      // Handle unsubscribe
      break;
    default:
      socket.send(JSON.stringify({ error: 'Unknown message type' }));
  }
}

async function subscribeFundingRates(
  clientId: string,
  socket: WebSocket,
  data: { userId: string }
) {
  // Subscribe to funding_rates table changes
  const channel = supabase
    .channel('funding-rates')
    .on(
      'postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        table: 'funding_rates'
      },
      (payload) => {
        socket.send(JSON.stringify({
          type: 'FUNDING_RATE_UPDATE',
          data: payload.new
        }));
      }
    )
    .subscribe();

  // Send initial data
  const { data: rates } = await supabase
    .from('funding_rates')
    .select('*')
    .order('timestamp', { ascending: false })
    .limit(100);

  socket.send(JSON.stringify({
    type: 'FUNDING_RATES_SNAPSHOT',
    data: rates
  }));
}

async function subscribePositions(
  clientId: string,
  socket: WebSocket,
  data: { userId: string }
) {
  // Subscribe to positions table changes for this user
  const channel = supabase
    .channel(`positions-${data.userId}`)
    .on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'positions',
        filter: `user_id=eq.${data.userId}`
      },
      (payload) => {
        socket.send(JSON.stringify({
          type: 'POSITION_UPDATE',
          data: payload.new || payload.old
        }));
      }
    )
    .subscribe();

  // Send initial positions
  const { data: positions } = await supabase
    .from('positions')
    .select('*')
    .eq('user_id', data.userId)
    .is('closed_at', null);

  socket.send(JSON.stringify({
    type: 'POSITIONS_SNAPSHOT',
    data: positions
  }));
}

async function subscribeOrders(
  clientId: string,
  socket: WebSocket,
  data: { userId: string }
) {
  // Subscribe to orders table changes for this user
  const channel = supabase
    .channel(`orders-${data.userId}`)
    .on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'orders',
        filter: `user_id=eq.${data.userId}`
      },
      (payload) => {
        socket.send(JSON.stringify({
          type: 'ORDER_UPDATE',
          data: payload.new || payload.old
        }));
      }
    )
    .subscribe();
}
```

**File: `/services/worker/funding-rate-fetcher.ts`**
```typescript
/**
 * Background worker to fetch funding rates from all exchanges
 * Runs every 5 minutes
 */
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

const EXCHANGES = [
  'hyperliquid',
  'paradex',
  'binance',
  'bybit',
  'okx',
  'aster',
  'xyz',
  'vntl',
  'km',
  'cash',
  'flx',
  'hyna'
];

const SYMBOLS = ['BTC-PERP', 'ETH-PERP', 'SOL-PERP', 'ARB-PERP', /* ... */];

async function fetchFundingRates() {
  console.log('Fetching funding rates...');
  
  for (const exchange of EXCHANGES) {
    for (const symbol of SYMBOLS) {
      try {
        const rate = await getFundingRate(exchange, symbol);
        
        if (rate) {
          // Insert into database
          await supabase.from('funding_rates').insert({
            exchange_id: exchange,
            symbol,
            rate: rate.rate,
            timestamp: new Date().toISOString(),
            next_funding_time: rate.nextFundingTime,
            daily_rate: rate.rate * 3,
            annualized_rate: ((1 + rate.rate) ** 1095) - 1
          });
        }
      } catch (error) {
        console.error(`Error fetching ${exchange} ${symbol}:`, error);
      }
    }
  }
  
  console.log('Funding rates updated');
}

async function getFundingRate(exchange: string, symbol: string) {
  // Implementation varies by exchange
  // Use Loris API for aggregated data or individual exchange APIs
  
  if (exchange === 'hyperliquid') {
    const response = await fetch('https://api.hyperliquid.xyz/info', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        type: 'metaAndAssetCtxs'
      })
    });
    
    const data = await response.json();
    const asset = data[0].find((a: any) => a.name === symbol.replace('-PERP', ''));
    
    return {
      rate: parseFloat(asset.funding),
      nextFundingTime: new Date(Date.now() + 8 * 60 * 60 * 1000).toISOString()
    };
  }
  
  // Add similar implementations for other exchanges...
  return null;
}

// Run every 5 minutes
setInterval(fetchFundingRates, 5 * 60 * 1000);

// Run immediately on start
fetchFundingRates();
```

#### Success Criteria
- [ ] WebSocket connections stable for 24+ hours
- [ ] Real-time updates arrive within 100ms
- [ ] Handles 1000+ concurrent connections
- [ ] Automatic reconnection on disconnect
- [ ] Funding rates update every 5 minutes

---

## PHASE 4: Frontend Foundation

### Task 4.1: Next.js App Setup with Web3
**Priority**: P0  
**Estimated Complexity**: Medium  
**Dependencies**: Task 1.1, 1.2

#### Objective
Set up Next.js 14 application with App Router, RainbowKit, Wagmi, and TailwindCSS.

#### Implementation

**File: `/apps/web/package.json`**
```json
{
  "name": "@bitfrost/web",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^14.1.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@rainbow-me/rainbowkit": "^2.0.0",
    "wagmi": "^2.5.0",
    "viem": "^2.7.0",
    "@tanstack/react-query": "^5.17.0",
    "@supabase/supabase-js": "^2.39.0",
    "zustand": "^4.5.0",
    "recharts": "^2.10.0",
    "lucide-react": "^0.316.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.2.0",
    "date-fns": "^3.0.0",
    "sonner": "^1.3.0"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "typescript": "^5",
    "tailwindcss": "^3.4.0",
    "autoprefixer": "^10",
    "postcss": "^8",
    "eslint": "^8",
    "eslint-config-next": "14.1.0"
  }
}
```

**File: `/apps/web/app/layout.tsx`**
```typescript
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';
import { Providers } from './providers';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Bitfrost | Funding Rate Arbitrage Platform',
  description: 'Professional-grade funding rate arbitrage across 12 crypto exchanges',
  keywords: 'crypto, arbitrage, funding rates, trading, DeFi',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className="dark">
      <body className={inter.className}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

**File: `/apps/web/app/providers.tsx`**
```typescript
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { WagmiProvider } from 'wagmi';
import { RainbowKitProvider, darkTheme } from '@rainbow-me/rainbowkit';
import { Toaster } from 'sonner';
import { wagmiConfig } from '@/lib/wagmi';
import '@rainbow-me/rainbowkit/styles.css';

const queryClient = new QueryClient();

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={wagmiConfig}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider theme={darkTheme()}>
          {children}
          <Toaster position="bottom-right" />
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}
```

**File: `/apps/web/lib/wagmi.ts`**
```typescript
import { getDefaultConfig } from '@rainbow-me/rainbowkit';
import { arbitrum } from 'wagmi/chains';

export const wagmiConfig = getDefaultConfig({
  appName: 'Bitfrost',
  projectId: process.env.NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID!,
  chains: [arbitrum],
  ssr: true,
});
```

**File: `/apps/web/tailwind.config.ts`**
```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: 'class',
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        success: 'hsl(var(--success))',
        warning: 'hsl(var(--warning))',
        error: 'hsl(var(--error))',
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [],
};

export default config;
```

**File: `/apps/web/app/globals.css`**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
    --success: 142.1 76.2% 36.3%;
    --warning: 38 92% 50%;
    --error: 0 72.2% 50.6%;
    --radius: 0.5rem;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

#### Success Criteria
- [ ] App loads without errors
- [ ] Wallet connection works
- [ ] Dark theme applies correctly
- [ ] Hot reload functional
- [ ] TypeScript strict mode enabled

---

### Task 4.2: Zustand State Management
**Priority**: P0  
**Estimated Complexity**: Medium  
**Dependencies**: Task 4.1

#### Objective
Implement Zustand stores for global state management (user, positions, orders, funding rates).

#### Implementation

**File: `/apps/web/lib/store/user.ts`**
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import type { User, Portfolio } from '@bitfrost/types';

interface UserState {
  user: User | null;
  portfolio: Portfolio | null;
  isLoading: boolean;
  error: string | null;
  
  // Actions
  setUser: (user: User | null) => void;
  setPortfolio: (portfolio: Portfolio) => void;
  updateBalance: (exchangeId: string, balance: number) => void;
  logout: () => void;
}

export const useUserStore = create<UserState>()(
  persist(
    (set) => ({
      user: null,
      portfolio: null,
      isLoading: false,
      error: null,

      setUser: (user) => set({ user }),
      
      setPortfolio: (portfolio) => set({ portfolio }),
      
      updateBalance: (exchangeId, balance) =>
        set((state) => {
          if (!state.portfolio) return state;
          
          const updatedBalances = state.portfolio.exchangeBalances.map((eb) =>
            eb.exchangeId === exchangeId
              ? { ...eb, totalBalance: balance }
              : eb
          );
          
          return {
            portfolio: {
              ...state.portfolio,
              exchangeBalances: updatedBalances,
              totalEquity: updatedBalances.reduce((sum, eb) => sum + eb.totalBalance, 0),
            },
          };
        }),
      
      logout: () => set({ user: null, portfolio: null }),
    }),
    {
      name: 'user-storage',
      partialize: (state) => ({ user: state.user }),
    }
  )
);
```

**File: `/apps/web/lib/store/positions.ts`**
```typescript
import { create } from 'zustand';
import type { Position, ArbitragePosition } from '@bitfrost/types';

interface PositionsState {
  positions: Position[];
  arbitragePositions: ArbitragePosition[];
  isLoading: boolean;
  error: string | null;
  
  // Actions
  setPositions: (positions: Position[]) => void;
  updatePosition: (position: Position) => void;
  removePosition: (positionId: string) => void;
  setArbitragePositions: (positions: ArbitragePosition[]) => void;
}

export const usePositionsStore = create<PositionsState>((set) => ({
  positions: [],
  arbitragePositions: [],
  isLoading: false,
  error: null,

  setPositions: (positions) => set({ positions }),
  
  updatePosition: (updatedPosition) =>
    set((state) => ({
      positions: state.positions.map((p) =>
        p.id === updatedPosition.id ? updatedPosition : p
      ),
    })),
  
  removePosition: (positionId) =>
    set((state) => ({
      positions: state.positions.filter((p) => p.id !== positionId),
    })),
  
  setArbitragePositions: (arbitragePositions) => set({ arbitragePositions }),
}));
```

**File: `/apps/web/lib/store/funding.ts`**
```typescript
import { create } from 'zustand';
import type { FundingRate, FundingSpread } from '@bitfrost/types';

interface FundingState {
  rates: FundingRate[];
  spreads: FundingSpread[];
  selectedCells: { exchange: string; symbol: string }[];
  isLoading: boolean;
  error: string | null;
  
  // Actions
  setRates: (rates: FundingRate[]) => void;
  updateRate: (rate: FundingRate) => void;
  calculateSpreads: () => void;
  selectCell: (exchange: string, symbol: string) => void;
  clearSelection: () => void;
}

export const useFundingStore = create<FundingState>((set, get) => ({
  rates: [],
  spreads: [],
  selectedCells: [],
  isLoading: false,
  error: null,

  setRates: (rates) => {
    set({ rates });
    get().calculateSpreads();
  },
  
  updateRate: (updatedRate) =>
    set((state) => {
      const rates = state.rates.map((r) =>
        r.exchangeId === updatedRate.exchangeId && r.symbol === updatedRate.symbol
          ? updatedRate
          : r
      );
      
      // Recalculate spreads
      setTimeout(() => get().calculateSpreads(), 0);
      
      return { rates };
    }),
  
  calculateSpreads: () => {
    const { rates } = get();
    const spreads: FundingSpread[] = [];
    
    // Group rates by symbol
    const bySymbol: Record<string, FundingRate[]> = {};
    rates.forEach((rate) => {
      if (!bySymbol[rate.symbol]) bySymbol[rate.symbol] = [];
      bySymbol[rate.symbol].push(rate);
    });
    
    // Calculate spreads for each symbol
    Object.entries(bySymbol).forEach(([symbol, symbolRates]) => {
      for (let i = 0; i < symbolRates.length; i++) {
        for (let j = i + 1; j < symbolRates.length; j++) {
          const rate1 = symbolRates[i];
          const rate2 = symbolRates[j];
          
          const spread = Math.abs(rate1.rate - rate2.rate);
          const annualizedSpread = ((1 + spread) ** 1095) - 1;
          
          if (spread > 0.0001) { // Minimum 0.01% spread
            spreads.push({
              symbol,
              longExchange: rate1.rate < rate2.rate ? rate1.exchangeId : rate2.exchangeId,
              shortExchange: rate1.rate < rate2.rate ? rate2.exchangeId : rate1.exchangeId,
              longRate: Math.min(rate1.rate, rate2.rate),
              shortRate: Math.max(rate1.rate, rate2.rate),
              spread,
              annualizedSpread,
              isFeasible: spread > 0.001, // >0.1% spread required
              minCapital: 1000,
              expectedDailyReturn: spread * 3,
              breakEvenDays: 0.01 / (spread * 3),
              calculatedAt: Date.now(),
              expiresAt: Date.now() + 8 * 60 * 60 * 1000,
            });
          }
        }
      }
    });
    
    // Sort by spread descending
    spreads.sort((a, b) => b.spread - a.spread);
    
    set({ spreads });
  },
  
  selectCell: (exchange, symbol) =>
    set((state) => {
      const selected = state.selectedCells;
      
      // If this cell is already selected, deselect it
      if (selected.some((s) => s.exchange === exchange && s.symbol === symbol)) {
        return {
          selectedCells: selected.filter(
            (s) => !(s.exchange === exchange && s.symbol === symbol)
          ),
        };
      }
      
      // If 2 cells already selected, replace oldest with new selection
      if (selected.length >= 2) {
        return {
          selectedCells: [selected[1], { exchange, symbol }],
        };
      }
      
      // Add to selection
      return {
        selectedCells: [...selected, { exchange, symbol }],
      };
    }),
  
  clearSelection: () => set({ selectedCells: [] }),
}));
```

#### Success Criteria
- [ ] State persists across refreshes (where appropriate)
- [ ] Store updates trigger component re-renders
- [ ] No unnecessary re-renders
- [ ] TypeScript types enforced
- [ ] Actions work correctly

---

### Task 4.3: API Client & Hooks
**Priority**: P0  
**Estimated Complexity**: Medium  
**Dependencies**: Task 3.2, 4.1

#### Objective
Create API client for backend communication and React Query hooks for data fetching.

#### Implementation

**File: `/apps/web/lib/api/client.ts`**
```typescript
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

export class ApiClient {
  private baseUrl: string;

  constructor() {
    this.baseUrl = process.env.NEXT_PUBLIC_API_URL || 
      `${process.env.NEXT_PUBLIC_SUPABASE_URL}/functions/v1/server`;
  }

  private async getAuthToken(): Promise<string> {
    const { data: { session } } = await supabase.auth.getSession();
    if (!session) throw new Error('Not authenticated');
    return session.access_token;
  }

  private async request<T>(
    endpoint: string,
    options?: RequestInit
  ): Promise<T> {
    const token = await this.getAuthToken();
    
    const response = await fetch(`${this.baseUrl}/${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`,
        ...options?.headers,
      },
    });

    if (!response.ok) {
      const error = await response.json().catch(() => ({ message: 'Unknown error' }));
      throw new Error(error.message || `HTTP ${response.status}`);
    }

    const result = await response.json();
    return result.data as T;
  }

  // Funding rates
  async getFundingRates(params?: {
    exchange?: string;
    symbol?: string;
    limit?: number;
  }) {
    const query = new URLSearchParams(params as any).toString();
    return this.request<any[]>(`funding-rates?${query}`);
  }

  // Positions
  async getPositions() {
    return this.request<any[]>('positions/list');
  }

  async getArbitragePositions() {
    return this.request<any[]>('positions/arbitrage');
  }

  // Orders
  async createOrder(params: {
    exchangeId: string;
    symbol: string;
    type: string;
    side: string;
    quantity: number;
    price?: number;
  }) {
    return this.request('orders/create', {
      method: 'POST',
      body: JSON.stringify(params),
    });
  }

  async createMultiLegOrder(params: { legs: any[] }) {
    return this.request('orders/create-multi', {
      method: 'POST',
      body: JSON.stringify(params),
    });
  }

  async cancelOrder(orderId: string) {
    return this.request('orders/cancel', {
      method: 'POST',
      body: JSON.stringify({ orderId }),
    });
  }

  async getOrders(params?: { status?: string; limit?: number }) {
    const query = new URLSearchParams(params as any).toString();
    return this.request<any[]>(`orders/list?${query}`);
  }

  // Portfolio
  async getPortfolio() {
    return this.request<any>('portfolio');
  }

  async getExchangeBalances() {
    return this.request<any[]>('portfolio/balances');
  }

  // Exchanges
  async getExchangeStatus(exchangeId: string) {
    return this.request(`exchanges/${exchangeId}/status`);
  }

  async saveExchangeCredentials(params: {
    exchangeId: string;
    apiKey: string;
    apiSecret: string;
    passphrase?: string;
  }) {
    return this.request('exchanges/credentials', {
      method: 'POST',
      body: JSON.stringify(params),
    });
  }
}

export const apiClient = new ApiClient();
```

**File: `/apps/web/lib/hooks/use-funding-rates.ts`**
```typescript
import { useQuery, useQueryClient } from '@tanstack/react-query';
import { useEffect } from 'react';
import { apiClient, supabase } from '@/lib/api/client';
import { useFundingStore } from '@/lib/store/funding';

export function useFundingRates() {
  const queryClient = useQueryClient();
  const { setRates, updateRate } = useFundingStore();

  const query = useQuery({
    queryKey: ['funding-rates'],
    queryFn: () => apiClient.getFundingRates(),
    refetchInterval: 5 * 60 * 1000, // Refetch every 5 minutes
  });

  // Update store when data changes
  useEffect(() => {
    if (query.data) {
      setRates(query.data);
    }
  }, [query.data, setRates]);

  // Subscribe to realtime updates
  useEffect(() => {
    const channel = supabase
      .channel('funding-rates')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'funding_rates',
        },
        (payload) => {
          updateRate(payload.new as any);
          queryClient.invalidateQueries({ queryKey: ['funding-rates'] });
        }
      )
      .subscribe();

    return () => {
      channel.unsubscribe();
    };
  }, [updateRate, queryClient]);

  return query;
}
```

**File: `/apps/web/lib/hooks/use-positions.ts`**
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useEffect } from 'react';
import { apiClient, supabase } from '@/lib/api/client';
import { usePositionsStore } from '@/lib/store/positions';
import { useUserStore } from '@/lib/store/user';

export function usePositions() {
  const queryClient = useQueryClient();
  const { setPositions, updatePosition } = usePositionsStore();
  const { user } = useUserStore();

  const query = useQuery({
    queryKey: ['positions'],
    queryFn: () => apiClient.getPositions(),
    enabled: !!user,
    refetchInterval: 30000, // Refetch every 30 seconds
  });

  // Update store
  useEffect(() => {
    if (query.data) {
      setPositions(query.data);
    }
  }, [query.data, setPositions]);

  // Subscribe to realtime updates
  useEffect(() => {
    if (!user) return;

    const channel = supabase
      .channel(`positions-${user.id}`)
      .on(
        'postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: 'positions',
          filter: `user_id=eq.${user.id}`,
        },
        (payload) => {
          if (payload.new) {
            updatePosition(payload.new as any);
          }
          queryClient.invalidateQueries({ queryKey: ['positions'] });
        }
      )
      .subscribe();

    return () => {
      channel.unsubscribe();
    };
  }, [user, updatePosition, queryClient]);

  return query;
}

export function useArbitragePositions() {
  const { user } = useUserStore();

  return useQuery({
    queryKey: ['arbitrage-positions'],
    queryFn: () => apiClient.getArbitragePositions(),
    enabled: !!user,
    refetchInterval: 30000,
  });
}
```

**File: `/apps/web/lib/hooks/use-orders.ts`**
```typescript
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import { toast } from 'sonner';
import { apiClient } from '@/lib/api/client';
import { useUserStore } from '@/lib/store/user';

export function useOrders(params?: { status?: string }) {
  const { user } = useUserStore();

  return useQuery({
    queryKey: ['orders', params],
    queryFn: () => apiClient.getOrders(params),
    enabled: !!user,
  });
}

export function useCreateOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: apiClient.createOrder.bind(apiClient),
    onSuccess: () => {
      toast.success('Order created successfully');
      queryClient.invalidateQueries({ queryKey: ['orders'] });
      queryClient.invalidateQueries({ queryKey: ['positions'] });
    },
    onError: (error: Error) => {
      toast.error(`Failed to create order: ${error.message}`);
    },
  });
}

export function useCreateMultiLegOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: apiClient.createMultiLegOrder.bind(apiClient),
    onSuccess: () => {
      toast.success('Multi-leg order executed successfully');
      queryClient.invalidateQueries({ queryKey: ['orders'] });
      queryClient.invalidateQueries({ queryKey: ['positions'] });
    },
    onError: (error: Error) => {
      toast.error(`Failed to execute order: ${error.message}`);
    },
  });
}

export function useCancelOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (orderId: string) => apiClient.cancelOrder(orderId),
    onSuccess: () => {
      toast.success('Order cancelled');
      queryClient.invalidateQueries({ queryKey: ['orders'] });
    },
    onError: (error: Error) => {
      toast.error(`Failed to cancel order: ${error.message}`);
    },
  });
}
```

#### Success Criteria
- [ ] API calls work with authentication
- [ ] React Query caching effective
- [ ] Realtime updates work
- [ ] Error handling comprehensive
- [ ] TypeScript types correct

---

## PHASE 5: Core Features

### Task 5.1: Explore Page - Funding Rates Table
**Priority**: P0  
**Estimated Complexity**: High  
**Dependencies**: Task 4.1, 4.2, 4.3

#### Objective
Build the Explore page with interactive funding rates table supporting cell selection for arbitrage identification.

#### Implementation

**File: `/apps/web/app/(dashboard)/explore/page.tsx`**
```typescript
'use client';

import { useFundingRates } from '@/lib/hooks/use-funding-rates';
import { useFundingStore } from '@/lib/store/funding';
import { FundingRatesTable } from '@/components/funding-rates-table';
import { SpreadCalculator } from '@/components/spread-calculator';
import { PageHeader } from '@/components/page-header';
import { Card } from '@/components/ui/card';

export default function ExplorePage() {
  const { isLoading, error } = useFundingRates();
  const { selectedCells, spreads } = useFundingStore();

  const selectedSpread = selectedCells.length === 2
    ? spreads.find(
        (s) =>
          (s.longExchange === selectedCells[0].exchange &&
            s.shortExchange === selectedCells[1].exchange &&
            s.symbol === selectedCells[0].symbol) ||
          (s.longExchange === selectedCells[1].exchange &&
            s.shortExchange === selectedCells[0].exchange &&
            s.symbol === selectedCells[1].symbol)
      )
    : null;

  return (
    <div className="container py-8">
      <PageHeader
        title="Explore Opportunities"
        description="Click two cells to identify arbitrage spreads"
      />

      {selectedSpread && (
        <Card className="mb-6">
          <SpreadCalculator spread={selectedSpread} />
        </Card>
      )}

      <Card>
        <FundingRatesTable
          isLoading={isLoading}
          error={error?.message}
        />
      </Card>
    </div>
  );
}
```

**File: `/apps/web/components/funding-rates-table.tsx`**
```typescript
'use client';

import { useFundingStore } from '@/lib/store/funding';
import { cn } from '@/lib/utils';
import { ArrowUpDown } from 'lucide-react';
import { useState } from 'react';

const EXCHANGES = [
  'hyperliquid',
  'paradex',
  'binance',
  'bybit',
  'okx',
  'aster',
  'xyz',
  'vntl',
  'km',
  'cash',
  'flx',
  'hyna',
];

const SYMBOLS = [
  'BTC-PERP',
  'ETH-PERP',
  'SOL-PERP',
  'ARB-PERP',
  'AVAX-PERP',
  'DOGE-PERP',
  'MATIC-PERP',
  'OP-PERP',
];

export function FundingRatesTable({
  isLoading,
  error,
}: {
  isLoading: boolean;
  error?: string;
}) {
  const { rates, selectedCells, selectCell } = useFundingStore();
  const [sortBy, setSortBy] = useState<'symbol' | 'rate'>('symbol');
  const [sortOrder, setSortOrder] = useState<'asc' | 'desc'>('asc');

  const getRateForCell = (exchange: string, symbol: string) => {
    return rates.find(
      (r) => r.exchangeId === exchange && r.symbol === symbol
    );
  };

  const isCellSelected = (exchange: string, symbol: string) => {
    return selectedCells.some(
      (s) => s.exchange === exchange && s.symbol === symbol
    );
  };

  const handleCellClick = (exchange: string, symbol: string) => {
    selectCell(exchange, symbol);
  };

  const formatRate = (rate: number) => {
    const percentage = rate * 100;
    return percentage.toFixed(4) + '%';
  };

  const formatAnnualized = (rate: number) => {
    const annualized = ((1 + rate) ** 1095 - 1) * 100;
    return annualized.toFixed(2) + '%';
  };

  const getCellColor = (rate: number) => {
    if (rate > 0.001) return 'bg-success/20 text-success';
    if (rate < -0.001) return 'bg-error/20 text-error';
    return 'bg-muted/50 text-muted-foreground';
  };

  if (error) {
    return (
      <div className="p-8 text-center text-error">
        <p>Error loading funding rates: {error}</p>
      </div>
    );
  }

  if (isLoading) {
    return (
      <div className="p-8">
        <div className="animate-pulse space-y-4">
          {[...Array(8)].map((_, i) => (
            <div key={i} className="h-12 bg-muted/50 rounded" />
          ))}
        </div>
      </div>
    );
  }

  return (
    <div className="relative overflow-x-auto">
      <table className="w-full border-collapse">
        <thead>
          <tr className="border-b border-border">
            <th className="sticky left-0 z-20 bg-background p-3 text-left font-medium">
              <button
                onClick={() => {
                  setSortBy('symbol');
                  setSortOrder(sortOrder === 'asc' ? 'desc' : 'asc');
                }}
                className="flex items-center gap-2 hover:text-primary"
              >
                Symbol
                <ArrowUpDown className="h-4 w-4" />
              </button>
            </th>
            {EXCHANGES.map((exchange) => (
              <th
                key={exchange}
                className="p-3 text-left font-medium capitalize"
              >
                {exchange}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {SYMBOLS.map((symbol) => (
            <tr key={symbol} className="border-b border-border/50">
              <td className="sticky left-0 z-10 bg-background p-3 font-medium">
                {symbol}
              </td>
              {EXCHANGES.map((exchange) => {
                const rate = getRateForCell(exchange, symbol);
                const isSelected = isCellSelected(exchange, symbol);

                return (
                  <td
                    key={`${exchange}-${symbol}`}
                    className={cn(
                      'p-3 cursor-pointer transition-all hover:ring-2 hover:ring-primary',
                      isSelected && 'ring-2 ring-primary',
                      rate && getCellColor(rate.rate)
                    )}
                    onClick={() => handleCellClick(exchange, symbol)}
                  >
                    {rate ? (
                      <div className="space-y-1">
                        <div className="font-mono text-sm font-semibold">
                          {formatRate(rate.rate)}
                        </div>
                        <div className="text-xs opacity-70">
                          {formatAnnualized(rate.rate)} APY
                        </div>
                      </div>
                    ) : (
                      <div className="text-muted-foreground text-xs">N/A</div>
                    )}
                  </td>
                );
              })}
            </tr>
          ))}
        </tbody>
      </table>

      {selectedCells.length > 0 && (
        <div className="sticky bottom-0 left-0 right-0 bg-card border-t border-border p-4">
          <div className="flex items-center justify-between">
            <div className="text-sm text-muted-foreground">
              Selected: {selectedCells.length} cell{selectedCells.length > 1 ? 's' : ''}
            </div>
            {selectedCells.length === 2 && (
              <div className="text-sm font-medium">
                Click "Trade" to execute arbitrage
              </div>
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

**File: `/apps/web/components/spread-calculator.tsx`**
```typescript
'use client';

import { type FundingSpread } from '@bitfrost/types';
import { Button } from '@/components/ui/button';
import { useRouter } from 'next/navigation';
import { TrendingUp, ArrowRight } from 'lucide-react';

export function SpreadCalculator({ spread }: { spread: FundingSpread }) {
  const router = useRouter();

  const handleTrade = () => {
    // Navigate to trade page with pre-filled order
    router.push(
      `/trade?long=${spread.longExchange}&short=${spread.shortExchange}&symbol=${spread.symbol}`
    );
  };

  return (
    <div className="p-6 space-y-4">
      <div className="flex items-center justify-between">
        <h3 className="text-lg font-semibold flex items-center gap-2">
          <TrendingUp className="h-5 w-5 text-success" />
          Arbitrage Opportunity Detected
        </h3>
        <Button onClick={handleTrade}>
          Execute Trade
          <ArrowRight className="ml-2 h-4 w-4" />
        </Button>
      </div>

      <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
        <div>
          <div className="text-sm text-muted-foreground">Symbol</div>
          <div className="text-2xl font-bold">{spread.symbol}</div>
        </div>

        <div>
          <div className="text-sm text-muted-foreground">Spread</div>
          <div className="text-2xl font-bold text-success">
            {(spread.spread * 100).toFixed(4)}%
          </div>
          <div className="text-xs text-muted-foreground">
            {(spread.annualizedSpread * 100).toFixed(2)}% APY
          </div>
        </div>

        <div>
          <div className="text-sm text-muted-foreground">Long on</div>
          <div className="text-xl font-semibold capitalize">
            {spread.longExchange}
          </div>
          <div className="text-xs text-success">
            +{(spread.longRate * 100).toFixed(4)}%
          </div>
        </div>

        <div>
          <div className="text-sm text-muted-foreground">Short on</div>
          <div className="text-xl font-semibold capitalize">
            {spread.shortExchange}
          </div>
          <div className="text-xs text-error">
            {(spread.shortRate * 100).toFixed(4)}%
          </div>
        </div>
      </div>

      <div className="border-t pt-4">
        <div className="grid grid-cols-3 gap-4 text-sm">
          <div>
            <div className="text-muted-foreground">Expected Daily Return</div>
            <div className="font-semibold">
              {(spread.expectedDailyReturn * 100).toFixed(4)}%
            </div>
          </div>
          <div>
            <div className="text-muted-foreground">Min Capital</div>
            <div className="font-semibold">${spread.minCapital}</div>
          </div>
          <div>
            <div className="text-muted-foreground">Break-even</div>
            <div className="font-semibold">{spread.breakEvenDays.toFixed(1)} days</div>
          </div>
        </div>
      </div>
    </div>
  );
}
```

#### Success Criteria
- [ ] Table renders all funding rates
- [ ] Cell selection works (max 2 cells)
- [ ] Spread calculation accurate
- [ ] Color coding intuitive
- [ ] Navigation to trade page works
- [ ] Real-time updates visible

---

### Task 5.2: Trade Page - Multi-Leg Order Execution
**Priority**: P0  
**Estimated Complexity**: High  
**Dependencies**: Task 5.1

#### Objective
Build trade page with dual-panel order form for executing delta-neutral arbitrage positions.

#### Implementation

**File: `/apps/web/app/(dashboard)/trade/page.tsx`**
```typescript
'use client';

import { useState, useEffect } from 'react';
import { useSearchParams } from 'next/navigation';
import { PageHeader } from '@/components/page-header';
import { Card } from '@/components/ui/card';
import { OrderPanel } from '@/components/order-panel';
import { OrderSummary } from '@/components/order-summary';
import { Button } from '@/components/ui/button';
import { useCreateMultiLegOrder } from '@/lib/hooks/use-orders';
import { toast } from 'sonner';

export default function TradePage() {
  const searchParams = useSearchParams();
  const createOrder = useCreateMultiLegOrder();

  const [buyOrder, setBuyOrder] = useState({
    exchangeId: searchParams.get('long') || '',
    symbol: searchParams.get('symbol') || 'BTC-PERP',
    type: 'MARKET' as const,
    quantity: 0,
    price: undefined as number | undefined,
  });

  const [sellOrder, setSellOrder] = useState({
    exchangeId: searchParams.get('short') || '',
    symbol: searchParams.get('symbol') || 'BTC-PERP',
    type: 'MARKET' as const,
    quantity: 0,
    price: undefined as number | undefined,
  });

  // Sync quantities
  useEffect(() => {
    if (buyOrder.quantity !== sellOrder.quantity) {
      setSellOrder((prev) => ({ ...prev, quantity: buyOrder.quantity }));
    }
  }, [buyOrder.quantity]);

  const handleSubmit = async () => {
    if (!buyOrder.exchangeId || !sellOrder.exchangeId) {
      toast.error('Please select both exchanges');
      return;
    }

    if (buyOrder.quantity === 0) {
      toast.error('Please enter a quantity');
      return;
    }

    try {
      await createOrder.mutateAsync({
        legs: [
          {
            ...buyOrder,
            side: 'BUY',
          },
          {
            ...sellOrder,
            side: 'SELL',
          },
        ],
      });

      // Reset form
      setBuyOrder({
        exchangeId: '',
        symbol: 'BTC-PERP',
        type: 'MARKET',
        quantity: 0,
        price: undefined,
      });
      setSellOrder({
        exchangeId: '',
        symbol: 'BTC-PERP',
        type: 'MARKET',
        quantity: 0,
        price: undefined,
      });
    } catch (error) {
      // Error handled by mutation
    }
  };

  return (
    <div className="container py-8">
      <PageHeader
        title="Execute Arbitrage Trade"
        description="Set up delta-neutral positions across exchanges"
      />

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
        <Card>
          <OrderPanel
            title="Buy (Long)"
            side="BUY"
            order={buyOrder}
            onChange={setBuyOrder}
          />
        </Card>

        <Card>
          <OrderPanel
            title="Sell (Short)"
            side="SELL"
            order={sellOrder}
            onChange={setSellOrder}
          />
        </Card>
      </div>

      <Card>
        <OrderSummary
          buyOrder={buyOrder}
          sellOrder={sellOrder}
          onSubmit={handleSubmit}
          isLoading={createOrder.isPending}
        />
      </Card>
    </div>
  );
}
```

**File: `/apps/web/components/order-panel.tsx`**
```typescript
'use client';

import { Label } from '@/components/ui/label';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Input } from '@/components/ui/input';

const EXCHANGES = [
  { id: 'hyperliquid', name: 'Hyperliquid' },
  { id: 'paradex', name: 'Paradex' },
  { id: 'binance', name: 'Binance' },
  { id: 'bybit', name: 'Bybit' },
  // Add others...
];

const SYMBOLS = ['BTC-PERP', 'ETH-PERP', 'SOL-PERP', 'ARB-PERP'];
const ORDER_TYPES = ['MARKET', 'LIMIT', 'TWAP'];

interface OrderPanelProps {
  title: string;
  side: 'BUY' | 'SELL';
  order: {
    exchangeId: string;
    symbol: string;
    type: string;
    quantity: number;
    price?: number;
  };
  onChange: (order: any) => void;
}

export function OrderPanel({ title, side, order, onChange }: OrderPanelProps) {
  return (
    <div className="p-6 space-y-4">
      <h3 className="text-lg font-semibold">{title}</h3>

      <div className="space-y-4">
        <div>
          <Label>Exchange</Label>
          <Select
            value={order.exchangeId}
            onValueChange={(value) =>
              onChange({ ...order, exchangeId: value })
            }
          >
            <SelectTrigger>
              <SelectValue placeholder="Select exchange" />
            </SelectTrigger>
            <SelectContent>
              {EXCHANGES.map((ex) => (
                <SelectItem key={ex.id} value={ex.id}>
                  {ex.name}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>

        <div>
          <Label>Symbol</Label>
          <Select
            value={order.symbol}
            onValueChange={(value) => onChange({ ...order, symbol: value })}
          >
            <SelectTrigger>
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              {SYMBOLS.map((symbol) => (
                <SelectItem key={symbol} value={symbol}>
                  {symbol}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>

        <div>
          <Label>Order Type</Label>
          <Select
            value={order.type}
            onValueChange={(value) => onChange({ ...order, type: value })}
          >
            <SelectTrigger>
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              {ORDER_TYPES.map((type) => (
                <SelectItem key={type} value={type}>
                  {type}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>

        <div>
          <Label>Quantity</Label>
          <Input
            type="number"
            step="0.001"
            value={order.quantity || ''}
            onChange={(e) =>
              onChange({ ...order, quantity: parseFloat(e.target.value) || 0 })
            }
            placeholder="0.00"
          />
        </div>

        {order.type === 'LIMIT' && (
          <div>
            <Label>Price</Label>
            <Input
              type="number"
              step="0.01"
              value={order.price || ''}
              onChange={(e) =>
                onChange({
                  ...order,
                  price: parseFloat(e.target.value) || undefined,
                })
              }
              placeholder="0.00"
            />
          </div>
        )}
      </div>
    </div>
  );
}
```

**File: `/apps/web/components/order-summary.tsx`**
```typescript
'use client';

import { Button } from '@/components/ui/button';
import { Loader2 } from 'lucide-react';
import { useFundingStore } from '@/lib/store/funding';

interface OrderSummaryProps {
  buyOrder: any;
  sellOrder: any;
  onSubmit: () => void;
  isLoading: boolean;
}

export function OrderSummary({
  buyOrder,
  sellOrder,
  onSubmit,
  isLoading,
}: OrderSummaryProps) {
  const { rates } = useFundingStore();

  const buyRate = rates.find(
    (r) => r.exchangeId === buyOrder.exchangeId && r.symbol === buyOrder.symbol
  );

  const sellRate = rates.find(
    (r) => r.exchangeId === sellOrder.exchangeId && r.symbol === sellOrder.symbol
  );

  const spread = buyRate && sellRate ? sellRate.rate - buyRate.rate : 0;
  const estimatedDailyReturn = spread * 3 * buyOrder.quantity * 50000; // Assuming $50k BTC

  const isValid =
    buyOrder.exchangeId &&
    sellOrder.exchangeId &&
    buyOrder.quantity > 0 &&
    buyOrder.symbol === sellOrder.symbol;

  return (
    <div className="p-6 space-y-4">
      <h3 className="text-lg font-semibold">Order Summary</h3>

      <div className="grid grid-cols-2 gap-4">
        <div>
          <div className="text-sm text-muted-foreground">Long Position</div>
          <div className="font-semibold capitalize">{buyOrder.exchangeId || 'N/A'}</div>
          <div className="text-xs">{buyOrder.quantity} {buyOrder.symbol}</div>
        </div>

        <div>
          <div className="text-sm text-muted-foreground">Short Position</div>
          <div className="font-semibold capitalize">{sellOrder.exchangeId || 'N/A'}</div>
          <div className="text-xs">{sellOrder.quantity} {sellOrder.symbol}</div>
        </div>
      </div>

      <div className="border-t pt-4 space-y-2">
        <div className="flex justify-between">
          <span className="text-muted-foreground">Spread</span>
          <span className="font-mono font-semibold text-success">
            {(spread * 100).toFixed(4)}%
          </span>
        </div>

        <div className="flex justify-between">
          <span className="text-muted-foreground">Estimated Daily Return</span>
          <span className="font-mono font-semibold">
            ${estimatedDailyReturn.toFixed(2)}
          </span>
        </div>

        <div className="flex justify-between">
          <span className="text-muted-foreground">Net Exposure</span>
          <span className="font-mono">~$0 (Delta Neutral)</span>
        </div>
      </div>

      <Button
        size="lg"
        className="w-full"
        onClick={onSubmit}
        disabled={!isValid || isLoading}
      >
        {isLoading ? (
          <>
            <Loader2 className="mr-2 h-4 w-4 animate-spin" />
            Executing...
          </>
        ) : (
          'Execute Multi-Leg Order'
        )}
      </Button>
    </div>
  );
}
```

#### Success Criteria
- [ ] Order panels pre-fill from URL params
- [ ] Quantities stay synced
- [ ] Spread calculation accurate
- [ ] Order execution works
- [ ] Success/error feedback clear
- [ ] Form resets after submission

---

### Task 5.3: Portfolio Page - Position Monitoring
**Priority**: P1  
**Estimated Complexity**: High  
**Dependencies**: Task 4.3

#### Objective
Build portfolio page with position tracking, PnL monitoring, and exchange balances.

**File: `/apps/web/app/(dashboard)/portfolio/page.tsx`**
```typescript
'use client';

import { usePortfolio } from '@/lib/hooks/use-portfolio';
import { usePositions, useArbitragePositions } from '@/lib/hooks/use-positions';
import { PageHeader } from '@/components/page-header';
import { Card } from '@/components/ui/card';
import { PortfolioSummary } from '@/components/portfolio-summary';
import { ExchangeBalances } from '@/components/exchange-balances';
import { PositionsTable } from '@/components/positions-table';
import { ArbitragePositionsTable } from '@/components/arbitrage-positions-table';

export default function PortfolioPage() {
  const { data: portfolio, isLoading: portfolioLoading } = usePortfolio();
  const { data: positions, isLoading: positionsLoading } = usePositions();
  const { data: arbPositions } = useArbitragePositions();

  return (
    <div className="container py-8 space-y-6">
      <PageHeader
        title="Portfolio"
        description="Track your positions and performance"
      />

      <PortfolioSummary
        portfolio={portfolio}
        isLoading={portfolioLoading}
      />

      <Card>
        <ExchangeBalances
          balances={portfolio?.exchangeBalances || []}
          isLoading={portfolioLoading}
        />
      </Card>

      <Card>
        <div className="p-6">
          <h3 className="text-lg font-semibold mb-4">Arbitrage Positions</h3>
          <ArbitragePositionsTable
            positions={arbPositions || []}
            isLoading={positionsLoading}
          />
        </div>
      </Card>

      <Card>
        <div className="p-6">
          <h3 className="text-lg font-semibold mb-4">All Positions</h3>
          <PositionsTable
            positions={positions || []}
            isLoading={positionsLoading}
          />
        </div>
      </Card>
    </div>
  );
}
```

*Due to length constraints, I'll provide the key components structure:*

**Components needed:**
- `PortfolioSummary` - Total equity, PnL, ROI display
- `ExchangeBalances` - Pie chart + table of balances per exchange
- `PositionsTable` - Sortable table of all positions
- `ArbitragePositionsTable` - Paired positions with spread tracking

#### Success Criteria
- [ ] Real-time PnL updates
- [ ] Exchange balances accurate
- [ ] Position details complete
- [ ] Charts render correctly
- [ ] Quick actions functional

---

## PHASE 6: Integration Layer

### Task 6.1: Hyperliquid HIP-3 DEX Integration
**Priority**: P1  
**Estimated Complexity**: Medium  
**Dependencies**: Task 3.2

#### Objective
Integrate HIP-3 DEXs (XYZ, VNTL, KM, CASH, FLX, HYNA) for additional trading venues.

**File: `/services/supabase/functions/server/services/hyperliquid-hip3.ts`**
```typescript
/**
 * Hyperliquid HIP-3 DEX Client
 * Handles trading on Hyperliquid-based DEXs
 */
export class Hip3Client {
  private dexName: string;
  private baseUrl = 'https://api.hyperliquid.xyz';

  constructor(dexName: string) {
    this.dexName = dexName;
  }

  async getMarkets(): Promise<any[]> {
    const response = await fetch(`${this.baseUrl}/info`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        type: 'metaAndAssetCtxs',
        user: this.dexName // HIP-3 DEX identifier
      })
    });

    return response.json();
  }

  async createOrder(params: {
    symbol: string;
    side: 'buy' | 'sell';
    quantity: number;
    price?: number;
  }): Promise<any> {
    // Use Hyperliquid SDK to create order on specific HIP-3 DEX
    // Implementation depends on SDK
  }

  async getFundingRate(symbol: string): Promise<number> {
    const markets = await this.getMarkets();
    const market = markets.find(m => m.name === symbol);
    return market?.funding || 0;
  }
}
```

#### Success Criteria
- [ ] All 6 HIP-3 DEXs integrated
- [ ] Funding rates fetched correctly
- [ ] Order execution works
- [ ] Position tracking functional

---

### Task 6.2: Tread.fi OMS Integration
**Priority**: P0  
**Estimated Complexity**: High  
**Dependencies**: Task 3.2

#### Objective
Complete integration with Tread.fi for unified order management across all exchanges.

Already implemented in Task 3.2 (`/services/supabase/functions/server/services/tread.ts`)

**Additional requirements:**
- Webhook handler for order status updates
- Position sync from Tread.fi
- Balance sync from all connected exchanges
- Error recovery mechanisms

**File: `/services/supabase/functions/webhooks/tread-updates.ts`**
```typescript
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const supabase = createClient(
  Deno.env.get('SUPABASE_URL')!,
  Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
);

serve(async (req) => {
  const signature = req.headers.get('x-tread-signature');
  
  // Verify webhook signature
  if (!verifySignature(signature, await req.text())) {
    return new Response('Unauthorized', { status: 401 });
  }

  const payload = await req.json();

  switch (payload.type) {
    case 'order.filled':
      await handleOrderFilled(payload.data);
      break;
    case 'order.cancelled':
      await handleOrderCancelled(payload.data);
      break;
    case 'position.updated':
      await handlePositionUpdated(payload.data);
      break;
  }

  return new Response('OK', { status: 200 });
});

async function handleOrderFilled(data: any) {
  await supabase
    .from('orders')
    .update({
      status: 'FILLED',
      filled_quantity: data.quantity,
      average_fill_price: data.averagePrice
    })
    .eq('client_order_id', data.clientOrderId);

  // Create fill record
  await supabase.from('order_fills').insert({
    order_id: data.orderId,
    quantity: data.quantity,
    price: data.price,
    trade_id: data.tradeId,
    fee: data.fee
  });
}

function verifySignature(signature: string, body: string): boolean {
  // Implement signature verification
  return true;
}
```

#### Success Criteria
- [ ] Order execution <2 seconds
- [ ] Status updates real-time
- [ ] Error handling comprehensive
- [ ] Webhook verification secure

---

## PHASE 7: Testing & Quality

### Task 7.1: Unit Testing - All Components
**Priority**: P1  
**Estimated Complexity**: High  
**Dependencies**: All feature tasks

#### Test Coverage Requirements
- **Contracts**: 100% coverage
- **Backend**: 90%+ coverage
- **Frontend Components**: 80%+ coverage
- **Integration Tests**: All critical paths

**File: `/packages/contracts/test/Vault.test.ts`**
*(Already provided in Task 2.1)*

**File: `/apps/web/__tests__/components/funding-rates-table.test.tsx`**
```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { FundingRatesTable } from '@/components/funding-rates-table';
import { useFundingStore } from '@/lib/store/funding';

jest.mock('@/lib/store/funding');

describe('FundingRatesTable', () => {
  beforeEach(() => {
    (useFundingStore as any).mockReturnValue({
      rates: [
        {
          exchangeId: 'hyperliquid',
          symbol: 'BTC-PERP',
          rate: 0.0005,
          annualizedRate: 0.5475
        },
        {
          exchangeId: 'paradex',
          symbol: 'BTC-PERP',
          rate: -0.0003,
          annualizedRate: -0.3285
        }
      ],
      selectedCells: [],
      selectCell: jest.fn()
    });
  });

  it('renders funding rates table', () => {
    render(<FundingRatesTable isLoading={false} />);
    expect(screen.getByText('BTC-PERP')).toBeInTheDocument();
  });

  it('selects cell on click', () => {
    const selectCell = jest.fn();
    (useFundingStore as any).mockReturnValue({
      rates: [],
      selectedCells: [],
      selectCell
    });

    render(<FundingRatesTable isLoading={false} />);
    
    const cell = screen.getAllByRole('cell')[1];
    fireEvent.click(cell);
    
    expect(selectCell).toHaveBeenCalled();
  });

  // Add more tests...
});
```

#### Success Criteria
- [ ] All test suites pass
- [ ] Coverage thresholds met
- [ ] CI/CD integrated
- [ ] E2E tests cover user journeys

---

### Task 7.2: Integration Testing
**Priority**: P1  
**Estimated Complexity**: Medium  
**Dependencies**: Task 7.1

**Test scenarios:**
1. Complete user journey: Wallet connect → Explore → Trade → Portfolio
2. Order execution flow
3. Real-time updates
4. Error recovery

**File: `/apps/web/__tests__/integration/user-journey.test.tsx`**
```typescript
import { test, expect } from '@playwright/test';

test.describe('User Journey', () => {
  test('complete arbitrage trade flow', async ({ page }) => {
    // 1. Navigate to app
    await page.goto('http://localhost:3000');

    // 2. Connect wallet
    await page.click('text=Connect Wallet');
    // ... wallet connection steps

    // 3. Go to Explore page
    await page.click('text=Explore');
    await expect(page).toHaveURL('/explore');

    // 4. Select two cells
    await page.click('[data-cell="hyperliquid-BTC-PERP"]');
    await page.click('[data-cell="paradex-BTC-PERP"]');

    // 5. Navigate to Trade
    await page.click('text=Execute Trade');
    await expect(page).toHaveURL(/\/trade/);

    // 6. Enter quantity
    await page.fill('[name="quantity"]', '0.1');

    // 7. Submit order
    await page.click('text=Execute Multi-Leg Order');

    // 8. Verify success
    await expect(page.locator('text=Order executed successfully')).toBeVisible();

    // 9. Check portfolio
    await page.click('text=Portfolio');
    await expect(page.locator('text=0.1 BTC-PERP')).toBeVisible();
  });
});
```

#### Success Criteria
- [ ] All integration tests pass
- [ ] Performance benchmarks met
- [ ] Error scenarios handled
- [ ] Cross-browser tested

---

## PHASE 8: Deployment & Operations

### Task 8.1: Production Deployment
**Priority**: P0  
**Estimated Complexity**: Medium  
**Dependencies**: All previous tasks

#### Deployment Checklist

**1. Smart Contracts**
```bash
# Deploy Vault to Arbitrum mainnet
cd packages/contracts
npx hardhat run scripts/deploy-vault.ts --network arbitrum

# Verify on Arbiscan
npx hardhat verify --network arbitrum <VAULT_ADDRESS>
```

**2. Supabase**
```bash
# Link to production project
supabase link --project-ref <PROJECT_ID>

# Run migrations
supabase db push

# Deploy Edge Functions
supabase functions deploy server
supabase functions deploy websocket
supabase functions deploy webhooks/tread-updates

# Set secrets
supabase secrets set \
  TREAD_API_KEY=xxx \
  HIP3_DEX_NAME=xyz \
  LORIS_API_KEY=xxx
```

**3. Frontend (Vercel)**
```bash
# Build
cd apps/web
npm run build

# Deploy
vercel --prod

# Set environment variables in Vercel dashboard:
# - NEXT_PUBLIC_SUPABASE_URL
# - NEXT_PUBLIC_SUPABASE_ANON_KEY
# - NEXT_PUBLIC_WALLET_CONNECT_PROJECT_ID
# - NEXT_PUBLIC_VAULT_ADDRESS
```

**4. Workers**
```bash
# Deploy funding rate fetcher
cd services/worker
docker build -t bitfrost-worker .
docker push bitfrost-worker

# Deploy to Cloud Run / Railway / Fly.io
```

#### Success Criteria
- [ ] All services deployed
- [ ] Environment variables set
- [ ] Domain configured (app.bitfrost.ai)
- [ ] SSL certificates active
- [ ] Monitoring enabled

---

### Task 8.2: Monitoring & Observability
**Priority**: P1  
**Estimated Complexity**: Medium  
**Dependencies**: Task 8.1

#### Tools Setup
- **Sentry**: Error tracking
- **Datadog/New Relic**: APM
- **Grafana**: Dashboards
- **PagerDuty**: Alerts

**File: `/apps/web/lib/monitoring/sentry.ts`**
```typescript
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  beforeSend(event, hint) {
    // Filter out sensitive data
    if (event.request?.headers) {
      delete event.request.headers['authorization'];
    }
    return event;
  },
});

export { Sentry };
```

**Monitoring Dashboards:**
1. **User Metrics**: Active users, new signups, retention
2. **Trading Metrics**: Orders placed, success rate, avg execution time
3. **Technical Metrics**: API latency, error rate, uptime
4. **Financial Metrics**: TVL, volume, fees collected

#### Success Criteria
- [ ] Error tracking operational
- [ ] Performance monitoring active
- [ ] Alerts configured
- [ ] Dashboards accessible

---

### Task 8.3: Documentation & User Guides
**Priority**: P2  
**Estimated Complexity**: Low  
**Dependencies**: Task 8.1

#### Documentation Required
1. **User Guide** - How to use the platform
2. **API Documentation** - For developers
3. **Smart Contract Docs** - Contract interactions
4. **Deployment Guide** - For operators
5. **Troubleshooting** - Common issues

**File: `/apps/docs/pages/index.mdx`**
```mdx
# Bitfrost Documentation

Welcome to Bitfrost, the professional funding rate arbitrage platform.

## Quick Start

1. **Connect Your Wallet**
   - Click "Connect Wallet" in the top right
   - Approve the connection in your wallet

2. **Deposit Funds**
   - Navigate to Portfolio
   - Click "Deposit"
   - Send USDC to your vault address

3. **Find Opportunities**
   - Go to the Explore page
   - Click two cells to identify spreads
   - Click "Trade" to execute

4. **Monitor Positions**
   - View your positions in Portfolio
   - Track PnL and funding collected

## Features

### Explore Page
[Details...]

### Trade Page
[Details...]

### Portfolio Page
[Details...]
```

#### Success Criteria
- [ ] All features documented
- [ ] User guide complete
- [ ] API docs published
- [ ] Video tutorials created

---

## Implementation Timeline

### Week 1-2: Foundation
- ✅ Task 1.1: Monorepo setup
- ✅ Task 1.2: TypeScript configuration
- ✅ Task 2.1: Vault contract
- ✅ Task 2.2: Vault SDK

### Week 3-4: Backend
- ✅ Task 3.1: Supabase setup
- ✅ Task 3.2: API server
- ✅ Task 3.3: WebSocket system
- ✅ Task 6.2: Tread.fi integration

### Week 5-6: Frontend Foundation
- ✅ Task 4.1: Next.js setup
- ✅ Task 4.2: State management
- ✅ Task 4.3: API client & hooks

### Week 7-8: Core Features
- ✅ Task 5.1: Explore page
- ✅ Task 5.2: Trade page
- ✅ Task 5.3: Portfolio page

### Week 9-10: Integration & Testing
- ✅ Task 6.1: HIP-3 integration
- ✅ Task 7.1: Unit tests
- ✅ Task 7.2: Integration tests

### Week 11-12: Deployment & Launch
- ✅ Task 8.1: Production deployment
- ✅ Task 8.2: Monitoring setup
- ✅ Task 8.3: Documentation

**Total Timeline: 12 weeks (3 months)**

---

## Critical Success Factors

1. **Security First**
   - Smart contract audit before mainnet
   - API key encryption
   - Rate limiting
   - Input validation

2. **Performance**
   - Order execution <2 seconds
   - API latency <200ms
   - WebSocket updates <100ms
   - Page load <1 second

3. **Reliability**
   - 99.9%+ uptime
   - Automatic failover
   - Data backup strategy
   - Disaster recovery plan

4. **User Experience**
   - Intuitive interface
   - Clear error messages
   - Responsive design
   - Comprehensive help

5. **Compliance**
   - Terms of service
   - Privacy policy
   - KYC/AML considerations
   - Regulatory compliance

---

## Next Steps for AI Implementation

1. **Start with Phase 1** - Set up project structure
2. **Move to Phase 2** - Deploy smart contracts to testnet
3. **Build Phase 3** - Backend infrastructure
4. **Develop Phase 4 & 5** - Frontend features
5. **Integrate Phase 6** - External services
6. **Test Phase 7** - Comprehensive testing
7. **Deploy Phase 8** - Production launch

Each task is designed to be:
- **Self-contained**: Can be completed independently
- **Well-defined**: Clear inputs, outputs, and success criteria
- **Testable**: Success criteria are measurable
- **Production-ready**: Includes error handling, security, and performance

---

## Appendix: File Structure Summary

```
bitfrost/
├── packages/
│   ├── contracts/          # Smart contracts (Vault)
│   ├── sdk/                # Contract interaction SDK
│   ├── types/              # Shared TypeScript types
│   └── ui/                 # Shared React components
├── apps/
│   ├── web/                # Next.js frontend
│   └── docs/               # Documentation site
├── services/
│   ├── supabase/           # Backend (Edge Functions, migrations)
│   └── worker/             # Background jobs (funding rate fetcher)
└── scripts/                # Deployment & utility scripts
```

**Total Files to Create**: ~150+  
**Total Lines of Code**: ~25,000+  
**Estimated AI Implementation Time**: 12 weeks with parallel task execution

---

*This task breakdown provides a complete roadmap for implementing the Bitfrost platform from scratch with production-ready code, comprehensive testing, and operational excellence.*
