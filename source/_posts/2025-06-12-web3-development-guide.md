---
title: Web3开发初探：智能合约与DApp搭建
date: 2025-06-12 15:00:00
updated: 2025-06-12 15:00:00
description: 'Web3开发入门实战，使用Solidity编写智能合约，Hardhat框架搭建DApp，包含完整的ERC20代币和NFT合约代码'
keywords: 'Web3,智能合约,Solidity,DApp,Hardhat,ERC20,NFT,区块链开发'
tags: 
  - Web3
  - 智能合约
  - Solidity
  - DApp
categories:
  - 技术笔记
  - 区块链
author: Yi Chen
toc: true
comments: true
---

> ⛓️ "代码即法律，在区块链上成为现实。"

## 🤔 为什么学习Web3开发

作为一个程序员，我对Web3的态度经历了三个阶段：

1. **2021年**：炒作期，觉得都是骗局
2. **2022-2023年**：熊市沉淀期，开始认真研究技术
3. **2024年至今**：理解了去中心化的价值，决定深入学习

**Web3不仅仅是加密货币，它是一种新的计算范式**：
- 代码开源且可验证
- 数据所有权归用户
- 去中心化治理

作为开发者，了解这个领域可以：
- 拓展技术视野
- 抓住新的职业机会
- 理解未来互联网的可能形态

## 🛠️ 技术栈选择

### 开发语言：Solidity

**为什么选择Solidity？**
- 以太坊生态最成熟
- 学习资源丰富
- 职业机会多

**备选**：Rust（Solana）、Move（Aptos/Sui）

### 开发框架：Hardhat

对比：

| 框架 | 特点 | 适合 |
|------|------|------|
| **Hardhat** | 灵活，插件丰富 | 专业开发 ✅ |
| **Foundry** | 快速，Solidity测试 | 高级用户 |
| **Truffle** | 老牌，稳定 | 入门学习 |

我选Hardhat，因为文档好，社区活跃。

## 🚀 环境搭建

### 安装依赖

```bash
# 创建项目目录
mkdir my-web3-project
cd my-web3-project

# 初始化npm项目
npm init -y

# 安装Hardhat
npm install --save-dev hardhat

# 初始化Hardhat项目
npx hardhat init
# 选择: Create a TypeScript project

# 安装其他依赖
npm install --save-dev @nomicfoundation/hardhat-toolbox
npm install dotenv
```

### 项目结构

```
my-web3-project/
├── contracts/          # 智能合约
├── scripts/            # 部署脚本
├── test/               # 测试文件
├── hardhat.config.ts   # Hardhat配置
├── .env                # 环境变量（不提交到git）
└── package.json
```

### Hardhat配置

```typescript
// hardhat.config.ts
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import * as dotenv from "dotenv";

dotenv.config();

const config: HardhatUserConfig = {
  solidity: "0.8.19",
  networks: {
    hardhat: {
      chainId: 1337
    },
    sepolia: {
      url: process.env.SEPOLIA_RPC_URL || "",
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : []
    },
    mainnet: {
      url: process.env.MAINNET_RPC_URL || "",
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : []
    }
  },
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY
  }
};

export default config;
```

## 💡 实战1：ERC20代币合约

### 什么是ERC20

ERC20是以太坊代币标准，定义了代币的基本功能：
- 转账
- 查询余额
- 授权

### 完整代码

```solidity
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is ERC20, Ownable {
    uint256 public constant MAX_SUPPLY = 1000000 * 10**18; // 100万代币
    uint256 public constant INITIAL_SUPPLY = 100000 * 10**18; // 初始发行10万
    
    // 事件定义
    event TokensMinted(address indexed to, uint256 amount);
    event TokensBurned(address indexed from, uint256 amount);
    
    constructor() ERC20("My Token", "MTK") Ownable(msg.sender) {
        _mint(msg.sender, INITIAL_SUPPLY);
    }
    
    // 铸造新代币（仅限合约所有者）
    function mint(address to, uint256 amount) public onlyOwner {
        require(totalSupply() + amount <= MAX_SUPPLY, "Exceeds max supply");
        _mint(to, amount);
        emit TokensMinted(to, amount);
    }
    
    // 销毁代币
    function burn(uint256 amount) public {
        _burn(msg.sender, amount);
        emit TokensBurned(msg.sender, amount);
    }
    
    // 空投功能
    function airdrop(address[] calldata recipients, uint256 amount) external onlyOwner {
        require(recipients.length > 0, "Empty recipients list");
        require(balanceOf(msg.sender) >= amount * recipients.length, "Insufficient balance");
        
        for (uint i = 0; i < recipients.length; i++) {
            _transfer(msg.sender, recipients[i], amount);
        }
    }
    
    // 查询合约ETH余额
    function getContractBalance() external view returns (uint256) {
        return address(this).balance;
    }
    
    // 提取合约ETH（仅限所有者）
    function withdraw() external onlyOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "No ETH to withdraw");
        
        (bool success, ) = payable(owner()).call{value: balance}("");
        require(success, "Transfer failed");
    }
    
    receive() external payable {}
}
```

### 编译合约

```bash
npx hardhat compile
```

### 编写测试

```typescript
// test/MyToken.test.ts
import { expect } from "chai";
import { ethers } from "hardhat";
import { MyToken } from "../typechain-types";

describe("MyToken", function () {
  let myToken: MyToken;
  let owner: any;
  let addr1: any;
  let addr2: any;

  beforeEach(async function () {
    [owner, addr1, addr2] = await ethers.getSigners();
    
    const MyTokenFactory = await ethers.getContractFactory("MyToken");
    myToken = await MyTokenFactory.deploy();
    await myToken.waitForDeployment();
  });

  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      expect(await myToken.owner()).to.equal(owner.address);
    });

    it("Should assign the total supply of tokens to the owner", async function () {
      const ownerBalance = await myToken.balanceOf(owner.address);
      expect(await myToken.totalSupply()).to.equal(ownerBalance);
    });

    it("Should have correct name and symbol", async function () {
      expect(await myToken.name()).to.equal("My Token");
      expect(await myToken.symbol()).to.equal("MTK");
    });
  });

  describe("Transactions", function () {
    it("Should transfer tokens between accounts", async function () {
      await myToken.transfer(addr1.address, 50);
      const addr1Balance = await myToken.balanceOf(addr1.address);
      expect(addr1Balance).to.equal(50);
    });

    it("Should fail if sender doesn't have enough tokens", async function () {
      const initialOwnerBalance = await myToken.balanceOf(owner.address);
      
      await expect(
        myToken.connect(addr1).transfer(owner.address, 1)
      ).to.be.revertedWithCustomError(myToken, "ERC20InsufficientBalance");

      expect(await myToken.balanceOf(owner.address)).to.equal(initialOwnerBalance);
    });
  });

  describe("Minting", function () {
    it("Should allow owner to mint tokens", async function () {
      await myToken.mint(addr1.address, 100);
      expect(await myToken.balanceOf(addr1.address)).to.equal(100);
    });

    it("Should not allow non-owner to mint tokens", async function () {
      await expect(
        myToken.connect(addr1).mint(addr1.address, 100)
      ).to.be.revertedWithCustomError(myToken, "OwnableUnauthorizedAccount");
    });

    it("Should not exceed max supply", async function () {
      const maxSupply = await myToken.MAX_SUPPLY();
      await expect(
        myToken.mint(addr1.address, maxSupply)
      ).to.be.revertedWith("Exceeds max supply");
    });
  });
});
```

运行测试：
```bash
npx hardhat test
```

### 部署脚本

```typescript
// scripts/deploy.ts
import { ethers } from "hardhat";

async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying contracts with the account:", deployer.address);

  const MyToken = await ethers.getContractFactory("MyToken");
  const myToken = await MyToken.deploy();

  await myToken.waitForDeployment();

  console.log("MyToken deployed to:", await myToken.getAddress());
  console.log("Token name:", await myToken.name());
  console.log("Token symbol:", await myToken.symbol());
  console.log("Total supply:", await myToken.totalSupply());
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

部署到本地网络：
```bash
npx hardhat run scripts/deploy.ts
```

部署到测试网：
```bash
npx hardhat run scripts/deploy.ts --network sepolia
```

## 🎨 实战2：NFT合约

### 什么是ERC721

ERC721是非同质化代币标准，每个代币都是唯一的。

应用场景：数字艺术品、游戏道具、身份凭证等。

### NFT合约代码

```solidity
// contracts/MyNFT.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract MyNFT is ERC721, ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;
    
    Counters.Counter private _tokenIdCounter;
    
    uint256 public constant MAX_SUPPLY = 10000;
    uint256 public constant MINT_PRICE = 0.01 ether;
    uint256 public constant MAX_PER_WALLET = 5;
    
    string public baseTokenURI;
    bool public saleIsActive = false;
    
    mapping(address => uint256) public walletMints;
    
    event NFTMinted(address indexed minter, uint256 indexed tokenId);
    event SaleStateChanged(bool isActive);
    
    constructor(string memory _baseURI) ERC721("My NFT Collection", "MNFT") Ownable(msg.sender) {
        baseTokenURI = _baseURI;
    }
    
    function mint() public payable {
        require(saleIsActive, "Sale is not active");
        require(msg.value >= MINT_PRICE, "Insufficient payment");
        require(_tokenIdCounter.current() < MAX_SUPPLY, "Max supply reached");
        require(walletMints[msg.sender] < MAX_PER_WALLET, "Max per wallet reached");
        
        uint256 tokenId = _tokenIdCounter.current();
        _tokenIdCounter.increment();
        
        _safeMint(msg.sender, tokenId);
        walletMints[msg.sender]++;
        
        emit NFTMinted(msg.sender, tokenId);
    }
    
    function mintMultiple(uint256 quantity) public payable {
        require(saleIsActive, "Sale is not active");
        require(msg.value >= MINT_PRICE * quantity, "Insufficient payment");
        require(_tokenIdCounter.current() + quantity <= MAX_SUPPLY, "Would exceed max supply");
        require(walletMints[msg.sender] + quantity <= MAX_PER_WALLET, "Would exceed max per wallet");
        
        for (uint256 i = 0; i < quantity; i++) {
            uint256 tokenId = _tokenIdCounter.current();
            _tokenIdCounter.increment();
            _safeMint(msg.sender, tokenId);
            emit NFTMinted(msg.sender, tokenId);
        }
        
        walletMints[msg.sender] += quantity;
    }
    
    function toggleSale() public onlyOwner {
        saleIsActive = !saleIsActive;
        emit SaleStateChanged(saleIsActive);
    }
    
    function setBaseURI(string memory _baseURI) public onlyOwner {
        baseTokenURI = _baseURI;
    }
    
    function withdraw() public onlyOwner {
        uint256 balance = address(this).balance;
        require(balance > 0, "No funds to withdraw");
        
        (bool success, ) = payable(owner()).call{value: balance}("");
        require(success, "Transfer failed");
    }
    
    function _baseURI() internal view override returns (string memory) {
        return baseTokenURI;
    }
    
    function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage) returns (string memory) {
        return super.tokenURI(tokenId);
    }
    
    function supportsInterface(bytes4 interfaceId) public view override(ERC721, ERC721URIStorage) returns (bool) {
        return super.supportsInterface(interfaceId);
    }
    
    function totalSupply() public view returns (uint256) {
        return _tokenIdCounter.current();
    }
}
```

## 🌐 构建前端DApp

### 技术栈

- **React** + **TypeScript**
- **ethers.js** - 与区块链交互
- **Tailwind CSS** - 样式
- **RainbowKit** - 钱包连接

### 安装前端依赖

```bash
# 在项目根目录创建前端文件夹
mkdir frontend
cd frontend
npx create-react-app . --template typescript

# 安装依赖
npm install ethers @rainbow-me/rainbowkit wagmi viem
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 连接钱包组件

```typescript
// frontend/src/App.tsx
import { RainbowKitProvider, getDefaultConfig } from '@rainbow-me/rainbowkit';
import { WagmiProvider } from 'wagmi';
import { mainnet, sepolia } from 'wagmi/chains';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import '@rainbow-me/rainbowkit/styles.css';
import TokenDashboard from './components/TokenDashboard';

const config = getDefaultConfig({
  appName: 'My Web3 App',
  projectId: 'YOUR_WALLET_CONNECT_PROJECT_ID',
  chains: [mainnet, sepolia],
  ssr: true,
});

const queryClient = new QueryClient();

function App() {
  return (
    <WagmiProvider config={config}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider>
          <div className="min-h-screen bg-gray-100">
            <header className="bg-white shadow">
              <div className="max-w-7xl mx-auto py-6 px-4 flex justify-between items-center">
                <h1 className="text-3xl font-bold text-gray-900">My Web3 App</h1>
                <ConnectButton />
              </div>
            </header>
            <main className="max-w-7xl mx-auto py-6 px-4">
              <TokenDashboard />
            </main>
          </div>
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}

export default App;
```

### 代币交互组件

```typescript
// frontend/src/components/TokenDashboard.tsx
import { useAccount, useReadContract, useWriteContract } from 'wagmi';
import { formatUnits, parseUnits } from 'viem';
import { useState } from 'react';

const TOKEN_ABI = [
  "function balanceOf(address owner) view returns (uint256)",
  "function transfer(address to, uint256 amount) returns (bool)",
  "function totalSupply() view returns (uint256)",
  "function name() view returns (string)",
  "function symbol() view returns (string)"
];

const TOKEN_ADDRESS = "YOUR_DEPLOYED_CONTRACT_ADDRESS";

export default function TokenDashboard() {
  const { address, isConnected } = useAccount();
  const [recipient, setRecipient] = useState('');
  const [amount, setAmount] = useState('');
  
  const { data: balance } = useReadContract({
    address: TOKEN_ADDRESS,
    abi: TOKEN_ABI,
    functionName: 'balanceOf',
    args: address ? [address] : undefined,
  });
  
  const { data: tokenName } = useReadContract({
    address: TOKEN_ADDRESS,
    abi: TOKEN_ABI,
    functionName: 'name',
  });
  
  const { writeContract } = useWriteContract();
  
  const handleTransfer = () => {
    if (!recipient || !amount) return;
    
    writeContract({
      address: TOKEN_ADDRESS,
      abi: TOKEN_ABI,
      functionName: 'transfer',
      args: [recipient, parseUnits(amount, 18)],
    });
  };
  
  if (!isConnected) {
    return <div className="text-center py-10">请连接钱包</div>;
  }
  
  return (
    <div className="bg-white shadow rounded-lg p-6">
      <h2 className="text-2xl font-bold mb-4">{tokenName}</h2>
      
      <div className="mb-6">
        <p className="text-gray-600">我的余额</p>
        <p className="text-3xl font-bold">
          {balance ? formatUnits(balance, 18) : '0'} MTK
        </p>
      </div>
      
      <div className="space-y-4">
        <h3 className="text-lg font-semibold">转账</h3>
        <input
          type="text"
          placeholder="接收地址"
          value={recipient}
          onChange={(e) => setRecipient(e.target.value)}
          className="w-full p-2 border rounded"
        />
        <input
          type="number"
          placeholder="数量"
          value={amount}
          onChange={(e) => setAmount(e.target.value)}
          className="w-full p-2 border rounded"
        />
        <button
          onClick={handleTransfer}
          className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
        >
          转账
        </button>
      </div>
    </div>
  );
}
```

## ⚠️ 安全注意事项

### 常见漏洞

1. **重入攻击**
```solidity
// 错误的写法
function withdraw() public {
    uint256 amount = balances[msg.sender];
    (bool success, ) = msg.sender.call{value: amount}(""); // 先转账
    require(success);
    balances[msg.sender] = 0; // 后更新状态
}

// 正确的写法（检查-生效-交互模式）
function withdraw() public {
    uint256 amount = balances[msg.sender];
    require(amount > 0);
    balances[msg.sender] = 0; // 先更新状态
    (bool success, ) = msg.sender.call{value: amount}(""); // 后转账
    require(success);
}
```

2. **整数溢出**
- 使用Solidity 0.8.0+（内置溢出检查）
- 或使用SafeMath库

3. **访问控制**
- 使用OpenZeppelin的Ownable或AccessControl
- 不要自己实现权限控制

4. **随机数**
- 不要在链上生成随机数（可预测）
- 使用Chainlink VRF等预言机

### 安全工具

- **Slither** - 静态分析
- **Mythril** - 符号执行
- **Echidna** - 模糊测试

```bash
# 安装Slither
pip install slither-analyzer

# 运行分析
slither .
```

## 🚀 下一步学习计划

### 进阶主题

- [ ] 去中心化金融（DeFi）协议
- [ ] Layer 2解决方案（Polygon、Arbitrum）
- [ ] 跨链桥
- [ ] DAO治理
- [ ] MEV和套利

### 推荐资源

**文档**：
- [Solidity官方文档](https://docs.soliditylang.org/)
- [OpenZeppelin文档](https://docs.openzeppelin.com/)
- [Ethereum开发文档](https://ethereum.org/en/developers/)

**课程**：
- CryptoZombies（游戏化学习Solidity）
- Ethernaut（安全挑战）
- Speed Run Ethereum（快速上手）

---

## 💭 写在最后

Web3开发是一个充满机遇和挑战的领域。

作为传统Web2开发者，转型Web3需要学习：
- 新的编程范式（智能合约）
- 新的安全模型（代码不可篡改）
- 新的用户体验（钱包、Gas费）

但这正是技术的魅力所在——**不断学习，不断进化**。

> "Web3还在早期，现在进入正是好时机。"

如果你也对Web3开发感兴趣，欢迎交流！⛓️

---

**风险提示**：智能合约代码一旦部署就不可修改，请确保在测试网充分测试后再部署到主网。本文代码仅供学习参考。

---

*写于 2025年6月12日 | Web3学习第4个月*
