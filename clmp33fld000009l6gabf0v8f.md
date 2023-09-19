---
title: "Ethers.js to Viem: A Hands-On Open Source Migration"
seoTitle: "Ether.js to Viem Migration: Open Source Project Guide"
seoDescription: "Learn how to migrate from Ether.js to Viem with this hands-on guide using an open-source project. Explore chain setup, provider configurations, contract int"
datePublished: Mon Sep 18 2023 16:11:50 GMT+0000 (Coordinated Universal Time)
cuid: clmp33fld000009l6gabf0v8f
slug: ethersjs-to-viem-a-hands-on-open-source-migration
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1695053488774/d2b3a9c7-c499-4b8c-8d4d-5972cfba00dc.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1695053475609/1ceaefc8-4f06-4305-8e7f-5239f1000714.png
tags: javascript, ethereum, solidity, web3, ethereum-smart-contracts

---

### Looking to get started with Viem......

Here is an open-source contribution that I have made to [TalentLayer](www.talentlayer.org) organization's [TalentLayer-Starter-Kit](https://github.com/TalentLayer-Labs/starter-kit) repository making a complete removal of ethers.js dependency and using viem instead through this [Pull request](https://github.com/TalentLayer-Labs/starter-kit/pull/20).

I'm excited to share the changes I've made to ease your migration from ethers.js to Viem. I'll do my best to cover all the modifications and provide relevant documentation links.

# Introduction

[Viem](https://viem.sh/) is a typeScript Interface for Ethereum that provides low-level stateless primitives for interacting with Ethereum. An **alternative to ethers.js and web3.js** with a focus on reliability, efficiency, and excellent developer experience.

### Version Details

```plaintext
"viem": "^1.10.9",
"wagmi": "^1.4.1",
```

# Migration

## Chain Setup

Using [defineChain](https://viem.sh/docs/clients/chains.html#chain-configuration) we could set up custom chain configurations.

```typescript
import { defineChain } from 'viem';
export const polygonMumbai = defineChain({
  id: 80_001,
  name: 'Polygon Mumbai',
  network: 'maticmum',
  nativeCurrency: { name: 'MATIC', symbol: 'MATIC', decimals: 18 },
  rpcUrls: {
    alchemy: {
      http: ['https://polygon-mumbai.g.alchemy.com/v2'],
      webSocket: ['wss://polygon-mumbai.g.alchemy.com/v2'],
    },
    infura: {
      http: ['https://polygon-mumbai.infura.io/v3'],
      webSocket: ['wss://polygon-mumbai.infura.io/ws/v3'],
    },
    default: {
      http: ['https://rpc-mumbai.maticvigil.com'],
    },
    public: {
      http: ['https://rpc-mumbai.maticvigil.com'],
    },
  },
  blockExplorers: {
    etherscan: {
      name: 'PolygonScan',
      url: 'https://mumbai.polygonscan.com',
    },
    default: {
      name: 'PolygonScan',
      url: 'https://mumbai.polygonscan.com',
    },
  },
  contracts: {
    multicall3: {
      address: '0xca11bde05977b3631167028862be2a173976ca11',
      blockCreated: 25770160,
    },
  },
  testnet: true,
});
```

To use existing chain configurations: [Chains](https://viem.sh/docs/clients/chains.html#chains)

## Provider -&gt; PublicClient

Provider in ethers.js is used as [PublicCient](https://viem.sh/docs/clients/public.html#public-client) in viem

1. Initialize a Client with your desired [**Chain**](https://viem.sh/docs/clients/chains.html) (e.g. `mainnet`) and [**Transport**](https://viem.sh/docs/clients/intro.html) (e.g. `HTTP`).
    
    ```typescript
    import { createPublicClient, http } from 'viem';
    import { /*chainName*/ } from 'viem/chains';
    
    const client = createPublicClient({ 
      chain: /*chainName*/,
      transport: http()
    });
    ```
    
2. Use Wagmi's hook for accessing viem [Public Client](https://viem.sh/docs/clients/public.html).
    
    ```typescript
    import { usePublicClient } from 'wagmi';
    const publicClient = usePublicClient({ /*chainId*/ });
    ```
    

## Signer -&gt; WalletClient

Signer in ethers.js is used as [WalletCient](https://viem.sh/docs/clients/wallet.html#wallet-client) in viem

1. Initialize a Client with your desired [**Chain**](https://viem.sh/docs/clients/chains.html) (e.g. `mainnet`) and [**Transport**](https://viem.sh/docs/clients/intro.html) (e.g. `custom`).
    
    ```typescript
    import { createWalletClient, custom } from 'viem';
    import { /*chainName*/ } from 'viem/chains';
    
    const client = createWalletClient({
      chain: /*chainName*/,
      transport: custom(window.ethereum)
    });
    ```
    
2. Use Wagmi's hook for accessing viem [Wallet Client](https://viem.sh/docs/clients/wallet.html#wallet-client).
    
    ```typescript
    import { useWalletClient } from 'wagmi';
    const { data: walletClient } = useWalletClient({ /*chainName*/ });
    ```
    

## Contract

### Read Contract

Here is how I implemented the function to read from the Contract.

```typescript
// ETHER.JS
const ERC20Token = new Contract(rateToken, ERC20.abi, signer);
const balance = await ERC20Token.balanceOf(signer.getAddress());

// VIEM
const balance: any = await publicClient.readContract({
        address: rateToken,
        abi: ERC20.abi,
        functionName: 'balanceOf',
        args: [walletClient.getAddresses()]
});
```

Docs: [readContract](https://viem.sh/docs/contract/readContract.html#readcontract)

### Write Contract

Here is how I implemented the function to write from the Contract.

```typescript
 //ETHERS.JS
const contract = new ethers.Contract(
            config.contracts.talentLayerId,
            TalentLayerID.abi,
            signer,
          );
const tx = await contract.updateProfileData(user.id, cid);

//VIEM
const tx = await walletClient.writeContract({
            address: config.contracts.talentLayerId,
            abi: TalentLayerID.abi,
            functionName: 'updateProfileData',
            args: [user.id, cid],
            account: address,
          });
```

Docs: [writeContract](https://viem.sh/docs/contract/writeContract.html)

### Simulate Contract

The `simulateContract` function **simulates**/**validates** a contract interaction. This is useful for retrieving **return data** and **revert reasons** of contract write functions. It is almost identical to [`readContract`](https://viem.sh/docs/contract/readContract.html), but also supports contract write functions.

The code pairs simulate contract and writeContract is better than using writeContract directly as it provides the option to validate the contract before writing.

```typescript
const {request} = await publicClient.simulateContract({
          address: config.contracts.talentLayerEscrow,
          abi: TalentLayerEscrow.abi,
          functionName: 'approve',
          args: [config.contracts.talentLayerEscrow, value]
        });
const tx1 = await walletClient.writeContract(request);
```

Docs: [Simulate Contract](https://viem.sh/docs/contract/simulateContract.html#simulatecontract)

## Utils

### formatUnits

```typescript
//ETHERS.JS
const formattedValue = ethers.utils.formatUnits(value, token.decimals);

//VIEM
import { formatUnits } from 'viem';
const formattedValue = formatUnits(BigInt(value), token.decimals);
```

Docs: [formatUnits](https://viem.sh/docs/utilities/formatUnits.html)

### formatEther

```typescript
//ETHERS.JS
const fee = ethers.utils.formatEther(platform?.proposalPostingFee);

//VIEM
import { formatEthers } from 'viem';
const fee = formatEther(BigInt(platform?.servicePostingFee));
```

Docs: [formatEther](https://viem.sh/docs/utilities/formatEther.html)

### parseUnits

```typescript
//ETHERS.JS
const val = ethers.utils.parseUnits(value.toString(), 'ether');

//VIEM
import { parseUnits } from 'viem';
const fee = parseUnits(value.toString(),decimals);
```

Docs: [parseUnits](https://viem.sh/docs/utilities/parseUnits.html)

### parseEther

```typescript
//ETHERS.JS
const fee = ethers.utils.parseUnits(value.toString(), 'ether').toBigInt()

//VIEM
import { parseEther } from 'viem';
const fee = parseEther(value.toString());
```

Docs: [parseEther](https://viem.sh/docs/utilities/parseEther.html)

## **Let's Connect** 👋

* [Twitter](http://www.twitter.com/gathin_twt)
    
* [**Github**](https://github.com/Gathin23)
    
* [**LinkedIn**](https://linkedin.com/in/gathint)
    

## Final thoughts

I hope this article has been helpful to you. Please feel free to reach out if you have any questions. Your thoughts, suggestions, and corrections are more than welcome.

Special credits to [Kirsten](https://twitter.com/kirstenrpomales) & [Romain](https://twitter.com/Romain__TL) from [TalentLayer](www.talentlayer.org) Team for their constant guidance and support of my work. [TalentLayer](www.talentlayer.org) is an open protocol and developer toolkit for building better hiring platforms.