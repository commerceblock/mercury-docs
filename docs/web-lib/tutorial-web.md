---
sidebar_position: 1
---

# Tutorial - Mercury web-app library

This tutorial demonstrates the usage of the mercury web library in a simple react app using [vitejs.dev](http://vitejs.dev). 

## Requirements

This tutorial assumes access to a command line terminal with [npm](https://www.npmjs.com/) installed, and a `WASM` compatible web browser. 

## Web app

Initialise a react demo-app using vite:

```
npm create vite@latest demo-app -- --template react
```

Then run:

```
cd demo-app
```

This creates the following files in the `demo-app` directory:

```
README.md
eslint.config.js
index.html
node_modules
package-lock.json
package.json
public
src/
    App.css
    App.jsx
    assets
    index.css
    main.jsx
vite.config.js
```

First, add the `mercuryweblib` to the `package.json` file:

```
"dependencies": {
    "mercuryweblib": "0.0.3",
}
```

Save the file and run: `npm install` to install the library and dependencies. 

Then create a a config file `ClientConfig.js` in the `src/` directory, with:

```js
const clientConfig = {
  esploraServer: "https://mutinynet.com",
  statechainEntity: "https://test.mercurylayer.com",
  network: "signet",
  feeRateTolerance: 5,
  confirmationTarget: 2,
  maxFee: 1
};

export default clientConfig;
```

Then edit the file `src/App.jsx`:

At the very top of the file, add the following two line:

```js
import mercuryweblib from 'mercuryweblib';
import clientConfig from './ClientConfig';
```

Immediately after, the line `function App() {` add the following lines:

```js
const [inputWallet, setInputWallet] = useState('');
const [inputAmount, setInputAmount] = useState('1000');
const [inputStatechainId, setInputStatechainId] = useState('');
const [inputToAddress, setInputToAddress] = useState('');
const [batchId, setBatchId] = useState('');
const [isBatchTransfer, setBatchTransfer] = useState(false);
```

Then add wallet functions immediately after this section:

```js
const createWallet = async () => {
  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  await mercuryweblib.createWallet(clientConfig, inputWallet);
  console.log(`wallet ${inputWallet} created`);

  setInputWallet('');
};

const newToken = async () => {
  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  await mercuryweblib.newToken(clientConfig, inputWallet);
  console.log(`token created in the wallet ${inputWallet}` );

  setInputWallet('');
};

const getDepositBitcoinAddress = async () => {
  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  if (inputAmount === '') {
    console.log('Please enter an amount');
    return;
  }

  const parsedAmount = parseInt(inputAmount, 10);

  if (isNaN(parsedAmount)) {
    console.error(`Error: Unable to convert "${inputAmount}" to an integer.`);
  }

  let btcAddr = await mercuryweblib.getDepositBitcoinAddress(clientConfig, inputWallet, parsedAmount);
  console.log(`Address from wallet ${inputWallet}`);
  console.log(btcAddr);

  setInputWallet('');
};

const listStatecoins = async () => {
  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  const coins = await mercuryweblib.listStatecoins(clientConfig, inputWallet);

  console.log(`Coins from wallet ${inputWallet}`);
  console.log(coins);

  setInputWallet('');
};

const withdrawCoin = async () => {
  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  if (inputStatechainId === '') {
    console.log('Please enter a statechain id');
    return;
  }

  if (inputToAddress === '') {
    console.log('Please enter a recipient address');
    return;
  }

  const txid = await mercuryweblib.withdrawCoin(clientConfig, inputWallet, inputStatechainId, inputToAddress, null);

  console.log(`Withdraw txid: ${txid}`);

  setInputWallet('');
  setInputStatechainId('');
  setInputToAddress('');
};

const broadcastBackupTransaction = async () => {

  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  if (inputStatechainId === '') {
    console.log('Please enter a statechain id');
    return;
  }

  if (inputToAddress === '') {
    console.log('Please enter a recipient address');
    return;
  }

  const txids = await mercuryweblib.broadcastBackupTransaction(clientConfig, inputWallet, inputStatechainId, inputToAddress, null);

  console.log("Txids:");
  console.log(txids);

  setInputWallet('');
  setInputStatechainId('');
  setInputToAddress('');
};

const newTransferAddress = async () => {
  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  const transferAddress = await mercuryweblib.newTransferAddress(inputWallet, isBatchTransfer);

  console.log(`Transfer address from wallet ${inputWallet}`);
  console.log(transferAddress);

  setInputWallet('');
};

const transferSend = async () => {

  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  if (inputStatechainId === '') {
    console.log('Please enter a statechain id');
    return;
  }

  if (inputToAddress === '') {
    console.log('Please enter a recipient address');
    return;
  }

  let batchIdToSend = null;

  if (!!batchId) {
    batchIdToSend = batchId;
  }

  console.log(`batchIdToSend: ${batchIdToSend}`);

  const coin = await mercuryweblib.transferSend(clientConfig, inputWallet, inputStatechainId, inputToAddress, batchIdToSend);
  console.log("Coin:");
  console.log(coin);

  setInputWallet('');
  setInputStatechainId('');
  setInputToAddress('');
};

const transferReceive = async () => {

  if (inputWallet === '') {
    console.log('Please enter a wallet name');
    return;
  }

  const received_statechain_ids = await mercuryweblib.transferReceive(clientConfig, inputWallet);
  console.log("received_statechain_ids:");
  console.log(received_statechain_ids);

  setInputWallet('');
};

const handleIsBatchTransferChange = (event) => {
  setBatchTransfer(event.target.checked);
};
```

Then finally add these HTML components (replace everything in the `return( ...  )` section):

```js
    <>
      <div>
        <a href="https://vitejs.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>

      <div className="card">
        <input
          type="text"
          value={inputWallet}
          onChange={(e) => setInputWallet(e.target.value)}
          placeholder="Enter wallet name"
          style={{ marginRight: '10px' }}
        />

        <input
          type="text"
          value={inputAmount}
          onChange={(e) => setInputAmount(e.target.value)}
          placeholder="Enter amount"
          style={{ marginRight: '10px' }}
        />

        <input
          type="text"
          value={inputStatechainId}
          onChange={(e) => setInputStatechainId(e.target.value)}
          placeholder="Enter statechain id"
          style={{ marginRight: '10px' }}
        />

        <input
          type="text"
          value={inputToAddress}
          onChange={(e) => setInputToAddress(e.target.value)}
          placeholder="Enter recipient address"
          style={{ marginRight: '10px' }}
        />

        <input
          type="text"
          value={batchId}
          onChange={(e) => setBatchId(e.target.value)}
          placeholder="Enter batchId"
          style={{ marginRight: '10px' }}
        />


      <label>
        <input
          type="checkbox"
          checked={isBatchTransfer}
          onChange={handleIsBatchTransferChange}
        />
        Is Batch Transfer ?
      </label>

      </div>

      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <button onClick={() => createWallet()}>
          Create Wallet
        </button>
        <button onClick={() => newToken()}>
          New Token
        </button>
        <button onClick={() => getDepositBitcoinAddress()}>
          New Deposit Address
        </button>
        <button onClick={() => listStatecoins()}>
          List Statecoins
        </button>
        <button onClick={() => withdrawCoin()}>
          Withdraw
        </button>
        <button onClick={() => broadcastBackupTransaction()}>
          Broadcast Backup Transaction
        </button>
        <br  />
        <button onClick={() => newTransferAddress()} style={{ marginTop: '10px' }}>
          New Transfer Address
        </button>
        <button onClick={() => transferSend()} style={{ marginTop: '10px' }}>
          Transfer Send
        </button>
        <button onClick={() => transferReceive()} style={{ marginTop: '10px' }}>
          Transfer Receive
        </button>
        <p>
          Edit <code>src/App.jsx</code> and save to test HMR
        </p>
      </div>
      
      <p className="read-the-docs">
        Click on the Vite and React logos to learn more
      </p>
    </>
```

To run the demo-app in a browser:

```
npm run dev
```
