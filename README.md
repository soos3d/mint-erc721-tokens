# Manually mint ERC721 tokens

This simple project shows how to use [Remix IDE](https://remix.ethereum.org/), [Filebase](https://filebase.com/), and [IPFS](https://ipfs.tech/) to mint an ERC721 token.

## TL;DR

In this repo, you will find an ERC721 smart contract and instructions to manually mint an NFT on the Polygon Mumbai network and see it on [testnet Rarible](https://testnet.rarible.com/), using a [Chainstack endpoint](https://chainstack.com/).

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

* Compiling the smart contract with `Ctrl + s` (Make sure to select at least the 0.8.0 version of the compiler).
* Deploying the smart contract from the `Deploy $ run transactions` section.
  * Select `Injected Provider - Metamask`.
  * Click deploy and confirm the transaction on MetaMask.
  
![screely-1662588123818](https://user-images.githubusercontent.com/99700157/188991640-5b292892-6012-496d-95a1-365b3a95596e.png)

Congratulations! You just deployed an NFT contract!

