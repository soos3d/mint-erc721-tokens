# Manually mint ERC721 tokens

This simple project shows how to use [Remix IDE](https://remix.ethereum.org/), [Filebase](https://filebase.com/), and [IPFS](https://ipfs.tech/) to mint an ERC721 token.

## TL;DR

In this repo, you will find an ERC721 smart contract and instructions to manually mint an NFT on the Polygon Mumbai network and see it on [testnet Rarible](https://testnet.rarible.com/) or [testnet Opensea](https://testnets.opensea.io/) using a [Chainstack endpoint](https://chainstack.com/).

## ERC721 smart contract

The smart contract in this repo is already good to be deployed out of the box. You can use the [OpenZeppelin contracts wizard](https://wizard.openzeppelin.com/) to create your custom smart contract easily. 

The OpenZeppelin contracts wizard base for this smart contract was:

* ERC721
  * Mintable.
    * Auto Increment Ids.
  * Enumerable.
  * URI Storage.
  
  **Note:** Check the [OpenZeppelin docs](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721) to see more details about the features.
  
  ![screely-1662586876527](https://user-images.githubusercontent.com/99700157/188987755-bccec1a5-448d-4f4e-a3e5-e072336d23a6.png)

### Modifications to the base ERC721 contract

The OpenZeppelin tool creates a primary smart contract based on the features that we have selected. But we want to make a few small changes to make it more usable. 

We'll make two changes:

1. Allow any address to use the `safeMint()` function so that not only the owner of the contract can mint. 
    * Note that this is for learning purposes, so in this case any address can mint for free.
1. Put a limited number of mints avalable. 

#### safeMint() function

This is the function that mints the NFT to the address selected. It takes two parameters:

* The address to mint the NFT to.
* The URI containing the Metadata of the NFT. 

We first need to modify the top of the contract to:

* Remove the `Ownable.sol` contract to allow anyone to mint.
* Add a variable to limit the number of NFTs that can be minted

```js
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts@4.7.3/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts@4.7.3/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts@4.7.3/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts@4.7.3/utils/Counters.sol";

contract LearnWeb3 is ERC721, ERC721Enumerable, ERC721URIStorage {
    using Counters for Counters.Counter;    // Initialize the counter library

    // Initialize a counter variable, private means that only this smart contract can access it. 
    Counters.Counter private _tokenIdCounter;

    uint256 MAX_SUPPLY = 100;   // Limits the number of mints to 100.
```

Then we modify the `safeMint()` function like this:

```js
function safeMint(address to, string memory uri) public {
        uint256 tokenId = _tokenIdCounter.current();
        require(tokenId <= MAX_SUPPLY, "LearnWeb3 is sold out!");
        _tokenIdCounter.increment();
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }
```

Now our contract is ready to be deployed.

## Deploy the smart contract on Polygon Mumbai

To deploy on Polygon Mumbai, you will need a Mumbai endpoint. For this, I use [Chainstack](https://chainstack.com/).

Set up the endpoint URL 

1. [Sign up with Chainstack](https://console.chainstack.com/user/account/create).  
1. [Deploy a node](https://docs.chainstack.com/platform/join-a-public-network).  
1. [View node access and credentials](https://docs.chainstack.com/platform/view-node-access-and-credentials). 

We can deploy directly from Remix by:

* Compiling the smart contract with `Ctrl + S` (Make sure to select at least the 0.8.0 version of the compiler).
* Deploying the smart contract from the `Deploy $ run transactions` section.
  * Select `Injected Provider - Metamask`.
  * Click deploy and confirm the transaction on MetaMask.
  
![screely-1662588123818](https://user-images.githubusercontent.com/99700157/188991640-5b292892-6012-496d-95a1-365b3a95596e.png)

Congratulations! You just deployed an NFT contract!

## Use Filebase to host the NFT image on IPFS

In this tutorial we use [Filebase](https://filebase.com/) to host the image on IPFS. 

Once you have an account and you are on dashboard:

* Select `Buckets` and create a new bucket.

![screely-1662594653071](https://user-images.githubusercontent.com/99700157/189002874-c2d82097-0880-41ae-a585-898b0959fb7a.png)

* Name the bucket (low caps only) and select IPFS as the `Storage Network`.

![screely-1662595023198](https://user-images.githubusercontent.com/99700157/189004708-fc37ce6d-2907-49ad-8517-cc988a4997d3.png)

Now you can click `Upload` and upload the image file.

Once it is uploaded, you can click on it and see the details. We are interested in the `IPFS Gateway URL`, which we need to use in the next section.

![screely-1662595401003](https://user-images.githubusercontent.com/99700157/189005728-fa0d6107-7aee-441c-bbc6-cb694feb43af.png)

## Set up the Metadata

The metadata describes the details of our NFT; it can be used to provide attributes, names, descriptions, etc.

We use a `JSON` file to set up the Metadata, which will provide the URI.
 
In this tutorial, we mint and set it up manually so you can see how it works, but there are ways to set it up programmatically, which is used to handle many NFTs. 

Check the [Rarible docs](https://docs.rarible.org/getting-started/ipfs-example/) to see the standard and examples for the metadata.

Copy the `IPFS Gateway URL` from Filebase and paste into the `"image"` parameter of the metadata `JSON` file.

The `JSON` file looks like this and we can name it `metadata.json`:

```shell
{
    "name": "LearnWeb3",
    "description": "NFTs to practice and learn web3 development.",
    "image": "IPFS Gateway URL from filesafe",
    "external_url": "Website URL if you have it",
    "attributes": [
       {
          "trait_type": "Learning",
          "value": "Web3 development"
       },
       {
        "trait_type": "Environment",
        "value": "The Matrix"
     },
     {
        "trait_type": "Charachter",
        "value": "Ape"
     }
    ]
 }
```

As you can see, we can set up the details and attributes, which will be picked up by the different NFT marketplaces and displayed to the user.

Now upload the `metatada.json` file in the same bucket we just uploaded the image in Filebase.

In this case we want to take the `IPFS CID`

![screely-1662595401003](https://user-images.githubusercontent.com/99700157/189006213-7f5435e5-c63b-4e00-9165-c02eb2943609.png)

## Mint an NFT

Now we have everything set up to mint our first NFT! 

As a recap:

* We deployed an ERC721 on the Mumbai testnet.
* We uploaded the image into `IPFS` using Filebase.
* We created a `metadata.json` for the attributes and the URI.

Minting the NFT now is simple, and we can do it from Remix.

From the deployed contract section expand the `safemint` function. You will have two parameters:

* `to`: Instert your wallet address (or the address the NFT will be minted to).
* `uri`: type `ipfs://` + `IPFS CID` from the `metadata.json` file uploaed on Filebase.
  * like this example: `ipfs://QmcWx9y6yfLY1v9NZxxxxxxxx`

![screely-1662596101916](https://user-images.githubusercontent.com/99700157/189006959-9fd01a52-ab9c-4077-8adc-1faf0a814499.png)

Then just press `transact`! Once you confirm the transaction you can verity it on the [Mumbai explorer like my example](https://mumbai.polygonscan.com/tx/0x8e34a3678b1ae94f443f9ae58f0fe5d72cab60c55fa79fa4f28bd8690211dc68).

## See your NFT on a marketplace

At this point, you minted your NFT, and you can see it on a marketplace! 

> Note that it might take time for the marketplace to fetch the metadata. 

See how Opensea displays the metadata.

![screely-1662596829141](https://user-images.githubusercontent.com/99700157/189007550-c4c10168-a398-4d21-8814-6a9c2a3e8f88.png)

Congratulations on completing this tutorial and learning how to manually mint NFTs!

See the NFT I minted on [testnet Opensea](https://testnets.opensea.io/assets/mumbai/0x1dd1d3a107908088b82c9095987786b3cf154231/0/) and [testnet Rarible](https://testnet.rarible.com/token/polygon/0x1dd1d3a107908088b82c9095987786b3cf154231:0). 
