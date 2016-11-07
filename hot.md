# Getting started
First of all you will need an Ethereum account, that will serve as a hot wallet.
You can generate one safely at: [MyEtherWallet](https://www.myetherwallet.com/#generate-wallet)

  * Type whatever password (don't need to remember it)
  * Click Generate
  * Copy and save **Private Key (unencrypted)** to a secure place. **Attention** anyone who knows this private key,
has access to your entire account
  * Copy and save **Your Address** somewhere. This will serve as your **Hot Wallet Address**
  * It is mandatory to have a separate Hot Wallet Address for every asset
  * Maintain some ETH on the balance of your Hot Wallet Address.

Preconditions:

  * You know asset of interest contract address, e.g. [CryptoCarbon](https://explorer.cryptocarbon.co.uk/#/asset/CC) **0xe4c94d45f7aef7018a5d66f44af780ec6023378e**
  * You know asset of interest ICAP currency code, e.g. **BCC** for CryptoCarbon.

## Balance
This section will explain on how to get balance of your Hot Wallet Address.
### [Server-side Examples](https://github.com/CryptoCarbon/etoken-server-side-examples)
Preconditions:

  * You know asset of interest base unit(decimals) number, found at [Assets](https://explorer.cryptocarbon.co.uk/#/assets), e.g. CryptoCarbon **6**
  * Set **config.js** parameters **address**(asset's contract address) and **baseUnit** accordingly.

```javascript
  'address': '0xe4c94d45f7aef7018a5d66f44af780ec6023378e', // CC
  'baseUnit': 6,
```
You can get balance with, replacing the **{address}** with your Hot Wallet Address:
```bash
node get-balance.js {address}
```
Check out example usages for **Ruby** and **PHP** in the repo, if neccessary.
Preconditions:

  * Start ETokenD with the following parameters: **--assetaddress 0xe4c94d45f7aef7018a5d66f44af780ec6023378e --asset BCC --privatekey {PrivateKeyOfHotWallet}**

You can get balance with the following JSON RPC request:
```bash
curl localhost:18545 -d '{"jsonrpc":"2.0","method":"getbalance","params":[],"id":0}'
```
## Receive deposits
This section will explain on how to get paid.
### [Notificator](http://etoken-tutorial.readthedocs.io)
Preconditions:

  * You know your company ICAP institution code, e.g. **MYCO**, request through contacting [support@cryptocarbon.co.uk](mailto:support@cryptocarbon.co.uk)
  * Asset of interest ICAP currency code with your company ICAP institution code is bound to your Hot Wallet Address, request through contacting [support@cryptocarbon.co.uk](mailto:support@cryptocarbon.co.uk)
  * Start ETokenD with the following parameters:

```
--assetaddress 0xe4c94d45f7aef7018a5d66f44af780ec6023378e
--asset BCC
--institution {ICAPInstitutionCode}
--privatekey {PrivateKeyOfHotWallet}
```

#### Deposits
Call **getnewaddress** once for each user account:
```bash
curl localhost:18545 -d '{"jsonrpc":"2.0","method":"getnewaddress","params":[],"id":0}'
```
Assign resulting deposit address to the user account, and ask user to do deposits to this same address every time.
#### Invoices
Call **getnewaddress** once for each new invoice:
```bash
curl localhost:18545 -d '{"jsonrpc":"2.0","method":"getnewaddress","params":[],"id":0}'
```
Assign resulting address to the invoice, and ask user to do invoice payment to this address.
#### Process payment
To know when payment arrived you can setup [Notificator](http://etoken-tutorial.readthedocs.io), deposit/invoice address can be accessed through **notification.eventData.icap**.
Another option is to poll ETokenD for incoming transactions:
```bash
curl localhost:18545 -d '{"jsonrpc":"2.0","method":"listtransactions","params":[],"id":0}'
```
Deposit/invoice address can be accessed through **transaction.icap**.

**Note**: payment's unique id is **transactionHash + logIndex**, make sure to save it in order to not double process the same payment.
## Send payouts
This section will explain how to do payouts.
Preconditions:

  * You know asset of interest base unit(decimals) number, found at [Assets](https://explorer.cryptocarbon.co.uk/#/assets), e.g. CryptoCarbon **6**
  * Set **config.js** parameters **address**(asset's contract address) and **baseUnit** accordingly.

```javascript
  'address': '0xe4c94d45f7aef7018a5d66f44af780ec6023378e', // CC
  'baseUnit': 6,
  'icapAssetCode': 'BCC',
  'privateKey': {PrivateKeyOfHotWallet}
```
You can send funds, replacing the **{destination}** with the recipient address or ICAP, and **{amount}** with desired amount(1.1 will result in 1100000 CC being sent):
```bash
node send-tx.js {destination} {amount}
```
Check out example usages for **Ruby** and **PHP** in the repo, if neccessary.
### [ETokenD](http://etokend-docs.cryptocarbon.co.uk), can be bought in [CryptoCarbon Store](https://www.cryptocarbon.co.uk/product/list)
Preconditions:

  * Start ETokenD with the following parameters: **--assetaddress 0xe4c94d45f7aef7018a5d66f44af780ec6023378e --asset BCC --privatekey {PrivateKeyOfHotWallet}**

Call **sendtoaddress** replacing the **{destination}** with the recipient address or ICAP, and **{amount}** with desired amount(1.1 will result in 1100000 CC being sent):
```bash
curl localhost:18545 -d '{"jsonrpc":"2.0","method":"sendtoaddress","params":["{destination}","{amount}"],"id":0}'
```
## ETH through EToken
This section will explain how to work with ETH.
### Setup
First you need to make your Hot Wallet Address activated to accept ETH in a WEI4 way:

  * Open [CryptoCarbon Ethereum Utils](https://rawgit.com/CryptoCarbon/etoken-lib/master/utils/index.html)
  * Press F12 to open a Developer Console
  * Execute the following code snippet replacing the **privatekey** in the end with a private key of your Hot Wallet Address:

```javascript
(function(key) {
  setPrivateKey(key);
  var wei4 = eth.contract([
    {
      "constant": false,
      "inputs": [
        {
          "name": "_enabled",
          "type": "bool"
        }
      ],
      "name": "setAutoDeposit",
      "outputs": [
        {
          "name": "",
          "type": "bool"
        }
      ],
      "type": "function"
    }
  ]).at('0x7660727d3cb947e807acead927ef3ede24c4a18d');
  log('Starting the activation process. It may take up to a couple of minutes.', $logs});
  safeTransactions([
    safeTransactionFunction(wei4.setAutoDeposit, [true], address, {waitReceipt: true}),
    syncFunction(function() {
      log(address + ' has been successfully activated to accept ETH in a WEI4 way.', $logs});
      log('You can close this page and proceed with the guide.', $logs});
    })
  ]);
})(privatekey);
```

  * To check balance, follow the [Balance](hot.md#balance) guide using the **WE4** as ICAP currency code, and **0x7660727d3cb947e807acead927ef3ede24c4a18d** as contract address, base unit is **0**. Meaning that **1 WE4** equals **0.000000000000000001 ETH**.
  * To send payouts, follow the [Send payouts](hot.md#send-payouts) guide using the **WE4** as ICAP currency code, and **0x7660727d3cb947e807acead927ef3ede24c4a18d** as contract address, base unit is **0**. Meaning that **1 WE4** equals **0.000000000000000001 ETH**.

### Receive ETH deposits
Preconditions:

  * You know your company ICAP institution code, e.g. **MYCO**, request through contacting [noreply@cryptocarbon.co.uk](mailto:support@cryptocarbon.co.uk)
  * WE4 asset with your company ICAP institution code is bound to your Hot Wallet Address, request through contacting [noreply@cryptocarbon.co.uk](mailto:noreply@cryptocarbon.co.uk)
  * Start EtherSweeper with the following parameters: **--secret {secretPhraseToEncryptPrivateKeys}**
  * Checkout EtherSweeper docs, keep the secret secure and backup state file regularly.

The process is as follows:
  
  1. Generate new ICAP address for deposit/invoice, one way is through [ETokenD](hot.md#deposits), another is to do it programmatically
  2. Assign resulting ICAP address to the deposit/invoice
  3. Call **addroute** replacing the **{destination}** with the address from previous step:
```bash
curl localhost:28545 -d '{"jsonrpc":"2.0","method":"addroute","params":["{destination}"],"id":0}'
```
  4. In response you will get simple ETH address, show it to user to do deposit or pay invoice. **Note**: payments of less than 0.01 ETH won't be sweeped, cause that amount is reserved for the sweep itself
  5. [Process payment](hot.md#process-payment).

********