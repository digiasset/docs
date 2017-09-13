# The ~~0~~cean RESTful API - v0.2
##### This API specification is for The 0cean and will be offered in addition to the [0x standard relayer API](https://github.com/0xProject/standard-relayer-api)

## Terminology
* **Funnel Address** : The Funnel Address is an Ethereum address controlled by The 0cean.  Any orders (matching or target) submitted to the The 0cean's orderbook must specify it as the taker.  
* **Target Order** : A 0x order submitted as an offer for a trade of tokens.  Its placed in the orderbook and publicized along with all other target orders.
* **Matching Order** : A 0x order submitted as a request to accept a target order.  If a target order is available, both will be filled in an atomic fashion by the Funnel Address.

## Important considerations
* **Units** : All units sent through the RESTful api should be in base units.  So, if you want to communicate 3.2 ZRX tokens, you would send the string `3200000000000000000`.  This will be abstracted out in the endpoints of the client libraries.
* **Order Excess** If a matching order is submitted without the hash for the corresponding target order (see POST /v0/match), it will be filled as much as possible from the existing order book.  Because prices are unlikely to be exactly equal, this will probably result in a small amount of tokens as a byproduct of the atomic transaction.  Ownership of these tokens will be transferred to the Funnel Address (as a result of the 0x smart contract) and it is up to the discretion of The 0cean what to do with them.  If this bothers you, please specify your target order.

## Methods
The following methods are provided as a means of interfacing with the Funnel Address model of 0x relayer.
#### Errors
* **Error Codes**--coming soon

#### GET /v0/order_book
Returns open orders in order book sorted by price for specified asset pair as well as required fees for order.  It is assumed that orders will be matched in order from best price to worst and the fees correspond to that.

_PARAMETERS:_
* **makerToken**  `string` the symbol or address of the token to be acquired    ie   “ZRX”
* **takerToken**   `string` the symbol or address of the token offered ie   "WETH”
* **bothSides**  `boolean` includes the other half of the order book (the back half).  Ie. offering ZRX to acquire WETH
* **priceWindow** - float  - the max difference from the best price.  Applies to the back half as well as the front


`TODO` : We probably don't need to send all this info, especially things that are definitely 0 or aren't relevant to the matcher

_Returns_
```javascript
{
front: [
    {
      price: '<>',
      order : {
        exchange: '<>',                     // Ethereum address
        maker: '<>',                        // Ethereum address
        taker: '<>',                        // Ethereum address for the Funnel Address
        makerToken: '<>',                   // ERC20 contract address
        takerToken: '<>',                   // ERC20 contract address
        feeRecipient: '<>',                 // Ethereum address provided by The 0cean
        makerTokenAmount: '<>',             // Amount on offer when order was made
        takerTokenAmount: '<>',             // Amount to be paid when order was made
        makerFee: '<>',                     // 0
        takerFee: '<>',                     // 0
        expirationTimestampInSec: '<>',     // Unix timestamp in seconds
        salt: '<>',                         // Generated
        orderHash: '<>',                    // Generated
        ecSignature: { v: '<>', r: '<>', s: '<>' },     // Generated
        amountRemaining: <0 to takerTokenAmount>        // Total taker tokens available
      },
      requiredFee : '<>'        // The fee that is required for the matching order.             
                                // Given in ZRX base units for the full order amount
    },
    ……
  ],
  back?: [/*<other side of book if applicable>*/]
}
```


#### POST /v0/match
Accepts signed orders and optionally the orders to which they should be matched.
* For complete order fills:  
  * Maker and Taker amounts must match exactly
  * Fee must match exactly.
* For partial order fills:
  * Price must meet or exceed 1/given price
  * Fee must meet or exceed the portion being filled
* For untargeted match requests:
  * `targetOrderHash` field is omitted.
  * Order will be filled as much as possible from existing
orderbook
  *   This may result in orders that do not perfectly align and the excess tokens will be consumed by the Funnel Address. (See 'Order Excess' above)


* `target order price`  = `target taker amount` / `target maker amount`
* `matching order price` = `matching taker amount` / `limit maker amount`
* `target order price` > 1 / `matching order price`


_Payload:_
* **matchRequests** - array of signed orders with optional hash of the `target order`.  
```javascript
{
  matchRequests:
  [
    {
      order : {
        exchange: '<>',
        maker: '<>',
        taker: '<>',
        makerToken: '<>',
        takerToken: '<>',
        feeRecipient: '<>',
        makerTokenAmount: '<>',
        takerTokenAmount: '<>',
        makerFee: '<>',
        takerFee: '<>',
        expirationTimestampInSec: '<>',
        salt: '<>',
        orderHash: '<>',
        ecSignature: { v: <>, r: '<>', s: '<>' },
        state: 'open|expired|filled|unfunded|pending',
        amountRemaining: <0 to takerTokenAmount>
      },
      targetOrderHash? : ‘<>’
    },
    ...
  ]
}
```
_Returns_
* Array of objects with the transaction hash, hashes of any filled orders and the token amounts of the match.
* Empty array if no orders are matched
```javascript
{
  orderResults : [
    {
      transactionHash: '<>',
      matchingOrderHash: '<>',  // The order submitted to match
      targetOrderHashes: ['<>', '<>', ...],  // All orders for this transaction
      tokenSent: '<>',          // MakerToken in the matching order
      tokenReceived: '<>',      // TakerToken in the matching order
      amountSent: '<>',         // TakerTokenAmount in the matching order (if the entire order was filled)
      amountReceived: '<>',     // MakerTokenAmount in the matching order (if the entire order was filled)   
      Fee: '<>'                 // The fee collected in ZRX base units
    }, …
  ]
}
```

#### POST /v0/cancel
Accepts an order hash and amount to cancel.  Depending on authentication, this may also require a signed statement from the maker of the order.
* If the amount exceeds the remaining amount of the order, the entire order will be canceled
* If the amount will leave less than the minimum amount of an order for this asset pair, the entire order will be canceled

* `orderHash`  = order hash of existing order in book
* `takerAmount` = the taker amount to cancel`


_Payload:_
```javascript
{
  orderHash: '<>',
  takerAmount: '<>'
}
```

*Returns: The amount canceled*  May not match amount submitted if the entire order is canceled (see above)
```javascript
{ amountCanceled: '<>'}
```
