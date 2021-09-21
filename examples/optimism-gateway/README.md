
## Optimism gateway ENS resolver example

## Summary

This example is a port of https://github.com/ensdomains/l2gateway-demo

## The flow

### When A user update smart contract on l2

- l1 state root gets updated
- `OVM_StateCommitmentChain` on l1 emit events

### When A user query l2 data via Durin

- The client library first calls `resolver.addr()` which returns Durin gateway url
- Durin gets the latest state batch header by `StateBatchAppended` events on listening to OVM_StateCommitmentChain contract.
- Durin constructs MerkleTree based on the state roots
- Durin calls l2 `eth_getProof` to fetch the first storage slot of the `OptimismResolver` contract where `mapping(bytes32=>address) addresses` is stored
- Durin returns the `stateBatchHeader`

```js
{
  stateBatchHeader: {
    batch: {
      batchIndex: [BigNumber],
      batchRoot: '0x2fe1dc132260ee7b45a468d9f84e1d1cecf02dba8e16e0646b59c2b11b5f4fa9',
      batchSize: [BigNumber],
      prevTotalElements: [BigNumber],
      extraData: '0x00000000000000000000000000000000000000000000000000000000614a3b3200000000000000000000000070997970c51812dc3a010c7d01b50e0d17dc79c8'
    },
    stateRoots: [
      '0x2fe1dc132260ee7b45a468d9f84e1d1cecf02dba8e16e0646b59c2b11b5f4fa9'
    ]
  }
}
```

- Once the client library receives the response, calls `resolver.addrWithProof` with the response

## How to set up

### 1. Setup optimism environment

Follow [the Optimism guide to create a node locally](https://community.optimism.io/docs/developers/l2/dev-node.html#creating-a-node)

This will start up l2 on port 8545 and l1 on port 9545.
All the subsequent accounts are generated from the following seed

```
test test test test test test test test test test test junk
```

### 2. clone the repo

```
git clone https://github.com/ensdomains/durin
cd examples/optimism-gateway
```

### 3. Deploy Resolver contracts

```
cd contracts
cd yarn
yarn deploy
```

This will deploy `OptimismResolver` into l2 and `OptimismResolverStub` to l1

```
~/.../optimism-gateway/contracts (optimism)$yarn deploy
yarn run v1.22.10
$ yarn deploy:l2 && yarn deploy:l1
$ IS_OPTIMISM=true npx hardhat --network optimistic run scripts/l2deploy.js
OptimismResolver deployed to 0x2279B7A0a67DB372996a5FaB50D91eAA73d2eBe6
Address set
$ npx hardhat --network integration run scripts/deploy.js
{ RESOLVER_ADDRESS: '0x2279B7A0a67DB372996a5FaB50D91eAA73d2eBe6' }
ENS registry deployed at 0xb7278A61aa25c888815aFC32Ad3cC52fF24fE575
OptimismResolverStub deployed at 0x7969c5eD335650692Bc04293B07F5BF2e7A673C0
```

### 4. Start gateway server


```
cd ../server
yarn
yarn start
```

### 4. Run client script

The demo demonstrates the followings

- 1. get the address of `test.test` via Durin
- 2. get the address of `test2.test` via Durin
- 3. Update the record of `test.test` to `0x...1` on l2
- 4. get the updated address of `test.test` via Durin

```
cd ../client
yarn
yarn start
```

```
yarn run v1.22.10
$ yarn build && node dist/index.js
$ tsdx build
@rollup/plugin-replace: 'preventAssignment' currently defaults to false. It is recommended to set this option to `true`, as the next major version will default this option to `true`.
@rollup/plugin-replace: 'preventAssignment' currently defaults to false. It is recommended to set this option to `true`, as the next major version will default this option to `true`.
✓ Creating entry file 1.2 secs
✓ Building modules 1.8 secs
{ RESOLVER_STUB_ADDRESS: '0xB06c856C8eaBd1d8321b687E188204C1018BC4E5' }
Ask durin for test.test
*** resolver.addr error: missing revert data in call exception (error={"reason":"processing response error","code":"SERVER_ERROR","body":"{\"jsonrpc\":\"2.0\",\"id\":44,\"error\":{\"code\":-32603,\"message\":\"Error: Transaction reverted without a reason string\"}}","error":{"code":-32603},"requestBody":"{\"method\":\"eth_call\",\"params\":[{\"to\":\"0xb06c856c8eabd1d8321b687e188204c1018bc4e5\",\"data\":\"0x3b3b57de28f4f6752878f66fd9e3626dc2a299ee01cfe269be16e267e71046f1022271cb\"},\"latest\"],\"id\":44,\"jsonrpc\":\"2.0\"}","requestMethod":"POST","url":"http://localhost:9545"}, data="0x", code=CALL_EXCEPTION, version=providers/5.4.5)
[ '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266' ]
Ask durin for test2.test
*** resolver.addr error: missing revert data in call exception (error={"reason":"processing response error","code":"SERVER_ERROR","body":"{\"jsonrpc\":\"2.0\",\"id\":48,\"error\":{\"code\":-32603,\"message\":\"Error: Transaction reverted without a reason string\"}}","error":{"code":-32603},"requestBody":"{\"method\":\"eth_call\",\"params\":[{\"to\":\"0xb06c856c8eabd1d8321b687e188204c1018bc4e5\",\"data\":\"0x3b3b57de28a0aea25f12a9cdf05dea70993899ec1bd771ced7ea789ffd733b1feaec1c21\"},\"latest\"],\"id\":48,\"jsonrpc\":\"2.0\"}","requestMethod":"POST","url":"http://localhost:9545"}, data="0x", code=CALL_EXCEPTION, version=providers/5.4.5)
[ '0x70997970C51812dc3A010C7d01b50e0d17dc79C8' ]
Update test.test on l2
Set new value to l2 0x0000000000000000000000000000000000000001
Wait 10 sec
Ask durin again
*** resolver.addr error: missing revert data in call exception (error={"reason":"processing response error","code":"SERVER_ERROR","body":"{\"jsonrpc\":\"2.0\",\"id\":52,\"error\":{\"code\":-32603,\"message\":\"Error: Transaction reverted without a reason string\"}}","error":{"code":-32603},"requestBody":"{\"method\":\"eth_call\",\"params\":[{\"to\":\"0xb06c856c8eabd1d8321b687e188204c1018bc4e5\",\"data\":\"0x3b3b57de28f4f6752878f66fd9e3626dc2a299ee01cfe269be16e267e71046f1022271cb\"},\"latest\"],\"id\":52,\"jsonrpc\":\"2.0\"}","requestMethod":"POST","url":"http://localhost:9545"}, data="0x", code=CALL_EXCEPTION, version=providers/5.4.5)
[ '0x0000000000000000000000000000000000000001' ]
✨  Done in 15.50s.
```

## TODO

- Extract gateway url from exception rather than hardcoding = Pending on [hardhat to fix the bug](https://github.com/nomiclabs/hardhat/issues/1882)


## Open questions

- What happens 