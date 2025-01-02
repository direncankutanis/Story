# Story Protocol Tutorial

## Introduction
This tutorial explains how to register Intellectual Property (IP) on the Story Protocol using the TypeScript SDK and Smart Contracts. Follow these steps to learn how to mint NFTs, register IPs, and manage royalties using the Story Protocol.

## Prerequisites
1. Add your Story Network Testnet wallet's private key to a `.env` file:

```env
WALLET_PRIVATE_KEY=<YOUR_WALLET_PRIVATE_KEY>
```

2. Generate a Pinata API Key and add the JWT to `.env`:

```env
PINATA_JWT=<YOUR_PINATA_JWT>
```

3. Add your preferred RPC URL to `.env`:

```env
RPC_PROVIDER_URL=https://rpc.odyssey.storyrpc.io
```

4. Install dependencies:

```bash
npm install @story-protocol/core-sdk @pinata/sdk viem
```

---

## Step 1: Set Up Story Config

### File: `main.ts`

```typescript
import { StoryClient, StoryConfig } from '@story-protocol/core-sdk';
import { http } from 'viem';
import { privateKeyToAccount, Address, Account } from 'viem/accounts';

const privateKey: Address = 0x${process.env.WALLET_PRIVATE_KEY};
const account: Account = privateKeyToAccount(privateKey);

const config: StoryConfig = {
  account: account,
  transport: http(process.env.RPC_PROVIDER_URL),
  chainId: 'odyssey',
};

const client = StoryClient.newClient(config);
```

---

## Step 2: Prepare Metadata

### IP Metadata

```typescript
import { IpMetadata } from '@story-protocol/core-sdk';

const ipMetadata: IpMetadata = client.ipAsset.generateIpMetadata({
  title: 'My IP Asset',
  description: 'This is a test IP asset',
  watermarkImg: 'https://picsum.photos/200',
  attributes: [
    {
      key: 'Rarity',
      value: 'Legendary',
    },
  ],
});
```

### NFT Metadata

```typescript
const nftMetadata = {
  name: 'Test NFT',
  description: 'This is a test NFT',
  image: 'https://picsum.photos/200',
};
```

---

## Step 3: Upload Metadata to IPFS

### File: `utils/uploadToIpfs.ts`

```typescript
const pinataSDK = require('@pinata/sdk');

export async function uploadJSONToIPFS(jsonMetadata): Promise<string> {
  const pinata = new pinataSDK({ pinataJWTKey: process.env.PINATA_JWT });
  const { IpfsHash } = await pinata.pinJSONToIPFS(jsonMetadata);
  return IpfsHash;
}
```

### Upload Metadata

```typescript
import { uploadJSONToIPFS } from './utils/uploadToIpfs';
import { createHash } from 'crypto';

const ipIpfsHash = await uploadJSONToIPFS(ipMetadata);
const ipHash = createHash('sha256').update(JSON.stringify(ipMetadata)).digest('hex');
const nftIpfsHash = await uploadJSONToIPFS(nftMetadata);
const nftHash = createHash('sha256').update(JSON.stringify(nftMetadata)).digest('hex');
```

---

## Step 4: Register the NFT as an IP Asset

### Option 1: Mint NFT and Register Separately

#### Mint NFT

```typescript
import { mintNFT } from './utils/mintNFT';

const tokenId = await mintNFT(account.address, `https://ipfs.io/ipfs/${nftIpfsHash}`);
```

#### Register IP Asset

```typescript
import { RegisterIpAndAttachPilTermsResponse, AddressZero } from '@story-protocol/core-sdk';

const response: RegisterIpAndAttachPilTermsResponse = await client.ipAsset.registerIpAndAttachPilTerms({
  nftContract: process.env.NFT_CONTRACT_ADDRESS as Address,
  tokenId: tokenId!,
  terms: [],
  ipMetadata: {
    ipMetadataURI: `https://ipfs.io/ipfs/${ipIpfsHash}`,
    ipMetadataHash: `0x${ipHash}`,
    nftMetadataURI: `https://ipfs.io/ipfs/${nftIpfsHash}`,
    nftMetadataHash: `0x${nftHash}`,
  },
  txOptions: { waitForTransaction: true },
});

console.log(`Root IPA created at transaction hash ${response.txHash}, IPA ID: ${response.ipId}`);
```

### Option 2: Mint and Register in One Transaction

#### Deploy SPG NFT Collection

```typescript
const newCollection = await client.nftClient.createNFTCollection({
  name: 'Test NFT',
  symbol: 'TEST',
  isPublicMinting: true,
  mintOpen: true,
  mintFeeRecipient: AddressZero,
  contractURI: '',
  txOptions: { waitForTransaction: true },
});

console.log(`SPG NFT Collection Address: ${newCollection.spgNftContract}`);
```

#### Mint and Register IP Asset

```typescript
import { CreateIpAssetWithPilTermsResponse } from '@story-protocol/core-sdk';

const response: CreateIpAssetWithPilTermsResponse = await client.ipAsset.mintAndRegisterIpAssetWithPilTerms({
  spgNftContract: process.env.SPG_NFT_CONTRACT_ADDRESS as Address,
  terms: [],
  ipMetadata: {
    ipMetadataURI: `https://ipfs.io/ipfs/${ipIpfsHash}`,
    ipMetadataHash: `0x${ipHash}`,
    nftMetadataURI: `https://ipfs.io/ipfs/${nftIpfsHash}`,
    nftMetadataHash: `0x${nftHash}`,
  },
  txOptions: { waitForTransaction: true },
});

console.log(`Root IPA created at transaction hash ${response.txHash}, IPA ID: ${response.ipId}`);
```

---

## Step 5: Done!
Congratulations! You have successfully registered an IP Asset on Story Protocol.

For further details, refer to the [Story Protocol Documentation](https://docs.story.foundation).
