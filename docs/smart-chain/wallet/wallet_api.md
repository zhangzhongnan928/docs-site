Binance Chain Wallet injects a global API into websites visited by its users at `window.BinanceChain`.

This API borrowed heavily from API metamask provided considered the massive adoption. So web3 site developer could easily connect their app with the Binance Chain Wallet. It allows websites to request users' Binance Smart Chain accounts, read data from the blockchain the user is connected to, and suggest that the user sign messages and transactions.
The presence of the provider object indicates an Binance Chain user.

The API this extension wallet provides includes API specified by [EIP-1193](https://eips.ethereum.org/EIPS/eip-1193) and API defined by [MetaMask](https://docs.metamask.io/guide/ethereum-provider.html) (including some massively relied legacy ones).

## Table of Contents

## Upcoming Breaking Changes

!!! warning

    Important Information

    On **November 16, 2020**, we are making changes to our provider API that will be breaking for some web3 sites.
    These changes are _upcoming_, but you can prepare for them today.
    Follow [this GitHub issue](https://github.com/MetaMask/metamask-extension/issues/8077) for updates.

    All consumers of MetaMask's provider may be affected by the `eth_chainId` bug (see [next subsection](#window-ethereum-api-changes)).
    Other than that, if you are new to using the provider, you do not have to worry about these changes, and can skip ahead [to the next section](#api).


## Basic Usage

For any non-trivial Binance Smart Chain web application — a.k.a. web3 site — to work, you will have to:

- Detect the Binance Smart Chain provider (`window.BinanceChain`)
- Detect which Binance Smart Chain network the user is connected to
- Get the user's Binance Smart Chain account(s)

The snippet at the top of this page is sufficient for detecting the provider.
You can learn how to accomplish the other two by reviewing the snippet in the [Using the Provider section](#using-the-provider).

The provider API is all you need to create a full-featured web3 application.

That said, many developers use a convenience library, such as [ethers](https://www.npmjs.com/package/ethers) and [web3.js](https://www.npmjs.com/package/web3), instead of using the provider directly.
If you are in need of higher-level abstractions than those provided by this API, we recommend that you use a convenience library.

## Chain IDs

::: warning
At the moment, the [`BinanceChain.chainId`](#ethereum-chainid) property and the [`chainChanged`](#chainchanged) event should be preferred over the `eth_chainId` RPC method.
Their chain ID values are correctly formatted, per the table below.

`eth_chainId` returns an incorrectly formatted (0-padded) chain ID for the chains in the table below, e.g. `0x01` instead of `0x1`.
See the [Upcoming Breaking Changes section](#upcoming-breaking-changes) for details on when the `eth_chainId` RPC method will be fixed.

Custom RPC endpoints are not affected; they always return the chain ID specified by the user.
:::

These are the IDs of the Binance Smart chains that Binance Chain Wallet supports by default.

| Hex  | Decimal | Network                                    |
| ---- | ------- | ------------------------------------------ |
| 0x38 | 56      | Binance Smart Chain Main Network (MainNet) |
| 0x61 | 97      | Binance Smart Chain Test Network           |

## Properties

### BinanceChain.chainId

!!! warning

    The value of this property can change at any time, and should not be exclusively relied upon. See the [`chainChanged`](#chainchanged) event for details.

**NOTE:** See the [Chain IDs section](#chain-ids) for important information about the Binance Chain Wallet provider's chain IDs.


A hexadecimal string representing the current chain ID.

### BinanceChain.autoRefreshOnNetworkChange

As the consumer of this API, it is your responsbility to handle chain changes using the [`chainChanged` event](#chainChanged).
We recommend reloading the page on `chainChange` unless you have good reason not to.


By default, this property is `true`.

If this property is truthy, Binance Chain Wallet will reload the page in the following cases:

- When the connected chain (network) changes, if `window.BinanceChain` has been accessed during the page lifecycle
- When `window.BinanceChain` is accessed, if the connected chain (network) has changed during the page lifecycle

To disable this behavior, set this property to `false` immediately after detecting the provider:

```javascript
BinanceChain.autoRefreshOnNetworkChange = false;
```

## Methods

### BinanceChain.isConnected()

!!! tip

    Note that this method has nothing to do with the user's accounts.

    You may often encounter the word "connected" in reference to whether a web3 site can access the user's accounts.
    In the provider interface, however, "connected" and "disconnected" refer to whether the provider can make RPC requests to the current chain.


```typescript
BinanceChain.isConnected(): boolean;
```

Returns `true` if the provider is connected to the current chain, and `false` otherwise.

If the provider is not connected, the page will have to be reloaded in order for connection to be re-established.
Please see the [`connect`](#connect) and [`disconnect`](#disconnect) events for more information.

### BinanceChain.request(args)

```typescript
interface RequestArguments {
  method: string;
  params?: unknown[] | object;
}

BinanceChain.request(args: RequestArguments): Promise<unknown>;
```

Use `request` to submit RPC requests to Binance Smart Chain via Binance Chain Wallet.
It returns a `Promise` that resolves to the result of the RPC method call.

The `params` and return value will vary by RPC method.
In practice, if a method has any `params`, they are almost always of type `Array<any>`.

If the request fails for any reason, the Promise will reject with an [Ethereum RPC Error](#errors). (Binance Smart Chain shares same RPC Error)

Binance Chain Wallet supports most standardized Ethereum RPC methods, in addition to a number of methods that may not be
supported by other wallets.
See the Binance Chain Wallet [RPC API documentation](./rpc-api.html) for details.

#### Example

```javascript
params: [
  {
    from: '0xb60e8dd61c5d32be8058bb8eb970870f07233155',
    to: '0xd46e8dd67c5d32be8058bb8eb970870f07244567',
    gas: '0x76c0', // 30400
    gasPrice: '0x9184e72a000', // 10000000000000
    value: '0x9184e72a', // 2441406250
    data:
      '0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675',
  },
];

BinanceChain
  .request({
    method: 'eth_sendTransaction',
    params,
  })
  .then((result) => {
    // The result varies by by RPC method.
    // For example, this method will return a transaction hash hexadecimal string on success.
  })
  .catch((error) => {
    // If the request fails, the Promise will reject with an error.
  });
```

## Events

The BinanceChain provider implements the [Node.js `EventEmitter`](https://nodejs.org/api/events.html) API.
This sections details the events emitted via that API.
There are innumerable `EventEmitter` guides elsewhere, but you can listen for events like this:

```javascript
BinanceChain.on('accountsChanged', (accounts) => {
  // Handle the new accounts, or lack thereof.
  // "accounts" will always be an array, but it can be empty.
});

BinanceChain.on('chainChanged', (chainId) => {
  // Handle the new chain.
  // Correctly handling chain changes can be complicated.
  // We recommend reloading the page unless you have a very good reason not to.
  window.location.reload();
});
```

### connect

```typescript
interface ConnectInfo {
  chainId: string;
}

BinanceChain.on('connect', handler: (connectInfo: ConnectInfo) => void);
```

The Binance Chain Wallet provider emits this event when it first becomes able to submit RPC requests to a chain.
We recommend using a `connect` event handler and the [`BinanceChain.isConnected()` method](#BinanceChain-isconnected) in order to determine when/if the provider is connected.

### disconnect

```typescript
BinanceChain.on('disconnect', handler: (error: ProviderRpcError) => void);
```

The MetaMask provider emits this event if it becomes unable to submit RPC requests to any chain.
In general, this will only happen due to network connectivity issues or some unforeseen error.

Once `disconnect` has been emitted, the provider will not accept any new requests until the connection to the chain has been re-restablished, which requires reloading the page.
You can also use the [`BinanceChain.isConnected()` method](#BinanceChain-isconnected) to determine if the provider is disconnected.

### accountsChanged

```typescript
BinanceChain.on('accountsChanged', handler: (accounts: Array<string>) => void);
```

The Binance Chain Wallet provider emits this event whenever the return value of the `eth_accounts` RPC method changes.
`eth_accounts` returns an array that is either empty or contains a single account address.
The returned address, if any, is the address of the most recently used account that the caller is permitted to access.
Callers are identified by their URL _origin_, which means that all sites with the same origin share the same permissions.

This means that `accountsChanged` will be emitted whenever the user's exposed account address changes.

!!! tip

    We plan to allow the `eth_accounts` array to be able to contain multiple addresses in the near future.

### chainChanged

!!! warning
    **NOTE:** See the [Chain IDs section](#chain-ids) for important information about the Binance Chain Wallet provider's chain IDs.


```typescript
BinanceChain.on('chainChanged', handler: (chainId: string) => void);
```

The Binance Chain Wallet provider emits this event when the currently connected chain changes.

All RPC requests are submitted to the currently connected chain.
Therefore, it's critical to keep track of the current chain ID by listening for this event.

We _strongly_ recommend reloading the page on chain changes, unless absolutely necessary not to.

```javascript
BinanceChain.on('chainChanged', (_chainId) => window.location.reload());
```

### message

```typescript
interface ProviderMessage {
  type: string;
  data: unknown;
}

BinanceChain.on('message', handler: (message: ProviderMessage) => void);
```

The Binance Chain Wallet provider emits this event when it receives some message that the consumer should be notified of.
The kind of message is identified by the `type` string.

RPC subscription updates are a common use case for the `message` event.
For example, if you create a subscription using `eth_subscribe`, each subscription update will be emitted as a `message` event with a `type` of `eth_subscription`.

## Errors

All errors thrown or returned by the Binance Chain Wallet provider follow this interface:

```typescript
interface ProviderRpcError extends Error {
  message: string;
  code: number;
  data?: unknown;
}
```

The [`BinanceChain.request(args)` method](#BinanceChain-request-args) throws errors eagerly.
You can often use the error `code` property to determine why the request failed.
Common codes and their meaning include:

* `4001`

    * The request was rejected by the user

* `-32602`

    * The parameters were invalid

* `-32603`

    * Internal error

For the complete list of errors, please see [EIP-1193](https://eips.ethereum.org/EIPS/eip-1193#provider-errors) and [EIP-1474](https://eips.ethereum.org/EIPS/eip-1474#error-codes).

!!! tip
    The [`eth-rpc-errors`](https://npmjs.com/package/eth-rpc-errors) package implements all RPC errors thrown by the MetaMask provider, and can help you identify their meaning.


## Using the Provider

This snippet explains how to accomplish the three most common requirements for web3 sites:

- Detect the BinanceChain provider (`window.BinanceChain`)
- Detect which BinanceChain network the user is connected to
- Get the user's BinanceChain account(s)


## Legacy API

!!! warning
    You should **never** rely on any of these methods, properties, or events in practice.

This section documents MetaMask's legacy provider API.

To be compatible with existing dApps that support MetaMask, Binance Chain Wallet implement them as well, but please don't rely on them. We may deprecate them soon in future.

## Legacy Properties

### BinanceChain.networkVersion (DEPRECATED)

!!! warning
    You should always prefer the chain ID over the network ID.

    If you must get the network ID, use [`BinanceChain.request({ method: 'net_version' })`](#BinanceChain-request-args).

    The value of this property can change at any time.


A decimal string representing the current blockchain's network ID.

### BinanceChain.selectedAddress (DEPRECATED)

!!! warning
    Use [`BinanceChain.request({ method: 'eth_accounts' })`](#BinanceChain-request-args) instead.

    The value of this property can change at any time.


Returns a hexadecimal string representing the user's "currently selected" address.

The "currently selected" address is the first item in the array returned by `eth_accounts`.

## Legacy Methods

### BinanceChain.enable() (DEPRECATED)

!!! warning
    Use [`BinanceChain.request({ method: 'eth_requestAccounts' })`](#BinanceChain-request-args) instead.


Alias for `BinanceChain.request({ method: 'eth_requestAccounts' })`.

### BinanceChain.sendAsync() (DEPRECATED)

!!! warning
    Use [`BinanceChain.request()`](#BinanceChain-request-args) instead.


```typescript
interface JsonRpcRequest {
  id: string | undefined;
  jsonrpc: '2.0';
  method: string;
  params?: Array<any>;
}

interface JsonRpcResponse {
  id: string | undefined;
  jsonrpc: '2.0';
  method: string;
  result?: unknown;
  error?: Error;
}

type JsonRpcCallback = (error: Error, response: JsonRpcResponse) => unknown;

BinanceChain.sendAsync(payload: JsonRpcRequest, callback: JsonRpcCallback): void;
```

This is the ancestor of `BinanceChain.request`. It only works for JSON-RPC methods, and takes a JSON-RPC request payload object and an error-first callback function as its arguments.

See the [Ethereum JSON-RPC API](https://eips.ethereum.org/EIPS/eip-1474) for details.

### BinanceChain.send() (DEPRECATED)

!!! warning
    Use [`BinanceChain.request()`](#BinanceChain-request-args) instead.


```typescript
BinanceChain.send(
  methodOrPayload: string | JsonRpcRequest,
  paramsOrCallback: Array<unknown> | JsonRpcCallback,
): Promise<JsonRpcResponse> | void;
```

This method behaves unpredictably and should be avoided at all costs.
It is essentially an overloaded version of [`BinanceChain.sendAsync()`](#BinanceChain-sendasync-deprecated).

`BinanceChain.send()` can be called in three different ways:

```typescript
// 1.
BinanceChain.send(payload: JsonRpcRequest, callback: JsonRpcCallback): void;

// 2.
BinanceChain.send(method: string, params?: Array<unknown>): Promise<JsonRpcResponse>;

// 3.
BinanceChain.send(payload: JsonRpcRequest): unknown;
```

You can think of these signatures as follows:

1. This signature is exactly like `BinanceChain.sendAsync()`

2. This signature is like an async `BinanceChain.sendAsync()` with `method` and `params` as arguments, instead of a JSON-RPC payload and callback

3. This signature enables you to call the following RPC methods synchronously:

   - `eth_accounts`
   - `eth_coinbase`
   - `eth_uninstallFilter`
   - `net_version`

## Legacy Events

### close (DEPRECATED)

!!! warning
    Use [`disconnect`](#disconnect) instead.


```typescript
BinanceChain.on('close', handler: (error: Error) => void);
```

### chainIdChanged (DEPRECATED)

!!! warning
    Use [`chainChanged`](#chainchanged) instead.


Misspelled alias of [`chainChanged`](#chainchanged).

```typescript
BinanceChain.on('chainChanged', handler: (chainId: string) => void);
```

### networkChanged (DEPRECATED)

!!! warning
    Use [`chainChanged`](#chainchanged) instead.


Like [`chainChanged`](#chainchanged), but with the `networkId` instead.
Network IDs are insecure, and were effectively deprecated in favor of chain IDs by [EIP-155](https://eips.ethereum.org/EIPS/eip-155).
Avoid using them unless you know what you are doing.

```typescript
BinanceChain.on('networkChanged', handler: (networkId: string) => void);
```

### notification (DEPRECATED)

!!! warning
    Use [`message`](#message) instead.


```typescript
BinanceChain.on('notification', handler: (payload: any) => void);
```
