# Client Librari API

This document is intended to be a pseudocode outline of the interface provided for all client libraries interacting with The 0cean RESTful API.

### Trading methods
Trading methods will all require authentication with an API key.

##### limitOrder
```javascript
limitOrder(tokenToSell,
           tokenToBuy,
           amount,
           price)
```
Creates, signs and submits a 0x order to The 0cean's specification.

##### marketOrder
```javascript
marketOrder(tokenToSell,
            tokenToBuy,
            amountRequested,
            maxPrice)
```

Queries The 0cean's orderbook and creates matching orders up to the amounts specified if possible.



### Public methods (read only)
Public methods require no authentication.

##### assetPairs
```javascript
assetPairs()
```

##### orderBook
```javascript
orderBook(tokenToSell,
          tokenToBuy)
```

##### candleSticks
```javascript
candleSticks(tokenToSell,
             tokenToBuy,
             StartTime,
             endTime,
             period)
```

### Subscription
