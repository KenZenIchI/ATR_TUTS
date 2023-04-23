
//@version=4
strategy("BitBell 1M")
//{
// # Author: BitBell
//}







//Format for a 10
len4 = input(32, minval=1, title="Length")
src4 = input(close, title="Source")
out4 = ema(src4, len4)

//End of format

//Format for a 50
len5 = input(32, minval=1, title="Length")
src5 = input(low, title="Source")
out5 = ema(src5, len5)

//End of format

//Format for a 200
len6 = input(32, minval=1, title="Length")
src6 = input(high, title="Source")
out6 = ema(src6, len6)

//End of format

// plot(out6, "Extremum",color = color.fuchsia)
// plot(out5, "Extremum",color = color.fuchsia)
// plot(out4, "Extremum",color = color.fuchsia)









trailType        = input("modified", "Trailtype", options = ["modified", "unmodified"])
ATRPeriod        = input(200, "ATR Period")
ATRFactor        = input(8, "ATR Factor")
show_fib_entries = input(true, "Show Fib Entries?")

norm_o = security(tickerid(syminfo.prefix,syminfo.ticker), timeframe.period, open)
norm_h = security(tickerid(syminfo.prefix,syminfo.ticker), timeframe.period, high)
norm_l = security(tickerid(syminfo.prefix,syminfo.ticker), timeframe.period, low)
norm_c = security(tickerid(syminfo.prefix,syminfo.ticker), timeframe.period, close)
//}

//////// FUNCTIONS //////////////
//{
// Wilders ma //
Wild_ma(_src, _malength) =>
    _wild  = 0.0
    _wild := nz(_wild[1]) + (_src - nz(_wild[1])) / _malength

/////////// TRUE RANGE CALCULATIONS /////////////////
HiLo = min(norm_h - norm_l, 1.5 * nz(sma((norm_h - norm_l), ATRPeriod)))

HRef = norm_l<= norm_h[1] ?
 norm_h - norm_c[1] :
 (norm_h - norm_c[1]) - 0.5 * (norm_l- norm_h[1])

LRef = norm_h >= norm_l[1] ?
 norm_c[1] - norm_l:
 (norm_c[1] - norm_l) - 0.5 * (norm_l[1] - norm_h)

trueRange =
 trailType == "modified" ? max(HiLo, HRef, LRef) :
 max(norm_h - norm_l, abs(norm_h - norm_c[1]), abs(norm_l - norm_c[1]))
//}


/////////// TRADE LOGIC ////////////////////////
//{
loss = ATRFactor * Wild_ma(trueRange, ATRPeriod)

Up = norm_c - loss
Dn = norm_c + loss

TrendUp   = Up
TrendDown = Dn
Trend     = 1

TrendUp   := norm_c[1] > TrendUp[1]   ? max(Up, TrendUp[1])   : Up
TrendDown := norm_c[1] < TrendDown[1] ? min(Dn, TrendDown[1]) : Dn

Trend := norm_c > TrendDown[1] ? 1 : norm_c < TrendUp[1]? -1 : nz(Trend[1],1)
trail = Trend == 1? TrendUp : TrendDown

ex = 0.0
ex :=
 crossover(Trend, 0)  ? norm_h :
 crossunder(Trend, 0) ? norm_l :
 Trend == 1  ? max(ex[1], norm_h) :
 Trend == -1 ? min(ex[1], norm_l) : ex[1]
//}

// //////// PLOT TP and SL /////////////
//{
// plot(trail, "Trailingstop", style = plot.style_line, color = Trend == 1 ? color.green : Trend == -1 ? color.red : na)
// plot(ex, "Extremum", style = plot.style_circles, color = Trend == 1? color.lime : Trend == -1? color.fuchsia : na)
//}

////// FIBONACCI LEVELS ///////////
//{
state = Trend == 1 ? "long" : "short"

fib1Level = 61.8
fib2Level = 78.6
fib3Level = 88.6

f1 = ex + (trail - ex) * fib1Level / 100
f2 = ex + (trail - ex) * fib2Level / 100
f3 = ex + (trail - ex) * fib3Level / 100
l100 = trail + 0

// Fib1 = plot(f1,  "Fib 1", style = plot.style_line, color = color.black)
// Fib2 = plot(f2,  "Fib 2", style = plot.style_line, color = color.black)
// Fib3 = plot(f3,  "Fib 3", style = plot.style_line, color = color.black)
// L100 = plot(l100, "l100", style = plot.style_line, color = color.black)

// fill(Fib1, Fib2, color = state == "long"? color.green : state == "short"? color.red : na)
// fill(Fib2, Fib3, color = state == "long"? color.new(color.green, 70) : state == "short"? color.new(color.red, 70) : na)
// fill(Fib3, L100, color = state == "long"? color.new(color.green, 60) : state == "short"? color.new(color.red, 60) : na)

l1 = state[1] == "long" and crossunder(norm_c, f1[1])
l2 = state[1] == "long" and crossunder(norm_c, f2[1])
l3 = state[1] == "long" and crossunder(norm_c, f3[1])
s1 = state[1] == "short" and crossover(norm_c, f1[1])
s2 = state[1] == "short" and crossover(norm_c, f2[1])
s3 = state[1] == "short" and crossover(norm_c, f3[1])

atr = sma(trueRange, 14)

var LongEntry = 0.0
var ShortEntry = 0.0


Risk_Percentage = abs((f3-close)*100/close)
Short_Risk_Percentage = abs((f3-close)*100/close)





if (ohlc4 < out4[0] and ohlc4 > out5[0]) and Trend == 1 and ohlc4 >= f3
    Quisk = ((5.0 / 100) * (strategy.initial_capital / close / Risk_Percentage)) * 30
    strategy.entry('Long',strategy.long, qty = Quisk)

if ohlc4 >= (1.005 * strategy.position_avg_price)
    strategy.close('Long')
if ohlc4 <= (0.995 * strategy.position_avg_price) and strategy.position_size > 0
    strategy.close('Long')

if (ohlc4 < out4[0] and ohlc4 > out5[0]) and Trend == -1 and ohlc4 < f3
    SQuisk = ((5.0 / 100) * ((strategy.initial_capital / close) / (Short_Risk_Percentage)) * 30)
    strategy.entry('Short',strategy.short, qty = SQuisk)

if ohlc4 <= (0.995 * strategy.position_avg_price) and strategy.position_size < 0
    strategy.close('Short')

if ohlc4 >= (1.005 * strategy.position_avg_price) and strategy.position_size < 0
    strategy.close('Short')


if Trend == 1
    strategy.close('Short')

if Trend == -1
    strategy.close('Long')










/////////// FIB PLOTS /////////////////.

// plotshape(show_fib_entries and l1 ? low - atr : na, "LS1", style = shape.triangleup, location = location.belowbar, color = color.yellow, size = size.tiny)
// plotshape(show_fib_entries and l2 ? low - 1.5 * atr : na, "LS2", style = shape.triangleup, location = location.belowbar, color = color.yellow, size = size.tiny)
// plotshape(show_fib_entries and l3 ? low - 2 * atr : na, "LS3", style = shape.triangleup, location = location.belowbar, color = color.yellow, size = size.tiny)
// plotshape(show_fib_entries and s1 ? high + atr : na, "SS1", style = shape.triangledown, location = location.abovebar, color = color.purple, size = size.tiny)
// plotshape(show_fib_entries and s2 ? high + 1.5 * atr : na, "SS2", style = shape.triangledown, location = location.abovebar, color = color.purple, size = size.tiny)
// plotshape(show_fib_entries and s3 ? high + 2 * atr : na, "SS3", style = shape.triangledown, location = location.abovebar, color = color.purple, size = size.tiny)
//}


// //////////// FIB ALERTS /////////////////////
// //{
// alertcondition(l1, title = "cross over Fib1",  message = "Price crossed below Fib1 level in long trend")
// alertcondition(l2, title = "cross over Fib2",  message = "Price crossed below Fib2 level in long trend")
// alertcondition(l3, title = "cross over Fib3",  message = "Price crossed below Fib3 level in long trend")
// alertcondition(s1, title = "cross under Fib1", message = "Price crossed above Fib1 level in short trend")
// alertcondition(s2, title = "cross under Fib2", message = "Price crossed above Fib2 level in short trend")
// alertcondition(s3, title = "cross under Fib3", message = "Price crossed above Fib3 level in short trend")

// alertcondition(fixnan(f1)!=fixnan(f1[1]), title = "Stop Line Change", message = "Stop Line Change")
//}
