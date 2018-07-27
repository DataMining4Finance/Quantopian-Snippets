<!-- TOC START min:1 max:3 link:true update:true -->
  - [Use StaticAssets / Define your own Universe](#use-staticassets--define-your-own-universe)
  - [Technical Analysis indicators](#technical-analysis-indicators)
  - [Trading on Multiple Signals](#trading-on-multiple-signals)
  - [Custom-factors](#custom-factors)

<!-- TOC END -->


## Use StaticAssets / Define your own Universe

```py
# import StaticAssets  
from quantopian.pipeline.filters import  StaticAssets

# Create a static asset filter (here we want only the ETFs SPY and TLT  
my_etfs = StaticAssets(symbols(['SPY', 'TLT']))

# Use the filter as a screen when making a pipe definition or as a mask in a factor  
myPipe=Pipeline(  
        columns={"factor": factor,  
                 "ma_cross": ma_cross,  
                 },  
        screen = my_etfs  
        )
```

## Technical Analysis indicators

https://www.quantopian.com/help#sample-talib

```py
# This example algorithm uses the Relative Strength Index indicator as a buy/sell signal.
# When the RSI is over 70, a stock can be seen as overbought and it's time to sell.
# When the RSI is below 30, a stock can be seen as oversold and it's time to buy.

import talib
import numpy as np
import math

# Setup our variables
def initialize(context):
    context.max_notional = 100000
    context.intc = sid(3951)  # Intel
    context.LOW_RSI = 30
    context.HIGH_RSI = 70

    schedule_function(rebalance, date_rules.every_day(), time_rules.market_open())

def rebalance(context, data):

    #Get a trailing window of data
    prices = data.history(context.intc, 'price', 15, '1d')

    # Use pandas dataframe.apply to get the last RSI value
    # for for each stock in our basket
    intc_rsi = talib.RSI(prices, timeperiod=14)[-1]

    # Get our current positions
    positions = context.portfolio.positions

    # Until 14 time periods have gone by, the rsi value will be numpy.nan

    # RSI is above 70 and we own GOOG, time to close the position.
    if intc_rsi > context.HIGH_RSI and context.intc in positions:
        order_target(context.intc, 0)

        # Check how many shares of Intel we currently own
        current_intel_shares = positions[context.intc].amount
        log.info('RSI is at ' + str(intc_rsi) + ', selling ' + str(current_intel_shares) + ' shares')

    # RSI is below 30 and we don't have any Intel stock, time to buy.
    elif intc_rsi < context.LOW_RSI and context.intc not in positions:
        o = order_target_value(context.intc, context.max_notional)
        log.info('RSI is at ' + str(intc_rsi) + ', buying ' + str(get_order(o).amount)  + ' shares')

    # record the current RSI value and the current price of INTC.
    record(intcRSI=intc_rsi, intcPRICE=data.current(context.intc, 'close'))

```
## Trading on Multiple Signals

https://www.quantopian.com/posts/trading-on-multiple-ta-lib-signals

## Custom-factors

https://www.quantopian.com/help#custom-factors
