# OpenZeppelin Learn

## Project Setup
```
mkdir learn && cd learn
npm init -y
```

### Guide to npx
A third binary was included when installing node: npx. This is used to run executables installed locally in your project.

## Use [Truffle](https://trufflesuite.com/) as development tool
```
npm install truffle
npx truffle init
```

## Truffle first configuration
```
// truffle-config.js
  ...

// Configure your compilers
compilers: {
    solc: {
        version: "0.8.4",    // Fetch exact version from solc-bin (default: truffle's version)
        // docker: true,        // Use "0.5.1" you've installed locally with docker (default: false)
        // settings: {          // See the solidity docs for advice about optimization and evmVersion
        //  optimizer: {
        //    enabled: false,
        //    runs: 200
        //  },
        //  evmVersion: "byzantium"
        // }
    }
},

  ...
```

## Truffle compile
```
npx truffle compile
```

## Install and use local blockchain [Ganache](https://github.com/trufflesuite/ganache)
```
npm install ganache-cli
npx ganache-cli --deterministic //run ganache on localhost:8545
```

## Deploy on ganache network
```
// migrations/2_deploy.js
const Box = artifacts.require('Box');

module.exports = async function (deployer) {
  await deployer.deploy(Box);
};
```


```
// truffle-config.js
module.exports = {
...
  networks: {
...
    development: {
     host: "127.0.0.1",     // Localhost (default: none)
     port: 8545,            // Standard Ethereum port (default: none)
     network_id: "*",       // Any network (default: none)
    },
    ...
```

```
npx truffle migrate --network development
```

## Use truffle console cli
```
//Only with runned truffle on ganache network
npx truffle console --network development
//some example
truffle(development)> await box.store(42)
truffle(development)> await box.retrieve()
truffle(development)> (await box.retrieve()).toString()

```

## Use code
```
npx truffle exec --network development ./scripts/index.js
```

# Write unit test
## Use [Chai](https://www.chaijs.com/) assertions for unit tests
```
npm install chai
```
Write tests inside test/ folder. Tests are best structured by mirroring the contracts directory: for each .sol file there, create a corresponding test file.
Write test calling test/Box.test.js
```
npx truffle test
```

## Performing complex assertions using [OpenZeppelin Test Helpers](https://docs.openzeppelin.com/test-helpers/0.5/)
```
npm install @openzeppelin/test-helpers
```

# Deploy on a public testnet
## Configure
```
npm install @truffle/hdwallet-provider
```

```
// truffle-config.js
const { alchemyApiKey, mnemonic } = require('./secrets.json');
const HDWalletProvider = require('@truffle/hdwallet-provider');

 module.exports = {
   ...
   networks: {
     development: {
      ...
     },
    rinkeby: {
      provider: () => new HDWalletProvider(
        mnemonic, `https://eth-rinkeby.alchemyapi.io/v2/${alchemyApiKey}`,
      ),
      network_id: 4,
      gasPrice: 10e9,
      skipDryRun: true,
    },
   },
   ...
 };
```
### ./secrets.json
```
{
  "mnemonic": "drama film snack motion ...",
  "alchemyApiKey": "JPV2..."
}
```

## Console test
```
npx truffle console --network rinkeby
truffle(rinkeby)> accounts
[ '0xEce6999C6c5BDA71d673090144b6d3bCD21d13d4',
  '0xC1310ade58A75E6d4fCb8238f9559188Ea3808f9',
... ]
## or
accounts = await web3.eth.getAccounts()
acc1 = accounts[1]
balance1 = await web3.eth.getBalance(acc1)
```

## Deploy on testnet
```
npx truffle migrate --network rinkeby
```

```
npx truffle console --network rinkeby
truffle(rinkeby)> box = await Box.deployed()
undefined
truffle(rinkeby)> await box.store(42)
{ tx:
   '0x7d1a556b34fcb26855c6646669fc926738b05589591de6c7bb1f8773546817e7',
...
truffle(rinkeby)> (await box.retrieve()).toString()
'42'
```

# Create [Upgradable smart contract](https://docs.openzeppelin.com/learn/upgrading-smart-contracts#upgrading-a-contract-via-plugins)
To implement upgrades the SM need to be deployed using Upgrade Plugins
```
npm install @openzeppelin/truffle-upgrades
```

```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

The deploy is different:
```
// migrations/3_deploy_upgradeable_box.js
const { deployProxy } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');

module.exports = async function (deployer) {
  await deployProxy(Box, [42], { deployer, initializer: 'store' });
};
```

Then to upgrade contract:
```
// contracts/BoxV2.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BoxV2 {
    // ... code from Box.sol

    // Increments the stored value by 1
    function increment() public {
        _value = _value + 1;
        emit ValueChanged(_value);
    }
}
```


```
// migrations/4_upgrade_box.js
const { upgradeProxy } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');
const BoxV2 = artifacts.require('BoxV2');

module.exports = async function (deployer) {
  const existing = await Box.deployed();
  await upgradeProxy(existing.address, BoxV2, { deployer });
};
```

## Limitations of contract upgrades
- Contracts **cannot** have a constructor, you can use [Initializable](https://docs.openzeppelin.com/learn/upgrading-smart-contracts#contracts::api/proxy.adoc#Initializable)

- The new version of the contract **cannot** change the storage layout of that contract. This means that, if you have already declared a state variable in your contract, you cannot remove it, change its type, or declare another variable before it.

- Follow [Writing Upgradeable Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable) and [Modifying Your Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#modifying-your-contracts)

