## Installation

```bash
conda install -c quantopian zipline
```

https://www.backtrader.com/docu/quickstart/quickstart.html#from-0-to-100-the-samples

## Basic Setup

Let’s get running.

```py
from __future__ import (absolute_import, division, print_function,
                        unicode_literals)

import backtrader as bt

if __name__ == '__main__':
    cerebro = bt.Cerebro()

    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())

    cerebro.run()

    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

## Setting the Cash
In the world of finance, for sure only “losers” start with 10k. Let’s change the cash and run the example again.

```py
import backtrader as bt

if __name__ == '__main__':
    cerebro = bt.Cerebro()
    cerebro.broker.setcash(100000.0)
```

## Adding a Data Feed
Having cash is fun, but the purpose behind all this is to let an automated strategy multiply the cash without moving a finger by operating on an asset which we see as a Data Feed

Ergo ... No Data Feed -> No Fun. Let’s add one to the ever growing example.

```py
import backtrader as bt

if __name__ == '__main__':
    # Create a cerebro entity
    cerebro = bt.Cerebro()

    # Create a Data Feed
    data = bt.feeds.YahooFinanceData(
        dataname="AAPL",
        # Do not pass values before this date
        fromdate=datetime.datetime(2000, 1, 1),
        # Do not pass values after this date
        todate=datetime.datetime(2000, 12, 31),
        reverse=False)

    # Add the Data Feed to Cerebro
    cerebro.adddata(data)
```

## Our First Strategy
The cash is in the broker and the Data Feed is there. It seems like risky business is just around the corner.

Let’s put a Strategy into the equation and print the “Close” price of each day (bar).

DataSeries (the underlying class in Data Feeds) objects have aliases to access the well known OHLC (Open High Low Close) daily values. This should ease up the creation of our printing logic.


## Adding some Logic to the Strategy
Let’s try some crazy idea we had by looking at some charts

If the price has been falling 3 sessions in a row ... BUY BUY BUY!!!

```py
from __future__ import (absolute_import, division, print_function,
                        unicode_literals)

import datetime  # For datetime objects
import os.path  # To manage paths
import sys  # To find out the script name (in argv[0])

# Import the backtrader platform
import backtrader as bt


# Create a Stratey
class TestStrategy(bt.Strategy):

    def log(self, txt, dt=None):
        ''' Logging function fot this strategy'''
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    def __init__(self):
        # Keep a reference to the "close" line in the data[0] dataseries
        self.dataclose = self.datas[0].close

    def next(self):
        # Simply log the closing price of the series from the reference
        self.log('Close, %.2f' % self.dataclose[0])

        if self.dataclose[0] < self.dataclose[-1]:
            # current close less than previous close

            if self.dataclose[-1] < self.dataclose[-2]:
                # previous close less than the previous close

                # BUY, BUY, BUY!!! (with all possible default parameters)
                self.log('BUY CREATE, %.2f' % self.dataclose[0])
                self.buy()


if __name__ == '__main__':
    # Create a cerebro entity
    cerebro = bt.Cerebro()

    # Add a strategy
    cerebro.addstrategy(TestStrategy)

    # Datas are in a subfolder of the samples. Need to find where the script is
    # because it could have been called from anywhere
    modpath = os.path.dirname(os.path.abspath(sys.argv[0]))
    datapath = os.path.join(modpath, '../../datas/orcl-1995-2014.txt')

    # Create a Data Feed
    data = bt.feeds.YahooFinanceCSVData(
        dataname=datapath,
        # Do not pass values before this date
        fromdate=datetime.datetime(2000, 1, 1),
        # Do not pass values before this date
        todate=datetime.datetime(2000, 12, 31),
        # Do not pass values after this date
        reverse=False)

    # Add the Data Feed to Cerebro
    cerebro.adddata(data)

    # Set our desired cash start
    cerebro.broker.setcash(100000.0)

    # Print out the starting conditions
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())

    # Run over everything
    cerebro.run()

    # Print out the final result
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

## The broker says: Show me the money! And the money is called “commission”.

Let’s add a reasonable 0.1% commision rate per operation (both for buying and selling ... yes the broker is avid ...)

A single line will suffice for it:
```py
cerebro.broker.setcommission(commission=0.001) # 0.1% ... divide by 100 to remove the %
```

## Customizing the Strategy: Parameters
It would a bit unpractical to hardcode some of the values in the strategy and have no chance to change them easily. Parameters come in handy to help.

Definition of parameters is easy and looks like:

```py
params = (('myparam', 27), ('exitbars', 5),)
```
Being this a standard Python tuple with some tuples inside it, the following may look more appealling to some:
```py
params = (
    ('myparam', 27),
    ('exitbars', 5),
)
```
With either formatting parametrization of the strategy is allowed when adding the strategy to the Cerebro engine:

## Add a strategy
```py
cerebro.addstrategy(TestStrategy, myparam=20, exitbars=7)
```

## Adding an indicator

```py
self.sma = bt.indicators.MovingAverageSimple(self.datas[0], period=self.params.maperiod)
```

## Visual Inspection: Plotting

```py
cerebro.plot()

# Indicators for the plotting show
bt.indicators.ExponentialMovingAverage(self.datas[0], period=25)
bt.indicators.WeightedMovingAverage(self.datas[0], period=25).subplot = True
bt.indicators.StochasticSlow(self.datas[0])
bt.indicators.MACDHisto(self.datas[0])
rsi = bt.indicators.RSI(self.datas[0])
bt.indicators.SmoothedMovingAverage(rsi, period=10)
bt.indicators.ATR(self.datas[0]).plot = False
```
