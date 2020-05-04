# Telos Free Account Creation Contract

One of the promises of the Telos Blockchain Network is to provide the first 1,000,000 accounts for free. To help make this promise a reality, this smart contract was designed to support the creation of free Telos accounts for users taking advantage of tools such as Sqrl. In this document, we briefly describe how the Free Account Creation smart contract works.

There are four actions allowed in this contract:
* `configure( int16_t max_accounts_per_hour, int64_t stake_cpu_tlos_amount, int64_t stake_net_tlos_amount )`
* `addwhitelist( name account_name )`
* `removewlist( name account_name )`
* `create( name account_creator, name account_name, string owner_key, string active_key, string key_prefix )`
* `createby( name account_creator, name account_name, string owner_key, string active_key )`

## Configuring Account Creation Limits
To minimize the creation of spam accounts on the Telos Network, the `configure` action accepts a `max_accounts_per_hour` value to limit the number of free accounts that can be created every 60 minutes. This defaults to 50 accounts. It also allows you to specified the amount of TLOS to delegate to new accounts for CPU and NET via `stake_cpu_tlos_amount` and `stake_net_tlos_amount`, respectively. These default to `0.90 TLOS` for CPU and `0.10 TLOS` for NET. Please note that these values must be in the correct format (e.g. 10000 = 1.000 TLOS, 1000 = 0.1000 TLOS, 100 = 0.0100 TLOS, etc)

## Whitelist
To prevent abuse, this Free Account Creation smart contract requires the authority of the `account_creator` supplied in `create` and also requires the `account_creator` account to be in the whitelist of approved accounts. The following two actions allows you to add or remove approved accounts to the whitelist:

* `addwhitelist( name account_name )` - gives `account_name` access to create accounts using this smart contract
* `removewlist( name account_name )` - removes `account_name`'s access to create accounts using this smart contract

## Creating Accounts
The `create` action requires four (4) parameters before an account can be successfully created.

`account_creator` - this is the 12-character name of the account with permission create accounts

`account_name` - this is the 12-character name of the new account

`owner_key` - this is the owner public key used to associate with the account

`active_key` - this is the active public key used to associate with the account

`key_prefix` - this is the prefix for the owner and active public keys. Set to 'EOS' for chains using EOS public key prefixes

The `createby` action is the same as the `create` action only does not require the `key_prefix` parameter and it will record the new account as created by the `account_creator`.  This requires the `account_creator@active` permission (or a permission linked to `eosio::newaccount`) to be granted to `free.tf@eosio.code`.

## How It Works
Once the smart contract has been configured with `max_accounts_per_hour`, `stake_cpu_tlos_amount`, and `stake_net_tlos_amount` values, calls to the `create` action performs the following steps:

1. Verify that the number of accounts created within the last hour does not exceed `max_accounts_per_hour`
2. Validates the owner and active public keys
3. Allocates RAM, NET and CPU resources for the account. The amount of TLOS needed to purchase RAM for new accounts is calculated in real-time and is based on the current `rammarket` pricing. The amount of TLOS delegated to CPU is derived from the `configure` table's `stake_cpu_tlos_amount` value and `stake_net_tlos_amount` for NET
4. The system's `newaccount`, `buyram`, and `delegatebw` actions are then called with the aforementioned settings
5. An entry is created in the `freeacctlogs` table for auditing







