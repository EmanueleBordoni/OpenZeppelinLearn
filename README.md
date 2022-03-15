# OpenZeppelin Learn

## Project Setup
```
mkdir learn && cd learn
npm init -y
```

### Guide to npx
A third binary was included when installing node: npx. This is used to run executables installed locally in your project.

## Use Truffle as development tool
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

## Install and use local blockchain (https://github.com/trufflesuite/ganache)[Ganache]
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
