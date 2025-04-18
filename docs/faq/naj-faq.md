---
id: naj-faq
title: FAQs for NEAR-API-JS
sidebar_label: JS Library FAQ
---

## General


### Can I use `near-api-js` on a static html page?

> Yep! See examples below:

**Example snippet:**

```html
  <script src="https://cdn.jsdelivr.net/npm/near-api-js@0.41.0/dist/near-api-js.min.js"></script>
```

<details>
<summary>**Example Implementation:** </summary>
<p>

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>

<body>
  <ul id="messages"></ul>
  <textarea id="text" placeholder="Add Message"></textarea>
  <button id="add-text">Add Text</button>
  <script src="https://cdn.jsdelivr.net/npm/near-api-js@0.41.0/dist/near-api-js.min.js"></script>
  <script>
    // connect to NEAR
    const near = new nearApi.Near({
      keyStore: new nearApi.keyStores.BrowserLocalStorageKeyStore(),
      networkId: 'testnet',
      nodeUrl: 'https://rpc.testnet.near.org',
      walletUrl: 'https://wallet.testnet.near.org'
    });
    
    // connect to the NEAR Wallet
    const wallet = new nearApi.WalletConnection(near, 'my-app');

    // connect to a NEAR smart contract
    const contract = new nearApi.Contract(wallet.account(), 'guest-book.testnet', {
      viewMethods: ['getMessages'],
      changeMethods: ['addMessage']
    });

    const button = document.getElementById('add-text');
    if (!wallet.isSignedIn()) {
      button.textContent = 'SignIn with NEAR'
    }

    // call the getMessages view method
    contract.getMessages()
      .then(messages => {
        const ul = document.getElementById('messages');
        messages.forEach(message => {
          const li = document.createElement('li');
          li.textContent = `${message.sender} - ${message.text}`;
          ul.appendChild(li);
        })
      });

    // Either sign in or call the addMessage change method on button click
    document.getElementById('add-text').addEventListener('click', () => {
      if (wallet.isSignedIn()) {
        contract.addMessage({
          args: { text: document.getElementById('text').value },
          amount: nearApi.utils.format.parseNearAmount('1')
        })
      } else {
        wallet.requestSignIn({
          contractId: 'guest-book.testnet',
          methodNames: ['getMessages', 'addMessage']
        });
      }
    });
  </script>
</body>

</html>
```

</p>
</details>

---

## Contracts

### How do I attach gas / a deposit?

> After [contract is instantiated](/docs/api/naj-quick-reference#load-contract) you can then call the contract and specify the amount of attached gas.

```js
await contract.method_name(
  {
    arg_name: "value", // argument name and value - pass empty object if no args required
  },
  300000000000000, // attached GAS (optional)
  1000000000000000000000000 // attached deposit in yoctoNEAR (optional)
);
```

---

## Testing

### How are Jest Tests Configured?

> Check out the `package.json` file of some of our examples, particularly near the bottom where there's this block:

```json
  "jest": {
    "testEnvironment": "near-cli/test_environment",
    "testPathIgnorePatterns": [
      "<rootDir>/contract/",
      "<rootDir>/node_modules/"
    ]
  }
```

_(Here we're using a custom environment which may be something most web2 developers haven't had to do.)_

---

## Common Errors

### Missing contract method

> Missing a contract method when trying to call a contract? Check to see if you added the view or change methods when instantiating your contract.

**Example:**
```js
const contract = await new nearAPI.Contract(
  walletConnection.account(),
  'guest-book.testnet',
  {
    viewMethods: ["getMessages"],  
    changeMethods: ["addMessage"], 
    sender: walletConnection.getAccountId(),
  }
);
```

---

### `regeneratorRuntime` is not defined

> You are probably using [Parcel](https://parceljs.org/) like we do in [other examples](https://near.dev). Please make sure you have this line at the top of your main JS file. (Most likely `index.js`):

```js
import "regenerator-runtime/runtime";
```

- Also, ensure the dependencies for this are added to the project by checking the dev dependencies in your `package.json`. If not found you can install this by running the following in your terminal:

```bash
npm install regenerator-runtime --save-dev
```

---

### Window error using `Node.js`

> You maybe using a KeyStore that's for the browser. Instead use a filesystem key or private key string. [See methods here](/docs/api/naj-quick-reference#key-store)

**Browser KeyStore:**

```js
const { keyStores } = require("near-api-js");
const keyStore = new keyStores.BrowserLocalStorageKeyStore();
```

**FileSystem KeyStore:**

```js
const { keyStores } = require("near-api-js");
const KEY_PATH = "~./near-credentials/testnet/example-account.json";
const keyStore = new keyStores.UnencryptedFileSystemKeyStore(KEY_PATH);
```
