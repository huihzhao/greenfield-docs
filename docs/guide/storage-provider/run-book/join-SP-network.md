---
title: Join Greenfield SP Network
---

This guide will help you join Greenfield SP Network: Mainnet and Testnet.

- [How to Join SP Network?](#how-to-join-sp-network)
  - [1. Submit Proposal](#1-submit-proposal)
    - [Hot Wallet Manual](#hot-wallet-manual)
    - [Hardware Wallet Manual](#hardware-wallet-manual)
    - [Understanding the Parameters](#understanding-the-parameters)
  - [2. Deposit BNB to Proposal](#2-deposit-bnb-to-proposal)
  - [3. Wait Voting and Check Voting Result](#3-wait-voting-and-check-voting-result)
  - [4. Activate SP](#4-activate-sp)
    - [Storage Provider Standard Test](#storage-provider-standard-test)
    - [Update SP status](#update-sp-status)
- [Storage Provider Operations](#storage-provider-operations)
  - [EditStorageProvider](#editstorageprovider)
  - [Update SP Price](#update-sp-price)
  - [Update SP Quota](#update-sp-quota)
- [Tools](#tools)
- [Trouble Shooting](#trouble-shooting)

## How to Join SP Network?

Greenfield Blockchain validators are responsible for selecting storage providers. For each on-chain proposal to add new storage provider, there are deposit period for depositing BNB and voting period for validators to make votes. Once the proposal passes, new SP can join the network afterwards.

You can query the governance parameters [here](https://docs.bnbchain.org/greenfield-docs/docs/greenfield-api/gov-v-1-params).

### 1. Submit Proposal

The SP needs to initiate an on-chain proposal that specifies the Msg information to be automatically executed after the vote is approved. In this case, the Msg is `MsgCreateStorageProvider`. It's worth noting that the deposit tokens needs to be greater than the minimum deposit tokens specified on the chain.

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs
defaultValue="mainnet"
values={[
{label: 'Mainnet', value: 'mainnet'},
{label: 'Testnet', value: 'testnet'},
]}>
  <TabItem value="mainnet">

    rpcAddr = "https://greenfield-chain.bnbchain.org:443"
    chainId = "greenfield_1017-1"

  </TabItem>
  <TabItem value="testnet">

    rpcAddr = "https://gnfd-testnet-fullnode-tendermint-us.bnbchain.org:443"
    chainId = "greenfield_5600-1"
  </TabItem>
</Tabs>

#### Hot Wallet Manual

You can use the `gnfd` command to directly send the transaction for creating a storage provider. To do this,  please
import the private key of the funding account into the Keystore.

However, it is not safe to use a hot wallet for Mainnet. Instead, you should refer to the [Hardware Wallet Manual](#hardware-wallet-manual)
for instructions on using a hardware wallet.

Command for creating storage provider:
```shell
./build/bin/gnfd tx sp create-storage-provider ./create_storage_provider.json --from {funding_address} --node ${rpcAddr} --chain-id ${chainId} --keyring-backend os
```

The content for create_storage_provider.json, modify it with the correct values as you need:
```shell
cat ./create_storage_provider.json
{
  "messages":[
  {
    "@type":"/greenfield.sp.MsgCreateStorageProvider",
    "description":{
      "moniker":"{moniker}",
      "identity":"{identity}",
      "website":"{website}",
      "security_contact":"{security_contract}",
      "details":"{details}"
    },
    "sp_address":"{operator_address}",
    "funding_address":"{funding_address}",
    "seal_address":"{seal_address}",
    "approval_address":"{approval_address}",
    "gc_address":"{gc_address}",
    "maintenance_address":"{maintenance__address}",
    "endpoint":"https://{your_endpoint}",
    "deposit":{
      "denom":"BNB",
      # Mainnet: 500000000000000000000, Testnet: 1000000000000000000000
      "amount":"500000000000000000000"
    },
    "read_price":"0.1469890427",
    "store_price":"0.02183945725",
    "free_read_quota": 1073741824,
    "creator":"0x7b5Fe22B5446f7C62Ea27B8BD71CeF94e03f3dF2",
    "bls_key":"{bls_pub_key}",
    "bls_proof":"{bls_proof}"
  }
],
  "metadata":"4pIMOgIGx1vZGU=",
  "title":"Create <name> Storage Provider",
  "summary":"create <name> Storage Provider",
  "deposit":"1000000000000000000BNB"
}
```

#### Hardware Wallet Manual

The gnfd command is not available for connecting with the hardware wallet, so you should use the [gnfd-tx-sender](https://gnfd-tx-sender.nodereal.io/) to send transactions. Here are the steps:

1. Generate the transaction data.
```shell
./build/bin/gnfd tx sp create-storage-provider ./create_storage_provider.json --from {funding_address} --print-eip712-msg-type
```
2. Visit the [gnfd-tx-sender](https://gnfd-tx-sender.nodereal.io/) website.
3. Add your hardware wallet into Metamask, and connect the wallet.
4. Navigate to the `Custom Tx` page and fill in the generated transaction data in step1.
5. Click the `Submit` button to send the transaction.

![submit proposal](../../../../static/asset/019-submit-proposal.jpg)

#### Understanding the Parameters

:::note
You can get the gov module address by this command

```shell
curl -X GET "https://greenfield-chain-us.bnbchain.org/cosmos/auth/v1beta1/module_accounts/gov" -H  "accept: application/json"
```

:::

- `endpoint` is URL of your gateway
- `read_price` and `store_price` unit is `wei/bytes/s`
- `free_read_quota` unit is *Bytes*
- `creator` is the address of `gov module`
- `metadata` is optional

### 2. Deposit BNB to Proposal

:::note
You can get the mininum deposit for proposal by the above command. Please make sure that the initial deposit is greater than `min_deposit` when submitting the proposal.

```shell
curl -X GET "https://greenfield-chain-us.bnbchain.org/cosmos/gov/v1/params/deposit" -H  "accept: application/json"
```

:::

You can skip this step if the initial deposit amount is greater than the min deposit required by the proposal.

Each proposal needs to deposit enough tokens to enter the voting phase.

```shell
./build/bin/gnfd tx gov deposit ${proposal_id} 1BNB --from ${funding_address} --keyring-backend os --node ${rpcAddr} --chain-id ${chainId}
```

### 3. Wait Voting and Check Voting Result

After submitting the proposal successfully, you must wait for the voting to be completed and the proposal to be approved.
It will last **7 days** on Mainnet while **1 day** on Testnet. Once it has passed and is executed successfully, you can
verify that the storage provider has been joined.

:::caution

Please ensure that the storage provider service is running before it has been joined.

:::

You can check the on-chain SP information to confirm whether the SP has been successfully created.

```shell
./build/bin/gnfd query sp storage-providers --node ${rpcAddr}
```

Alternatively, you can check the proposal to know about its execution status.

```shell
./build/bin/gnfd query gov proposal ${proposal_id} --node ${rpcAddr}
```

### 4. Activate SP

#### Storage Provider Standard Test

After the proposal has passed, the status of SP is `STATUS_IN_MAINTENANCE`. To prevent being slashed due to functional
abnormalities, you should first perform a full functional test using the maintenance account.
You can refer to the [SP standard test](https://github.com/bnb-chain/greenfield-sp-standard-test).

#### Update SP status

Once the testing is completed, you need to send a tx to activate the SP to `STATUS_IN_SERVICE`.

```shell
gnfd tx sp update-status [sp-address] STATUS_IN_SERVICE [flags]
```

Refer to [Maintenance Mode](../../core-concept/storage-provider-lifecycle.md#in-maintenance) for more details.

## Storage Provider Operations

### EditStorageProvider

This command is used to edit the information of the SP, including endpoint, description, etc.

Usage:
```shell
gnfd tx sp edit-storage-provider [sp-address] [flags]
```

For example, edit the endpoint:
```shell
./build/bin/gnfd tx sp edit-storage-provider ${operator_address} --endpoint ${new_endpoint} --from ${operator_address} --keyring-backend os --node ${rpcAddr} --chain-id ${chainId}
```

### Update SP Price

Update the storage provider read, store price and free read quota, if there is no change to a specific value, the current value should also be provided.

The unit of price is a decimal, which indicates wei BNB per byte per second.
E.g. the price is 0.02183945725, means approximately $0.018 / GB / Month.
`(0.02183945725 * (30 * 86400) * (1024 * 1024 * 1024) * 300 / 10 ** 18 ≈ 0.018, assume the BNB price is 300 USD)`

The free-read-quota unit is bytes, for 1GB free quota, it should be 1073741824.

Usage:
```shell
gnfd tx sp update-price [sp-address] [read-price] [store-price] [free-read-quota] [flags]
```

Example:
```shell
./build/bin/gnfd tx sp update-price ${operator_address} 0.1469890427 0.02183945725 1073741824 --from ${operator_address} --keyring-backend os ---node ${rpcAddr} --chain-id ${chainId}
```

### Update SP Quota

Besides the above `update-price` command, you can also use the `gnfd-sp` command to update the free read quota for SP.
The update.quota command is used to update the free quota of the SP, it will send a transaction to the blockchain to update
the free read quota, but keep the storage price and read price unchanged.

Usage:
```shell
gnfd-sp update.quota [command options] [arguments...]
```

Example:
```shell
./build/bin/gnfd-sp update.quota --quota 1073741824 --config ./config.toml
```

## Tools

SP can use Greenfield Cmd or DCellar to verify its functions:

- Greenfield Cmd: Get more details from the [repo](https://github.com/bnb-chain/greenfield-cmd).
- DCellar: [Mainnet](https://dcellar.io/), [Testnet](https://testnet.dcellar.io).

## Trouble Shooting

If you meet issues, please refer to [SP common issues](./common-issues).
