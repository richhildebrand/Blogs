## Introduction

While practicing Python, I stumbled across [interviewcake](https://www.interviewcake.com) which has a lot of really well explained algorithm focus questions. I highly recommed trying them out!

Here is my solution to one of their questions, but for maximum value you will want to subscribe to get their step by step break down.

## The Problem

Suppose we could access yesterday's stock prices as a list, where:

* The indices are the time in minutes past trade opening time, which was 9:30am local time.
* The values are the price in dollars of Apple stock at that time.
So if the stock cost $500 at 10:30am, `stock_prices_yesterday[60] = 500`.

Write an efficient function that takes stock_prices_yesterday and returns **the best profit I could have made from 1 purchase and 1 sale of 1 Apple stock yesterday.**

For example:

[code language="python"]

	stock_prices_yesterday = [10, 7, 5, 8, 11, 9]

    get_max_profit(stock_prices_yesterday)
    # Returns 6 (buying for $5 and selling for $11)
	
[/code]

No "shorting"—you must buy before you sell. You may not buy and sell in the same time step (at least 1 minute must pass).


### Solution in O(n) time

[code language="python"]

    import sys as sys

    def get_max_profit(arrayOfPrices):
        maxProfit = -sys.maxsize - 1

        currentMin = arrayOfPrices[0]
        length = len(arrayOfPrices)
        for index in range(1, length):
            value = arrayOfPrices[index]

            difference = value - currentMin
            maxProfit = max(maxProfit, difference)
            currentMin = min(currentMin, value)

        return maxProfit
	
[/code]

### Lets make sure it works!


[code language="python"]

    import unittest
    import main as main

    class OurTests(unittest.TestCase):

        def test_arrayWithMin_BeforeMax(self):
            stock_prices_yesterday = [10, 7, 5, 8, 11, 9]
            result = main.get_max_profit(stock_prices_yesterday)
            assert result == 6

        def test_arrayWithMin_AfterMax(self):
            stock_prices_yesterday = [10, 12, 5, 8, 11, 9]
            result = main.get_max_profit(stock_prices_yesterday)
            assert result == 6

        def test_WithoutUsing_MaxValue(self):
            stock_prices_yesterday = [11, 22, 15, 8, 20, 3]
            result = main.get_max_profit(stock_prices_yesterday)
            assert result == 12

        def test_WithoutUsing_MinValue(self):
            stock_prices_yesterday = [10, 30, 5, 8, 20, 3]
            result = main.get_max_profit(stock_prices_yesterday)
            assert result == 20

        def test_SamePrice(self):
            stock_prices_yesterday = [10, 10, 10, 10]
            result = main.get_max_profit(stock_prices_yesterday)
            assert result == 0

        def test_DecendingPrice(self):
            stock_prices_yesterday = [10, 8, 7, 3]
            result = main.get_max_profit(stock_prices_yesterday)
            assert result == -1

    if __name__ == '__main__':
        unittest.main()
	
[/code]

