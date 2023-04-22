//@version=4

strategy(title="NEO#DX_Bot", overlay = true)



src = close 
 
keyvalue = input(8, title = "Key Vaule. 'This changes the sensitivity'", step = .5)
atrperiod = input(10, title="ATR Period")
xATR = atr(atrperiod)
nLoss = keyvalue * xATR

xATRTrailingStop = 0.0
xATRTrailingStop := iff(src > nz(xATRTrailingStop[1], 0) and src[1] > nz(xATRTrailingStop[1], 0), max(nz(xATRTrailingStop[1]), src - nLoss),
   iff(src < nz(xATRTrailingStop[1], 0) and src[1] < nz(xATRTrailingStop[1], 0), min(nz(xATRTrailingStop[1]), src + nLoss), 
   iff(src > nz(xATRTrailingStop[1], 0), src - nLoss, src + nLoss)))
 
pos = 0   
pos :=	iff(src[1] < nz(xATRTrailingStop[1], 0) and src > nz(xATRTrailingStop[1], 0), 1,
   iff(src[1] > nz(xATRTrailingStop[1], 0) and src < nz(xATRTrailingStop[1], 0), -1, nz(pos[1], 0))) 
   
xcolor = pos == -1 ? color.red: pos == 1 ? color.green : color.blue



plot(xATRTrailingStop, color = xcolor, title = "Trailing Stop")
buy = crossover(src,xATRTrailingStop)
sell = crossunder(src,xATRTrailingStop)
barcolor = src > xATRTrailingStop 

Ema300_Input = input(300,title = 'EMA SuperSlow')
EMA150 = ema(close,Ema300_Input)
plot(EMA150, title="EMA150", color=#2f618d)


var RandeLong = 0.0
var RandeMartin_QTY_Long = 0.0
var RandeShort = 0.0
var RandeMartin_QTY_Short = 0.0

var RandeMartin_Long = 0.0
var RandeMartin_Short = 0.0

Risk_Percentage = abs((EMA150-close)*100/close)
riskoLong = Risk_Percentage * 0.995
riskoShort = Risk_Percentage * 1.005
if (buy)
    Quisk = ((5.0 / 100) * (strategy.initial_capital / close / riskoLong)) * 30
    strategy.entry("+", strategy.long, qty = Quisk * 0.5)
    RandeLong := 1.0
    RandeShort := 0.0
    RandeMartin_Long := 1.0
    RandeMartin_QTY_Long := 0.0

if RandeMartin_Long == 1.0 and ohlc4 <= (0.985 * strategy.position_avg_price) and strategy.position_size > 0.0

    strategy.entry("+", strategy.long, qty = RandeMartin_QTY_Long)
    RandeMartin_Long := 0.0




if (sell)
    Quisk = ((5.0 / 100) * (strategy.initial_capital / close / riskoShort)) * 30
    strategy.entry("-", strategy.short, qty = Quisk * 0.5)
    RandeLong := 0.0
    RandeShort := 1.0
    RandeMartin_Short := 1.0
    RandeMartin_QTY_Short := Quisk
    RandeMartin_QTY_Long := 0.0

if RandeMartin_Short == 1.0 and ohlc4 >= (1.015 * strategy.position_avg_price) and strategy.position_size < 0.0
    strategy.entry("+", strategy.long, qty = RandeMartin_QTY_Short)
    RandeMartin_Long := 0.0


if RandeLong == 1.0 and ohlc4 >= (1.005 * strategy.position_avg_price)
    strategy.close("+",qty_percent = 10)
    RandeLong := 2.0
if RandeShort == 1.0 and ohlc4 <= (0.995 * strategy.position_avg_price)
    strategy.close( "-",qty_percent = 10)
    RandeShort := 2.0

if RandeLong == 2.0 and ohlc4 >= (1.01 * strategy.position_avg_price)
    strategy.close("+",qty_percent = 50)
    RandeLong := 3.0
if RandeShort == 2.0 and ohlc4 <= (0.99 * strategy.position_avg_price)
    strategy.close( "-",qty_percent = 50)
    RandeShort := 3.0

if RandeLong == 3.0 and ohlc4 >= (1.015 * strategy.position_avg_price)
    strategy.close("+",qty_percent = 50)
    RandeLong := 4.0
if RandeShort == 3.0 and ohlc4 <= (0.985 * strategy.position_avg_price)
    strategy.close( "-",qty_percent = 50)
    RandeShort := 4.0


if RandeLong == 4.0 and ohlc4 >= (1.02 * strategy.position_avg_price)
    strategy.close("+",qty_percent = 90)
    RandeLong := 5.0
if RandeShort == 4.0 and ohlc4 <= (0.98 * strategy.position_avg_price)
    strategy.close( "-",qty_percent = 90)
    RandeShort := 5.0



barcolor(barcolor? color.green:color.red)

// alertcondition(buy, title='BOT Buy', message='BOT Buy')
// alertcondition(sell, title='BOT Sell', message='BOT Sell')
