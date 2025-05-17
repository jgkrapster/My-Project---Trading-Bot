**MetaTrader5 Gold Trading Bot**

**Project Overview**

This project implements an automated trading bot designed to trade Gold (XAUUSD) using the MetaTrader5 platform. The bot leverages the SuperTrend indicator, a popular technical analysis tool, to generate buy and sell signals. By continuously monitoring market conditions and executing trades based on predefined criteria, the bot aims to capitalize on price movements in the gold market without requiring constant manual intervention.

The trading strategy is built around the SuperTrend indicator with specifically calibrated parameters (period=7, multiplier=3) to identify potential trend changes in the gold market. When the closing price crosses above the SuperTrend line, a buy signal is generated; conversely, when it crosses below, a sell signal is triggered. The bot includes sophisticated position management features, including dynamic stop-loss adjustments to protect profits as trades move in favorable directions.
**Components and Functionality**

**Data Collection and Preparation**
The bot connects to MetaTrader5 to retrieve historical price data for gold (XAUUSD) using the copy_rates_range function. It processes this data into a pandas DataFrame to facilitate technical analysis. The code handles timezone conversions appropriately, ensuring that all data is synchronized to the correct market time (America/New York).
pythonend_datetime = target_timezone.localize(datetime.now())
data1 = mt.copy_rates_range('XAUUSD', mt.TIMEFRAME_M1, datetime(2022,1,1), end_datetime)
ohlc = pd.DataFrame(data1)
ohlc['time'] = pd.to_datetime(ohlc['time'], unit='s')

**Technical Analysis Implementation**
The bot utilizes the pandas_ta library to calculate the SuperTrend indicator values. This technical indicator helps identify the current market trend and potential reversal points. The SuperTrend calculation is performed with a period of 7 and multiplier of 3, parameters that were chosen after testing for optimal performance in the gold market.
pythonohlc['sup'] = ta.supertrend(high=ohlc['high'], low=ohlc['low'], close=ohlc['close'], period=7, multiplier=3)['SUPERT_7_3.0']
**Signal Generation Logic**
Signal generation is a critical component of the trading strategy. The bot analyzes price movements relative to the SuperTrend line and identifies crossover events that indicate potential trade opportunities:

**Buy Signal:** Generated when the closing price crosses above the SuperTrend line
**Sell Signal:** Generated when the closing price crosses below the SuperTrend line

pythonfor i in range(n, len(ohlc)):
    if ohlc['close'][i-1] <= ohlc['sup'][i-1] and ohlc['close'][i] > ohlc['sup'][i]:
        ohlc['Buy Signal'][i] = 1
    if ohlc['close'][i-1] >= ohlc['sup'][i-1] and ohlc['close'][i] < ohlc['sup'][i]:
        ohlc['Sell Signal'][i] = 1
**Order Management Functions**
The bot includes three essential order management functions:

**create_order():** Initiates new trading positions with specified parameters including stop-loss and take-profit levels
**close_order():** Closes existing positions when signals reverse or targets are hit
**modify_order():** Dynamically adjusts stop-loss levels to protect profits as trades move favorably

These functions interface with the MetaTrader5 API to execute actual trades in the market. The implementation follows best practices by specifying all necessary order parameters including magic number for trade identification and appropriate time and filling types.

**Position Management and Risk Control**
Risk management is implemented through carefully calculated stop-loss and take-profit levels:

Stop-loss is set at 0.06% of the entry price
Take-profit is set at 0.9% of the entry price
Dynamic trailing stop-loss adjusts as positions move favorably

This asymmetric risk-reward ratio (approximately 1:15) was a deliberate design choice to capitalize on trending moves in the gold market while minimizing losses on incorrect signals.
pythonsl_pct = 0.0006
tp_pct = 0.0090
buy_sl = buy_price * (1-sl_pct)
buy_tp = buy_price * (1+tp_pct)
**Main Trading Loop**
The core of the bot is an infinite loop that:

Checks existing positions and adjusts stop-losses if needed
Retrieves updated market data on a 5-minute timeframe
Calculates technical indicators and generates signals
Makes trading decisions based on current signals and position status
Executes appropriate trades through the MetaTrader5 API
Waits for 300 seconds (5 minutes) before repeating

The 5-minute interval was chosen as a balance between capturing significant market moves and avoiding excessive trading activity that could lead to higher transaction costs.
**Design Choices and Considerations**
**Timeframe Selection**
While the bot initially retrieves 1-minute historical data for analysis, the actual trading decisions are made on the 5-minute timeframe. This design choice was made to filter out market noise present in lower timeframes while still providing timely entry and exit signals. The 5-minute chart offers a good compromise between signal quality and trading frequency.
**SuperTrend Parameters**
After extensive testing, the SuperTrend parameters of period=7 and multiplier=3 were selected as optimal for gold trading. These settings provide a balance between sensitivity to market changes and resistance to false signals. Alternative parameters were tested but either generated too many false signals (with lower values) or missed important market moves (with higher values).
**Position Sizing**
The bot uses a fixed position size of 0.5 lots for all trades. While adaptive position sizing based on market volatility or account equity could be implemented in future versions, a fixed size was chosen for simplicity and consistent testing. Users should adjust this value according to their risk tolerance and account size.
**Signal Confirmation Strategy**
The bot includes logic to prevent excessive trading by tracking the last executed signal type. This prevents the system from repeatedly entering and exiting positions on noise or small price fluctuations around the SuperTrend line, which could lead to unnecessary transaction costs.
**Future Improvements**
Potential enhancements to consider for future versions:

Implementing adaptive position sizing based on market volatility
Adding multiple confirmatory indicators to reduce false signals
Incorporating machine learning for parameter optimization
Developing a more sophisticated trailing stop mechanism
Adding functionality to trade multiple instruments simultaneously

This trading bot represents a solid foundation for algorithmic gold trading that can be extended and customized for various market conditions and trading preferences.
