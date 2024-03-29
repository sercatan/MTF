// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © Duyck
//
//
// This script is not intended to be used as an indicator but gives you a workaround to solve the missing seconds MTF capabilities

// The "resolution" function in Pinescript doesn't allow for seconds values to be put in MTF
// So I wrote a little helper code with arrays to get MTF on seconds timeframes.

// If you want to add MTF in minutes, hours,... you can always add 'resolution = "" ' to the study line of the script

// As an example I plotted the sma from the MTF values on the chart, but you can add anything you want that you can calculate from the array values.


// Have fun with it !

// Gr, JD.


//@version=4
study("MTF seconds values - JD", overlay = true)

seconds_tf  = input( 30, title = "seconds timeframe, set this higher then the chart's timeframe !!", options = [5, 15, 30, 45])
avg_length  = input( 20, title = "average length", minval = 2)

////////////////////////////////////////////
// keep track of seconds timeframe values //
////////////////////////////////////////////
// one array is used per candle value
// {
var float[] open_MTF        = array.new_float(1, 0.0)
var float[] high_MTF        = array.new_float(1, 0.0)
var float[] low_MTF         = array.new_float(1, 0.0)
var float[] close_MTF       = array.new_float(1, 0.0)
var float[] volume_MTF      = array.new_float(1, 0.0)

new_bar                 = second % seconds_tf == 0

if array.size(open_MTF) > avg_length
    array.pop(open_MTF)
    array.pop(high_MTF)
    array.pop(low_MTF)
    array.pop(close_MTF)
    array.pop(volume_MTF)

if new_bar
    array.unshift(open_MTF,   open)
    array.unshift(high_MTF,   high)
    array.unshift(low_MTF,    low)
    array.unshift(close_MTF,  close)
    array.unshift(volume_MTF, volume)

if not new_bar
    array.set(high_MTF,   0, max(high, array.get(high_MTF,  0)))
    array.set(low_MTF,    0, min(low,  array.get(low_MTF,   0)))
    array.set(close_MTF,  0, close)
    array.set(volume_MTF, 0, volume + array.get(volume_MTF, 0))
//}


//////////////////////////////////////////////////////////////////////////////////
// retrieve previous and current MTF values to make writing easier for plotting //
//////////////////////////////////////////////////////////////////////////////////
//{
open_mtf    = array.get(open_MTF,   0),  open_mtf_prev   = array.get(open_MTF,   1)
high_mtf    = array.get(high_MTF,   0),  high_mtf_prev   = array.get(high_MTF,   1)
low_mtf     = array.get(low_MTF,    0),   low_mtf_prev   = array.get(low_MTF,    1)
close_mtf   = array.get(close_MTF,  0), close_mtf_prev   = array.get(close_MTF,  1)
vol_mtf     = array.get(volume_MTF, 0),   vol_mtf_prev   = array.get(volume_MTF, 1)

// example with simple moving average
// other array math functions are also possible
avg_mtf     = array.avg(close_MTF)
//}


///////////
// Plots //
///////////
//{
o = plot(open_mtf,                  title = "open",  style = plot.style_circles, color = color.green)
h = plot(high_mtf,                  title = "high",  style = plot.style_circles, color = color.yellow)
l = plot(low_mtf,                   title = "low",   style = plot.style_circles, color = color.fuchsia)
c = plot(close_mtf,                 title = "close", style = plot.style_circles, color = color.red)
a = plot(new_bar ? avg_mtf : na ,   title = "averag",                            color = color.yellow)
v = plot(vol_mtf    ,               title = "volume",                            color = na)

fill(o,c, color = open_mtf < close_mtf ? color.green : color.red)

// show beginning of new MTF bar
bgcolor(new_bar ? color.gray : na)
//}
