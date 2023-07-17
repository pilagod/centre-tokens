# centre-tokens

Fiat tokens on the [CENTRE](https://centre.io) network.

## Setup

Requirements:

- Node >= v12
- Yarn

```
$ git clone git@github.com:centrehq/centre-tokens.git
$ cd centre-tokens
$ npm i -g yarn       # Install yarn if you don't already have it
$ yarn install        # Install dependencies
$ yarn setup          # Setup Git hooks
```

## TypeScript type definition files for the contracts

To generate type definitions:

```
$ yarn compile && yarn typechain
```

## Linting and Formatting

To check code for problems:

```
$ yarn typecheck      # Type-check TypeScript code
$ yarn lint           # Check JavaScript and TypeScript code
$ yarn lint --fix     # Fix problems where possible
$ yarn solhint        # Check Solidity code
$ yarn slither        # Run Slither
```

To auto-format code:

```
$ yarn fmt
```

## Testing

First, make sure Ganache is running.

```
$ yarn ganache
```

Run all tests:

```
$ yarn test
```

To run tests in a specific file, run:

```
$ yarn test [path/to/file]
```

To run tests and generate test coverage, run:

```
$ yarn coverage
```

## Deployment

Create a copy of the file `config.js.example`, and name it `config.js`. Enter
the BIP39 mnemonic phrase, the INFURA API key to use for deployment, and the
addresses of proxy admin, owner, master minter, blacklister, and pauser in
`config.js`. This file must not be checked into the repository. To prevent
accidental check-ins, `config.js` is in `.gitignore`.

Also, please check the gas price of the latest transaction on your deploying chain, 
and update gas price in `truffle-config.js` of that network.

* To do v1 deployment, run `yarn deploy:v1 --network NETWORK`, where NETWORK is either `mainnet` or
`goerli`.

* To do full deployment, run `yarn migrate --network NETWORK`, where NETWORK is either `mainnet` or
`goerli`.

This script will first dry run the deployment in local forked network. After the dry run, it will prompt a dialog for you to confirm the deployment details. If everything is as expected, just hit y to proceed the real deployment.

## Contracts

The implementation uses 2 separate contracts - a proxy contract
(`FiatTokenProxy.sol`) and an implementation contract (`FiatToken.sol`). This
allows upgrading the contract, as a new implementation contact can be deployed
and the Proxy updated to point to it.

### FiatToken

The FiatToken offers a number of capabilities, which briefly are described
below. There are more [detailed design docs](./doc/tokendesign.md) in the `doc`
folder.

### ERC20 compatible

The FiatToken implements the ERC20 interface.

### Pausable

The entire contract can be frozen, in case a serious bug is found or there is a
serious key compromise. No transfers can take place while the contract is
paused. Access to the pause functionality is controlled by the `pauser` address.

### Upgradable

A new implementation contract can be deployed, and the proxy contract will
forward calls to the new contract. Access to the upgrade functionality is
guarded by a `proxyOwner` address. Only the `proxyOwner` address can change the
`proxyOwner` address.

### Blacklist

The contract can blacklist certain addresses which will prevent those addresses
from transferring or receiving tokens. Access to the blacklist functionality is
controlled by the `blacklister` address.

### Minting/Burning

Tokens can be minted or burned on demand. The contract supports having multiple
minters simultaneously. There is a `masterMinter` address which controls the
list of minters and how much each is allowed to mint. The mint allowance is
similar to the ERC20 allowance - as each minter mints new tokens their allowance
decreases. When it gets too low they will need the allowance increased again by
the `masterMinter`.

### Ownable

The contract has an Owner, who can change the `owner`, `pauser`, `blacklister`,
or `masterMinter` addresses. The `owner` can not change the `proxyOwner`
address.
