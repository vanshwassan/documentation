---
title: API3
description: API3 is a collaborative project to deliver traditional API services to smart contract platforms in a decentralized and trust-minimized way. It is governed by a decentralized autonomous organization (DAO), namely the API3 DAO.
keywords: [api3, airnode, oracle, data feeds, dapis, qrng]
---

# API3

[API3](https://api3.org/) is a collaborative project to deliver traditional API services to smart contract platforms in a decentralized and trust-minimized way. It is governed by a decentralized autonomous organization (DAO), namely the [API3 DAO](https://api3.org/dao).

:::info The API3 DAO
Read more about how The API3 DAO works. [Click here](https://docs.api3.org/dao-members/)
:::


## Airnode

Developers can use [Airnode](https://docs.api3.org/airnode/) to request off-chain data inside their Smart Contracts on the Gnosis Chain. An Airnode is a first-party oracle that pushes off-chain API data to your on-chain contract. Airnode lets API providers easily run their own first-party oracle nodes. That way, they can provide data to any on-chain dApp that's interested in their services, all without an intermediary.

An on-chain smart contract makes a request in the [RRP (Request Response Protocol)](https://docs.api3.org/airnode/v0.10/concepts/) contract (`AirnodeRrpV0.sol`) that adds the request to the event logs. The Airnode then accesses the event logs, fetches the API data and performs a callback to the requester with the requested data.

![API3 Remix deploy](/img/tools/api3/airnode1.png)

## Requesting off-chain data by calling an Airnode
Requesting off-chain data essentially involves triggering an Airnode and getting its response through your smart contract. The smart contract in this case would be the requester contract which will make a request to the desired off-chain Airnode and then capture its response.

The requester calling an Airnode primarily focuses on two tasks:

- Make the request
- Accept and decode the response

![API3 Remix deploy](/img/tools/api3/airnode2.png)


Here is an example of a basic requester contract to request data from an Airnode:

```solidity
pragma solidity 0.8.9;

import "@api3/airnode-protocol/contracts/rrp/requesters/RrpRequesterV0.sol";

// A Requester that will return the requested data by calling the specified airnode.
// Make sure you specify the right _rrpAddress for your chain.

contract Requester is RrpRequesterV0 {
    mapping(bytes32 => bool) public incomingFulfillments;
    mapping(bytes32 => int256) public fulfilledData;

    constructor(address _rrpAddress) RrpRequesterV0(_rrpAddress) {}

    /**
     * The main makeRequest function that will trigger the Airnode request
     * airnode: Airnode address
     * endpointId: The endpoint ID for the specific endpoint
     * sponsor: The requester contract itself (in this case)
     * sponsorWallet: The wallet that will make the actual request (needs to be funded)
     * parameters: encoded API parameters
     */
    function makeRequest(
        address airnode,
        bytes32 endpointId,
        address sponsor,
        address sponsorWallet,
        bytes calldata parameters
        
    ) external {
        bytes32 requestId = airnodeRrp.makeFullRequest(
            airnode,
            endpointId,
            sponsor,
            sponsorWallet,
            address(this),
            this.fulfill.selector,
            parameters
        );
        incomingFulfillments[requestId] = true;
    }

    // The callback function with the requested data
    function fulfill(bytes32 requestId, bytes calldata data)
        external
        onlyAirnodeRrp
    {
        require(incomingFulfillments[requestId], "No such request made");
        delete incomingFulfillments[requestId];
        int256 decodedData = abi.decode(data, (int256));
        fulfilledData[requestId] = decodedData;
    }
}
```

The `_rrpAddress` is the main `airnodeRrpAddress`. The RRP Contracts have already been deployed on-chain. You can check the address for Gnosis Chain [here](https://docs.api3.org/airnode/v0.9/reference/airnode-addresses.html). You can also try [deploying it on Remix](https://remix.ethereum.org/#url=https://github.com/vanshwassan/RemixContracts/blob/master/contracts/Requester.sol&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.9+commit.e5eed63a.js)

|         Contract         |                     Addresses                    |
|:------------------------:|:------------------------------------------------:|
| AirnodeRrpV0 |   `0xa0AD79D995DdeeB18a14eAef56A549A04e3Aa1Bd`    |


### Request parameters

The `makeRequest()` function expects the following parameters to make a valid request.

- [**`airnode`**](https://docs.api3.org/airnode/v0.10/concepts/airnode.html): Specifies the Airnode Address.
- [**`endpointId`**](https://docs.api3.org/airnode/v0.10/concepts/endpoint.html): Specifies which endpoint to be used.
- [**`sponsor`**](https://docs.api3.org/airnode/v0.10/concepts/sponsor.html) and [**`sponsorWallet`**](https://docs.api3.org/airnode/v0.10/concepts/sponsor.html#sponsorwallet): Specifies which wallet will be used to fulfill the request.
- [**`parameters`**](https://docs.api3.org/ois/v1.4/reserved-parameters.html): Specifies the API and Reserved Parameters (see [Airnode ABI specifications](awd) for how these are encoded). Parameters can be encoded off-chain using [`@airnode-abi`](https://docs.api3.org/airnode/v0.10/reference/specifications/airnode-abi-specifications.html) library.

### Response parameters

The callback to the Requester contains two parameters:

- [**`requestId`**](https://docs.api3.org/airnode/v0.10/concepts/request.html#requestid): First acquired when making the request and passed here as a reference to identify the request for which the response is intended.
- **`data`**: In case of a successful response, this is the requested data which has been encoded and contains a timestamp in addition to other response data. Decode it using the `decode()` function from the `abi` object.

:::info Note !
Sponsors should not fund a `sponsorWallet` with more then they can trust the Airnode with, as the Airnode controls the private key to the `sponsorWallet`. The deployer of such Airnode undertakes no custody obligations, and the risk of loss or misuse of any excess funds sent to the `sponsorWallet` remains with the sponsor.
:::

## Using dAPIs

[dAPIs](https://docs.api3.org/dapis/) are continuously updated streams of off-chain data, such as the latest cryptocurrency, stock and commodity prices. They can power various decentralized applications such as DeFi lending, synthetic assets, stablecoins, derivatives, NFTs and more.

Due to being composed of first-party data feeds, dAPIs offer security, transparency, cost-efficiency and scalability in a turn-key package.

![API3 Remix deploy](/img/tools/api3/SS4.png)

### Subscribing to self-funded dAPIs

With Self-Funded dAPIs, you can fund the dAPI with your own funds. The amount of gas you supply will determine how long your dAPI will be available for use. If you run out of gas, you can fund the dAPI again to keep it available for use.

#### **Exploring and selecting your dAPI**

The [API3 Market](https://market.api3.org/dapis) provides a list of all the dAPIs available across multiple chains including testnets. You can filter the list by chains and data providers. You can also search for a specific dAPI by name. Once selected you will land on the details page where you can find more information about the dAPI.

#### **Funding a sponsor wallet**

Once you have selected your dAPI, you can fund it by using the Market to send funds to the `sponsorWallet`, make sure your:

- Wallet is connected to the Market and is the same network as the dAPI you are funding.
- Balance of the wallet should be greater than the amount you are sending to the `sponsorWallet`.

![API3 Remix deploy](/img/tools/api3/SS1.png)

To fund the dAPI, you need to click on the **Fund sponsor wallet/Fund Gas** button. Depending upon if a proxy contract is already deployed, you will see a different UI.

![API3 Remix deploy](/img/tools/api3/SS2.png)

Use the gas estimator to select how much gas is needed by the dAPI. Click on **Send XDAI** to send the entered amount to the sponsor wallet of the respective dAPI.

![API3 Remix deploy](/img/tools/api3/SS3.png)

Once the transaction is broadcasted & confirmed on the blockchain a transaction confirmation screen will appear.

![API3 Remix deploy](/img/tools/api3/SS5.png)

#### **Deploying a proxy contract to access the dAPI**

Smart contracts can interact and read values from contracts that are already deployed on the blockchain. By deploying a proxy contract via the API3 market, a dAPP can interact and read values from a dAPI like ETH/USD.

:::info Note:
 If a proxy is already deployed for a self-funded dAPI, the dAPP can read the dAPI without having to deploy a proxy contract by using the address of the already deployed proxy contract which will be visible on the API3 market.
:::

If you are deploying a proxy contract during the funding process, clicking on the **Deploy proxy** button will initiate a transaction to your Metamask that will deploy a proxy contract.

![API3 Remix deploy](/img/tools/api3/SS6.png)

Once the transaction is broadcasted & confirmed on the blockchain, the proxy contract address will be shown on the UI.

![API3 Remix deploy](/img/tools/api3/SS7.png)


### Reading from a self-funded dAPI

Here's an example of a basic contract that reads from a self-funded dAPI.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.9;

import "@api3/airnode-protocol-v1/contracts/dapis/proxies/interfaces/IDapiProxy.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Reader is Ownable {
    address dapiProxy;

    function setDapiProxyAddress(address _proxyAddress) public onlyOwner {
        dapiProxy = _proxyAddress;
    }

    function readDapi() public view returns (int224 value, uint32 timestamp){
        return IDapiProxy(dapiProxy).read();
    }
}
```

- `setDapiProxyAddress()` is used to set the address of the dAPI Proxy Contract.

- `readDapi()` is a view function that returns the latest price of the set dAPI.

You can read more about dAPIs [here](https://docs.api3.org/dapis/). 

### [Try deploying it on Remix!](https://remix.ethereum.org/#url=https://gist.githubusercontent.com/vanshwassan/1ec4230956a78c73a00768180cba3649/raw/caff497e5b4b61d89d920b49da70779a0a24ac58/DapiReader.sol)

## API3 QRNG

[API3 QRNG](https://docs.api3.org/qrng/) is a public utility we provide with the courtesy of Australian National University (ANU). It is powered by an Airnode hosted by ANU Quantum Random Numbers, meaning that it is a first-party service. It is served as a public good and is free of charge (apart from the gas costs), and it provides ‘true’ quantum randomness via an easy-to-use solution when requiring RNG on-chain.

To request randomness on-chain, the requester submits a request for a random number to AirnodeRrpV0. The ANU Airnode gathers the request from the AirnodeRrpV0 protocol contract, retrieves the random number off-chain, and sends it back to AirnodeRrpV0. Once received, it performs a callback to the requester with the random number.

Here is an example of a basic `QrngRequester` that requests a random number:

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
import "@api3/airnode-protocol/contracts/rrp/requesters/RrpRequesterV0.sol";

contract RemixQrngExample is RrpRequesterV0 {
    event RequestedUint256(bytes32 indexed requestId);
    event ReceivedUint256(bytes32 indexed requestId, uint256 response);

    address public airnode;
    bytes32 public endpointIdUint256;
    address public sponsorWallet;
    mapping(bytes32 => bool) public waitingFulfillment;

    // These are for Remix demonstration purposes, their use is not practical.
    struct LatestRequest { 
      bytes32 requestId;
      uint256 randomNumber;
    }
    LatestRequest public latestRequest;

    constructor(address _airnodeRrp) RrpRequesterV0(_airnodeRrp) {}

    // Normally, this function should be protected, as in:
    // require(msg.sender == owner, "Sender not owner");
    function setRequestParameters(
        address _airnode,
        bytes32 _endpointIdUint256,
        address _sponsorWallet
    ) external {
        airnode = _airnode;
        endpointIdUint256 = _endpointIdUint256;
        sponsorWallet = _sponsorWallet;
    }

    function makeRequestUint256() external {
        bytes32 requestId = airnodeRrp.makeFullRequest(
            airnode,
            endpointIdUint256,
            address(this),
            sponsorWallet,
            address(this),
            this.fulfillUint256.selector,
            ""
        );
        waitingFulfillment[requestId] = true;
        latestRequest.requestId = requestId;
        latestRequest.randomNumber = 0;
        emit RequestedUint256(requestId);
    }

    function fulfillUint256(bytes32 requestId, bytes calldata data)
        external
        onlyAirnodeRrp
    {
        require(
            waitingFulfillment[requestId],
            "Request ID not known"
        );
        waitingFulfillment[requestId] = false;
        uint256 qrngUint256 = abi.decode(data, (uint256));
        // Do what you want with `qrngUint256` here...
        latestRequest.randomNumber = qrngUint256;
        emit ReceivedUint256(requestId, qrngUint256);
    }
}
```

- The `setRequestParameters()` takes in `airnode` (The ANU/Quintessence/byog Airnode address) , `endpointIdUint256`, `sponsorWallet` and sets these parameters. You can get Airnode address and the endpoint ID [here](https://docs.api3.org/qrng/reference/providers.html).

- The `makeRequestUint256()` function calls the `airnodeRrp.makeFullRequest()` function of the `AirnodeRrpV0.sol` protocol contract which adds the request to its storage and returns a `requestId`.

- The targeted off-chain Airnode gathers the request and performs a callback to the requester with the random number.

### [Try deploying it on Remix!](https://remix.ethereum.org/#url=https://github.com/vanshwassan/RemixContracts/blob/master/contracts/QrngRequester.sol&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.9+commit.e5eed63a.js)

You can try QRNG on the Gnosis Chain for free. Check out the all the QRNG Providers for Gnosis Chain [here](https://docs.api3.org/qrng/reference/providers.html).

[Click here to read more about API3 QRNG](https://docs.api3.org/qrng)