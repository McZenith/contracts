# EOSIS Article Publisher

## Basics

Study: [Getting Started with Smart Contracts](https://developers.eos.io/eosio-cpp/docs/introduction-to-smart-contracts)

Practice: [Testnet](https://github.com/meet-one/Testnet)

## Create Accounts

~~~
$ cleos create key --to-console
Private key: 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
Public key: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

$ cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
~~~

**Note:** Be sure to use the actual key value generated by the cleos command and not the one shown in the example above!

**Note:** These are test only keys and should never be used for the production blockchain.

~~~
$ cleos create account eosio eosioarticle EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

$ cleos get accounts EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
{
  "account_names": [
    "eosio.token",
    "eosioarticle"
  ]
}

$ cleos push action eosio.token transfer '["eosio","eosioarticle","618.0000 EOS", "m"]' -p eosio
executed transaction: 434108132815a6ee3ac3261ce32bf67434314ef762fa88aa183f07feb60c87fb  128 bytes  1315 us
#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"eosioarticle","quantity":"618.0000 EOS","memo":"m"}
#         eosio <= eosio.token::transfer        {"from":"eosio","to":"eosioarticle","quantity":"618.0000 EOS","memo":"m"}
#  eosioarticle <= eosio.token::transfer        {"from":"eosio","to":"eosioarticle","quantity":"618.0000 EOS","memo":"m"}

$ cleos get currency balance eosio.token eosioarticle EOS
618.0000 EOS
~~~

When you test online, you need buyram:

~~~
$ cleos -u <TestnetURL> system buyram eosioarticle eosioarticle "10 EOS"
~~~

## Build

~~~
$ ./build.sh
~~~

Or

~~~
contract='eosioarticle'

eosiocpp -o ${contract}.wast ${contract}.cpp
eosiocpp -g ${contract}.abi ${contract}.cpp
~~~

## Deploy

~~~
$ cleos set contract ${contract} ../${contract} -p ${contract}@active
Reading WASM from ../eosioarticle/eosioarticle.wasm...
Publishing contract...
executed transaction: 86d771ba38471b3c6545f368fac9f96a825a5dc4e09062f749f26fc549f3b1df  6112 bytes  829 us
#         eosio <= eosio::setcode               {"account":"eosioarticle","vmtype":0,"vmversion":0,"code":"0061736d01000000017f1460067f7e7e7f7f7f006...
#         eosio <= eosio::setabi                {"account":"eosioarticle","abi":"0e656f73696f3a3a6162692f312e3000040c61727469636c655f646174610005026...
~~~

You can combine build and deploy, by:

~~~
./build_deploy.sh
~~~

## Action

#### publish

Publish an article on Multi-Index DB. (You must pay for RAM)

Parameters:

- owner
- title
- author
- body

#### edit

Edit an article on Multi-Index DB by id.

Parameters:

- id
- owner
- title
- author
- body

#### record

Publish an article on blockchain. You can't edit it later. (Free)

Parameters:

- title
- author
- body

#### post

Post a message to an account. You can't edit it later. (Free)

Parameters:

- to
- title
- body

## Test

~~~
$ cleos push action eosioarticle publish '["eosioarticle","Title","UMU618","Main"]' -p eosioarticle
executed transaction: 32e6608340eeb27a188695c83d46a6f682586b3dd56d24076a75d8d6fcefd34a  120 bytes  846 us
#  eosioarticle <= eosioarticle::publish        {"owner":"eosioarticle","title":"Title","author":"UMU618","body":"Main"}

$ cleos get table eosioarticle eosioarticle articles
{
  "rows": [{
      "id": 0,
      "owner": "eosioarticle",
      "title": "Title",
      "author": "UMU618",
      "body": "Main"
    }
  ],
  "more": false
}

$ cleos push action eosioarticle edit '[0,"eosioarticle","Modify","UMU618","Other text"]' -p eosioarticle
executed transaction: 77ad1e77c2b172a45cdbebdc5afcc4e30f8c1c802ced301046d930920d3ae6a5  136 bytes  3154 us
#  eosioarticle <= eosioarticle::edit           {"id":0,"owner":"eosioarticle","title":"Modify","author":"UMU618","body":"Other text"}

$ cleos push action eosioarticle record '["Record","UMU618","Main"]' -p eosioarticle
executed transaction: dffeb3db6b157c66c32bab7d1b5216e5089a8c48d75ff30a1e677f1a64934bfb  112 bytes  562 us
#  eosioarticle <= eosioarticle::record         {"title":"Record","author":"UMU618","body":"Main"}

$ cleos push action eosioarticle post '["eosio","Hi","Hello, eosio!"]' -p eosioarticle
executed transaction: 541769298a34966f72933777bb952b5d996f90a27cb87bb29263bcc8a24758ba  120 bytes  451 us
#  eosioarticle <= eosioarticle::post           {"to":"eosio","title":"Hi","body":"Hello, eosio!"}
#         eosio <= eosioarticle::post           {"to":"eosio","title":"Hi","body":"Hello, eosio!"}
~~~