# Client Librari API

This document is intended to be a pseudocode outline of the interface provided for all client libraries interacting with The 0cean RESTful API.

### Trading methods


##### marketOrder
Places a market order for any available trading better than `price`.
* Returns details of fill
* Throws errors if not fillable

```javascript
marketOrder(tokenToSell,
            tokenToBuy,
            amountToBuy,
            statusUpdateCallback)
```

##### limitOrder
Places a market order for any available trading better than `price`.  Then posts an order to the order book for the remaining amount at `price`.
* Returns details of fill
* Throws errors if not fillable

```javascript
limitOrder(tokenToSell,
           tokenToBuy,
           amount,
           price,
           statusUpdateCallback)
```


### Public methods (read only)

##### tokenPairs
* Returns all tradeable pairs as base/quote

```javascript
assetPairs()
```

##### orderBook
* Returns bids and asks
* Throws errors if pair is not tradeable
* If `updateCallback` is specified, library should subscribe for updates and trigger callback when received

```javascript
orderBook(baseToken,
          quoteToken,
          depth,
          updateCallback)
```

##### candleSticks
* Returns high, low, open, close, volume for each candle stick of size interval
* Throws errors
* If `updateCallback` is specified, library should subscribe for updates and trigger callback when received

```javascript
candleSticks(baseToken,
             quoteToken,
             StartTime,
             endTime,
             interval,
             updateCallback)
```
