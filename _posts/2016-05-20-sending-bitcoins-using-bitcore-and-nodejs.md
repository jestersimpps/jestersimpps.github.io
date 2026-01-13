---
layout: post
title: "Sending Bitcoins using Bitcore and Node.js"
description: "A comprehensive tutorial on creating a web app to send Bitcoin transactions using Bitcore library and Node.js."
date: 2016-05-20
author: "Jester Simpps"
categories: [Development]
tags: [Bitcore, Bitcoin, Bitpay, nodejs, Yeoman, cryptocurrency]
image: /images/blockchain1.jpg
---

This week I was asked to create a simple web app that allows users to send bitcoins from one address to another using node.js. After skimming through some online articles and forums I discovered [Bitcore](https://bitcore.io/api). I also looked at an alternative called [BitcoinJs](http://bitcoinjs.org/)

After comparing both, I decided to use bitcore as it seemed a little bit ahead of BitcoinJs and they also provide [nice documentation and development resources on their website](https://bitcore.io/api/lib/transaction#Transaction+checkedSerialize). Also, the company that open sourced Bitcore, Bitpay, is an established brand in the Bitcoin industry. BitPay is a payment processor that specializes in processing bitcoin payments and enabling merchants to accept bitcoin using a variety of website plugins and other integrations. Bitcore is their javascript library that allows developers to interface directly with the real bitcoin network.

## 1. Setting up the app

To kickstart my project, I looked for a suitable [Yeoman](http://yeoman.io/) generator. I had no desire setting up custom or advanced build automation for thisone. It would literally be a single page app that makes two simple API calls:

- Get the transaction history of a Bitcoin address
- Perform a transaction

I decided to use the [ng-fullstack](https://github.com/ericmdantas/generator-ng-fullstack#readme) generator as it seemed to have all the tools I like to work with, except for sass, which I discovered later, after the project was scaffolded. For a moment I was tempted to go with the Angular 2 configuration, but again... it's literally a 1 page app, so I just went with Angular 1.x. The generator includes a simple todo app, which is nice and it provides you some structure to work with.

After an initial UI overhaul, nothing remained of the todo app, but the UI was kept simple. We have 4 input fields, two address verification buttons and a send button in green.

- An input field for the origin address
- An input field for the recipient address
- An input field for the origin private key
- An input field for the amount of mBTC to transact

![kbt](/images/kbt2.png)

## 2. Fetching transaction data

When users enter a Bitcoin address, the app sends an api request to the [blockchain.info](https://blockchain.info) api and fetches the transaction history for the address. You can see the output of an example request to the Blockchain.info api [here](https://blockchain.info/address/15CrPRVdNUaXX1DCZqttnP21wyJLTTmy8y?format=json)

```
server
|    api
|         wallet
|               services
|                     wallet-service.js
```

```javascript
const url = 'https://blockchain.info/address/' + address + '?format=json';

request(url, function(error, response, body) {
  if (error) {
    return reject(error);
  }
  if (response.statusCode !== 200) {
    return reject(response.statusCode);
  }
  let balance = JSON.parse(body);
  resolve(balance);
});
```

Before sending actual requests, I verify Bitcoin addresses using a simple regex on the clientside:

```javascript
var BITCOINADDRESS = '(?:[13][1-9A-Za-z][^O0Il]{24,32})';
```

and the [bitcoin-address](https://github.com/defunctzombie/bitcoin-address) library on the server side.

```javascript
import bitcoinaddress from 'bitcoin-address';

if (!bitcoinaddress.validate(address)) {
  return reject('Address checksum failed');
}
```

When the address data arrives in my angular client, I display the transaction history table with the dates, transaction results and hashes like so:
![kbt](/images/kbt3.png)

## 3. Creating Bitcoin transactions

To create a Bitcoin transaction, we need 4 things:

- The Bitcoin address of the sender
- The private key of the sender
- A recipient address
- The amount of Bitcoins to be sent

The information from the input boxes will be checked by a transaction model on the client side and is then sent to the `wallet-service.js` in the back end. To create transactions using bitcore, we first need to install the bitcore and bitcore-explorers libraries using npm:

```bash
npm i --save bitcore-lib && bitcore-explorers
```

And reference them so we can use them:

```javascript
import bitcore from 'bitcore-lib';
import explorers from 'bitcore-explorers';
```

### 3.1 Getting the sender's balance

Before creating a transaction, we have to do some checks.

Firstly, we have to take in account the miner fee, which is the price we have to pay to send the transaction to the Bitcoin Blockchain. This fee can directly influence the time it takes for a transaction to be confirmed on the blockchain. At the moment of writing, the approximate minimum mining fee is 12,800 satoshis (0.05$). ([https://bitcoinfees.21.co/](https://bitcoinfees.21.co/))

Secondly, we need to check if the private key can sign the inputs. This is done by the bitcore library and is pretty complex, so I will not expand on this, but you can find more on signing transactions [here](http://www.righto.com/2014/02/bitcoins-hard-way-using-raw-bitcoin.html).

So, our first check will be to see if the sender address balance can at least cover the mining fee. The `bitcore-explorers` library includes [insight](https://insight.is/), which is an open-source bitcoin blockchain API that will help us get information regarding our senders' address. To determine the balance of the sender address, we need to go over the unspent outputs of the sender.

> Bitcoin works on the concept of discrete inputs and outputs not spending part of a balance. All transactions have as their input a reference to a previous unspent output. Each transaction records one or more new outputs (which are referenced in the inputs of some future transactions). Outputs are "spent" when they are referenced in a new transaction. Outputs can only be unspent or spent, they can't be partially spent. Wallet balance (or address balance) is an abstraction to help us humans and make Bitcoin more like conventional payment systems. Balances are not used at the protocol level. When the wallet indicates your confirmed balance is 1.2 BTC it is saying that the sum of the value of all unspent outputs in the blockchain which correspond to public keys it has the private key for total 1.2 BTC. In other words the wallet is computing the total value of the outputs which it can spend which requires a) the output be unspent and b) the client has the private key necessary to spend it. ~ Pieter Wuille

More information on utxos and balances can be found in [this answer on stackexchange](http://bitcoin.stackexchange.com/questions/22997/how-is-a-wallets-balance-computed)

Using insight, we can return the array of utxos or Unspent Transaction Outputs:

```javascript
const insight = new explorers.Insight();
```

```javascript
insight.getUnspentUtxos(transaction.fromaddress, function(error, utxos) {
  console.log(utxos);
});
```

The output in the terminal should be something like:

![kbt](/images/kbt4.png)

The sum of the utxos is basically the balance on the senders' Bitcoin address, so we can now try the following:

```javascript
insight.getUnspentUtxos(transaction.fromaddress, function(error, utxos) {
  let balance = 0;
  for (var i = 0; i < utxos.length; i++) {
    balance +=utxos[i]['satoshis'];
  }
  console.log('balance:'+ balance);
});
```

Which, in my case, returned a balance of 105042196 satoshis:

```bash
[0] balance:105042196
```

### 3.2 Error handling and input validation

Let's expand our code, create some constants for the miner fee and the transaction amount and include some error handling. We will also make use of the bitcore Unit utility to define and convert our minerfee, transaction amount and balance variables:

```javascript
const unit = bitcore.Unit;
const insight = new explorers.Insight();
const minerFee = unit.fromMilis(0.128).toSatoshis();
const transactionAmount = unit.fromMilis(transaction.amount).toSatoshis();
```

```javascript
insight.getUnspentUtxos(transaction.fromaddress, function(error, utxos) {

  if (error) {
    return reject(error);
  } else {

    if (utxos.length == 0) {
      return reject("You don't have enough Satoshis to cover the miner fee.");
    }

    let balance = unit.fromSatoshis(0).toSatoshis();
    for (var i = 0; i < utxos.length; i++) {
      balance += unit.fromSatoshis(parseInt(utxos[i]['satoshis'])).toSatoshis();
    }

    if (balance - transactionAmount - minerFee > 0) {
      //our transaction code will come here
    } else {
      return reject("You don't have enough Satoshis to cover the miner fee.");
    }

  }
});
```

I am also defining the same mining fee in the transaction model on the client side so the send button only becomes enabled once the amount entered exceeds the miner fee:

```javascript
var MININGFEE = 0.12800; //mining fee in mBTC
```

Let's also validate the origin and recipient addresses in the backend before any api requests happen.

```javascript
if (!bitcoinaddress.validate(transaction.fromaddress)) {
  return reject('Origin address checksum failed');
}
if (!bitcoinaddress.validate(transaction.toaddress)) {
  return reject('Recipient address checksum failed');
}
```

### 3.3 Creating a new transaction and serialization

This was a bit of a struggle for me at first since it was the first time I worked with the bitcore library and discovered that the error handling was a bit flaky. Hence, I discovered the best way to get around this was to surround the transaction code with a try catch and make use of [getSerializationError](https://bitcore.io/api/lib/transaction#Transaction+getSerializationError).

```javascript
try {
  let bitcore_transaction = new bitcore.Transaction()
    .from(utxos)
    .to(transaction.toaddress, transactionAmount)
    .fee(minerFee)
    .change(transaction.fromaddress)
    .sign(transaction.privatekey);

  if (bitcore_transaction.getSerializationError()) {
    let error = bitcore_transaction.getSerializationError().message;
    switch (error) {
      case 'Some inputs have not been fully signed':
        return reject('Please check your private key');
        break;
      default:
        return reject(error);
    }
  }
} catch (error) {
  return reject(error.message);
}
```

I decided to create a switch case so I could rewrite possible errors in my own language as I found `'Some inputs have not been fully signed'` not very clear towards the end user.

The actual bitcore transaction takes the following inputs:

```javascript
let bitcore_transaction = new bitcore.Transaction()
  .from(utxos)
  .to(transaction.toaddress, transactionAmount)
  .fee(minerFee)
  .change(transaction.fromaddress)
  .sign(transaction.privatekey);
```

As you can see I have set the change address to the senders' address. Because outputs can only be unspent or spent, and not partially spent, there's a concept of "change address" in the bitcoin ecosystem:

> If an output of 10 BTC is available for me to spend, but I only need to transmit 1 BTC, I'll create a transaction with two outputs, one with 1 BTC that I want to spend, and the other with 9 BTC to a change address, so I can spend this 9 BTC with another private key that I own

[Here](http://bitzuma.com/posts/five-ways-to-lose-money-with-bitcoin-change-addresses/) is a very interesting article on change addresses which explains the concept very well. If you haven't heard of change addresses and are involved in bitcoin transactions in any way, it's definitely a recommended read.

### 3.4 Broadcasting the transaction to the blockchain

Now that our transaction has been signed and created, we can broadcast it to the blockchain. As we are not running our own bitcoin node, this will happen via the [insight api](https://bitcore.io/api/explorers/).

```javascript
insight.broadcast(bitcore_transaction, function(error, body) {
  if (error) {
    reject('Error in broadcast: ' + error);
  } else {
    resolve({transactionId: body});
  }
});
```

When we put everything together, the entire transaction function looks like this:

```javascript
static createTransaction = (transaction) => {
  return new Promise((resolve, reject) => {

    const unit = bitcore.Unit;
    const insight = new explorers.Insight();
    const minerFee = unit.fromMilis(0.128).toSatoshis();
    const transactionAmount = unit.fromMilis(transaction.amount).toSatoshis();

    if (!bitcoinaddress.validate(transaction.fromaddress)) {
      return reject('Origin address checksum failed');
    }
    if (!bitcoinaddress.validate(transaction.toaddress)) {
      return reject('Recipient address checksum failed');
    }

    insight.getUnspentUtxos(transaction.fromaddress, function(error, utxos) {
      if (error) {
        return reject(error);
      } else {

        if (utxos.length == 0) {
          return reject("You don't have enough Satoshis to cover the miner fee.");
        }

        let balance = unit.fromSatoshis(0).toSatoshis();
        for (var i = 0; i < utxos.length; i++) {
          balance += unit.fromSatoshis(parseInt(utxos[i]['satoshis'])).toSatoshis();
        }

        if ((balance - transactionAmount - minerFee) > 0) {

          try {
            let bitcore_transaction = new bitcore.Transaction()
              .from(utxos)
              .to(transaction.toaddress, transactionAmount)
              .fee(minerFee)
              .change(transaction.fromaddress)
              .sign(transaction.privatekey);

            if (bitcore_transaction.getSerializationError()) {
              let error = bitcore_transaction.getSerializationError().message;
              switch (error) {
                case 'Some inputs have not been fully signed':
                  return reject('Please check your private key');
                  break;
                default:
                  return reject(error);
              }
            }

            insight.broadcast(bitcore_transaction, function(error, body) {
              if (error) {
                reject('Error in broadcast: ' + error);
              } else {
                resolve({
                  transactionId: body
                });
              }
            });

          } catch (error) {
            return reject(error.message);
          }
        } else {
          return reject("You don't have enough Satoshis to cover the miner fee.");
        }
      }
    });
  });
}
```

We are done! Now we should be able to send Bitcoins using the bitcore library and node.js. You can find code of the entire app in the following github repo: [https://github.com/jestersimpps/bitcoin-transact](https://github.com/jestersimpps/bitcoin-transact).
