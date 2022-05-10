# Auto-Harmonic-V2
// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © HeWhoMustNotBeNamed

//   __    __            __       __  __                  __       __                        __      __    __              __      _______             __    __                                          __ 
//  /  |  /  |          /  |  _  /  |/  |                /  \     /  |                      /  |    /  \  /  |            /  |    /       \           /  \  /  |                                        /  |
//  $$ |  $$ |  ______  $$ | / \ $$ |$$ |____    ______  $$  \   /$$ | __    __   _______  _$$ |_   $$  \ $$ |  ______   _$$ |_   $$$$$$$  |  ______  $$  \ $$ |  ______   _____  ____    ______    ____$$ |
//  $$ |__$$ | /      \ $$ |/$  \$$ |$$      \  /      \ $$$  \ /$$$ |/  |  /  | /       |/ $$   |  $$$  \$$ | /      \ / $$   |  $$ |__$$ | /      \ $$$  \$$ | /      \ /     \/    \  /      \  /    $$ |
//  $$    $$ |/$$$$$$  |$$ /$$$  $$ |$$$$$$$  |/$$$$$$  |$$$$  /$$$$ |$$ |  $$ |/$$$$$$$/ $$$$$$/   $$$$  $$ |/$$$$$$  |$$$$$$/   $$    $$< /$$$$$$  |$$$$  $$ | $$$$$$  |$$$$$$ $$$$  |/$$$$$$  |/$$$$$$$ |
//  $$$$$$$$ |$$    $$ |$$ $$/$$ $$ |$$ |  $$ |$$ |  $$ |$$ $$ $$/$$ |$$ |  $$ |$$      \   $$ | __ $$ $$ $$ |$$ |  $$ |  $$ | __ $$$$$$$  |$$    $$ |$$ $$ $$ | /    $$ |$$ | $$ | $$ |$$    $$ |$$ |  $$ |
//  $$ |  $$ |$$$$$$$$/ $$$$/  $$$$ |$$ |  $$ |$$ \__$$ |$$ |$$$/ $$ |$$ \__$$ | $$$$$$  |  $$ |/  |$$ |$$$$ |$$ \__$$ |  $$ |/  |$$ |__$$ |$$$$$$$$/ $$ |$$$$ |/$$$$$$$ |$$ | $$ | $$ |$$$$$$$$/ $$ \__$$ |
//  $$ |  $$ |$$       |$$$/    $$$ |$$ |  $$ |$$    $$/ $$ | $/  $$ |$$    $$/ /     $$/   $$  $$/ $$ | $$$ |$$    $$/   $$  $$/ $$    $$/ $$       |$$ | $$$ |$$    $$ |$$ | $$ | $$ |$$       |$$    $$ |
//  $$/   $$/  $$$$$$$/ $$/      $$/ $$/   $$/  $$$$$$/  $$/      $$/  $$$$$$/  $$$$$$$/     $$$$/  $$/   $$/  $$$$$$/     $$$$/  $$$$$$$/   $$$$$$$/ $$/   $$/  $$$$$$$/ $$/  $$/  $$/  $$$$$$$/  $$$$$$$/ 
//                                                                                                                                                                                                          
//                                                                                                                                                                                                          
//
//@version=5
indicator('Auto Harmonic Patterns - Open Source', shorttitle='Auto Patterns', overlay=true, max_bars_back=1000, max_lines_count=500, max_labels_count=500)
Length = input.int(10, minval=0, step=5)
DeviationThreshold = 0
errorPercent = input.int(10, minval=5, step=5, maxval=20)
showPivots = input(false)
showRatios = input(false)
showZigZag = input(false)
bullishColor = input(color.green)
bullTrapColor = input(color.orange)
bearishColor = input(color.red)
bearTrapColor = input(color.lime)
MaxRiskPerReward = input.int(40, title='Max Risk Per Reward (Double Top/Bottom)', step=10, minval=0)

abcdClassic = input(true)
abEQcd = input(true)
abcdExt = input(true)
doubleTopBottom = input(true)
gartley = input(true)
crab = input(true)
deepCrab = input(true)
bat = input(true)
butterfly = input(true)
shark = input(true)
cypher = input(true)
threeDrives = input(true)
fiveZero = input(true)

var abcdlines = array.new_line(3)
var abcdtype = array.new_int(2, 1)

var wmlines = array.new_line(4)
var wmtype = array.new_int(2, 1)
var wmLabels = array.new_bool(9, false)

var patterncount = array.new_int(26, 0)


var zigzaglines = array.new_line(0)
var zigzaglabels = array.new_label(0)
var zigzagdir = array.new_int(0)
var zigzagratios = array.new_float(0)

int max_array_size = 10
transparent = color.new(#FFFFFF, 100)
err_min = (100 - errorPercent) / 100
err_max = (100 + errorPercent) / 100
pivots(length) =>
    float ph = ta.highestbars(high, length) == 0 ? high : na
    float pl = ta.lowestbars(low, length) == 0 ? low : na
    dir = 0
    iff_1 = pl and na(ph) ? -1 : dir[1]
    dir := ph and na(pl) ? 1 : iff_1
    [dir, ph, pl]

get_edir(dir, y2) =>
    eDir = dir
    if array.size(zigzaglines) > 0
        lastLine = array.get(zigzaglines, 0)
        lastPivot = line.get_y1(lastLine)
        eDir := (dir * y2 > dir * lastPivot ? 2 : 1) * dir
        eDir
    lineColor = eDir == 2 ? bullishColor : eDir == 1 ? bullTrapColor : eDir == -1 ? bearTrapColor : bearishColor
    [eDir, lineColor]

add_to_zigzaglines(x1, y1, x2, y2, dir) =>
    [eDir, lineColor] = get_edir(dir, y2)
    color = showZigZag ? lineColor : color.new(#FFFFFF, 100)
    zline = line.new(x1=x1, y1=y1, x2=x2, y2=y2, color=color, width=2, style=line.style_solid)
    array.unshift(zigzaglines, zline)

add_to_zigzaglabels(x1, x2, y1, y2, dir) =>
    [eDir, lineColor] = get_edir(dir, y2)
    pivotLabel = eDir == 2 ? 'HH' : eDir == 1 ? 'LH' : eDir == -1 ? 'HL' : 'LL'
    lastLineLen = 0.0
    currentLineLen = math.abs(y2 - y1)
    if array.size(zigzaglines) > 0
        lastLine = array.get(zigzaglines, 0)
        lastLineLen := math.abs(line.get_y2(lastLine) - line.get_y1(lastLine))
        lastLineLen

    ratio = math.round(lastLineLen != 0 ? currentLineLen / lastLineLen : 0, 3)
    labelText = (showPivots ? pivotLabel : '') + (showPivots and showRatios ? ' - ' : '') + (showRatios ? str.tostring(ratio) : '')
    yloc = dir > 0 ? yloc.abovebar : yloc.belowbar
    labelStyle = dir > 0 ? label.style_label_down : label.style_label_up
    labelSize = showRatios and showPivots ? size.normal : size.small
    zlabel = label.new(x=x2, y=y2, text=labelText, xloc=xloc.bar_index, yloc=yloc, color=lineColor, size=labelSize, style=labelStyle)

    array.unshift(zigzaglabels, zlabel)
    array.unshift(zigzagdir, eDir)
    array.unshift(zigzagratios, ratio)
    if not showRatios and not showPivots
        label.delete(zlabel)

add_to_zigzag(dir, dirchanged, ph, pl, index) =>
    value = dir == 1 ? ph : pl

    y1 = dir == 1 ? ta.lowest(Length) : ta.highest(Length)
    x1 = bar_index + (dir == 1 ? ta.lowestbars(Length) : ta.highestbars(Length))
    x2 = index
    y2 = value
    skip = false
    if array.size(zigzaglines) > 0
        if not dirchanged
            lastline = array.get(zigzaglines, 0)
            lasty2 = line.get_y2(lastline)
            if lasty2 * dir > y2 * dir
                skip := true
                skip
            else
                line.delete(array.shift(zigzaglines))
                label.delete(array.shift(zigzaglabels))
                array.shift(zigzagdir)
                array.shift(zigzagratios)
                skip := false
                skip

        if array.size(zigzaglines) > 0
            lastLine = array.get(zigzaglines, 0)
            x1 := line.get_x2(lastLine)
            y1 := line.get_y2(lastLine)
            y1

    outsideDeviationThreshold = math.abs(y1 - y2) * 100 / y1 > DeviationThreshold
    if outsideDeviationThreshold and not skip
        add_to_zigzaglabels(x1, x2, y1, y2, dir)
        add_to_zigzaglines(x1, y1, x2, y2, dir)
    if array.size(zigzaglines) > max_array_size
        array.pop(zigzaglines)
        array.pop(zigzaglabels)
        array.pop(zigzagdir)
        array.pop(zigzagratios)

zigzag(length, DeviationThreshold) =>
    [dir, ph, pl] = pivots(length)
    dirchanged = ta.change(dir)
    if ph or pl
        add_to_zigzag(dir, dirchanged, ph, pl, bar_index)

calculate_abcd() =>
    abcd = false

    if array.size(zigzagratios) >= 3 and array.size(zigzaglines) >= 4
        abcRatio = array.get(zigzagratios, 2)
        bcdRatio = array.get(zigzagratios, 1)

        ab = array.get(zigzaglines, 3)
        bc = array.get(zigzaglines, 2)
        cd = array.get(zigzaglines, 1)

        ab_time = math.abs(line.get_x1(ab) - line.get_x2(ab))
        ab_price = math.abs(line.get_y1(ab) - line.get_y2(ab))

        cd_time = math.abs(line.get_x1(cd) - line.get_x2(cd))
        cd_price = math.abs(line.get_y1(cd) - line.get_y2(cd))

        a = line.get_y1(ab)
        b = line.get_y2(ab)

        c = line.get_y1(cd)
        d = line.get_y2(cd)

        abcdDirection = a < b and a < c and c < b and c < d and a < d and b < d or a > b and a > c and c > b and c > d and a > d and b > d
        dir = a < b and a < c and c < b and c < d and a < d and b < d ? 1 : a > b and a > c and c > b and c > d and a > d and b > d ? -1 : 0
        time_ratio = cd_time / ab_time
        price_ratio = cd_price / ab_price

        // if (ab == ab[1] and bc == bc[1] and cd == cd[1])
        //     abcd := false
        if abcdClassic and abcRatio >= 0.618 * err_min and abcRatio <= 0.786 * err_max and bcdRatio >= 1.272 * err_min and bcdRatio <= 1.618 * err_max and abcdDirection
            abcd := true
            array.set(abcdtype, 0, 1)
        if abEQcd and time_ratio >= err_min and time_ratio <= err_max and price_ratio >= err_min and price_ratio <= err_max and abcdDirection
            abcd := true
            array.set(abcdtype, 0, 2)
        if abcdExt and price_ratio >= 1.272 * err_min and price_ratio <= 1.618 * err_max and abcRatio >= 0.618 * err_min and abcRatio <= 0.786 * err_max and abcdDirection
            abcd := true
            array.set(abcdtype, 0, 3)
        if abcd
            array.set(abcdlines, 0, ab)
            array.set(abcdlines, 1, bc)
            array.set(abcdlines, 2, cd)
            array.set(abcdtype, 1, dir)
    abcd

draw_abcd() =>
    abcd = calculate_abcd()
    if array.size(abcdlines) > 2 and array.size(zigzaglines) >= 4
        ab = array.get(abcdlines, 0)
        bc = array.get(abcdlines, 1)
        cd = array.get(abcdlines, 2)

        abcd_type = array.get(abcdtype, 0)
        dir = array.get(abcdtype, 1)

        labelColor = dir > 0 ? bearishColor : bullishColor
        labelStyle = dir > 0 ? label.style_label_down : label.style_label_up
        yloc = dir > 0 ? yloc.abovebar : yloc.belowbar

        labelText = abcd_type == 1 ? 'ABCD' : abcd_type == 2 ? 'AB=CD' : abcd_type == 3 ? 'ABCD Ext' : ''
        ab_zg = array.get(zigzaglines, 3)
        bc_zg = array.get(zigzaglines, 2)
        cd_zg = array.get(zigzaglines, 1)
        count_index = abcd_type * 2 - 2 + (dir > 0 ? 1 : 0)
        abcdLabel = label.new(x=line.get_x2(cd), y=line.get_y2(cd), text=labelText, xloc=xloc.bar_index, yloc=yloc, color=labelColor, size=size.normal, style=labelStyle)
        array.set(patterncount, count_index, array.get(patterncount, count_index) + 1)

        line.set_color(ab, labelColor)
        line.set_color(bc, labelColor)
        line.set_color(cd, labelColor)

        if abcd[1] and bc == bc_zg and ab == ab_zg
            label.delete(abcdLabel[1])
            array.set(patterncount, count_index, array.get(patterncount, count_index) - 1)
            line.set_color(ab, transparent)
            line.set_color(bc, transparent)
            line.set_color(cd, transparent)

        if not abcd or label.get_x(abcdLabel) == label.get_x(abcdLabel[1]) and label.get_y(abcdLabel) == label.get_y(abcdLabel[1])
            label.delete(abcdLabel)
            array.set(patterncount, count_index, array.get(patterncount, count_index) - 1)
        else
            line.set_color(ab, labelColor)
            line.set_color(bc, labelColor)
            line.set_color(cd, labelColor)

calculate_double_pattern() =>
    doubleTop = false
    doubleBottom = false
    if array.size(zigzagdir) >= 4 and doubleTopBottom
        value = line.get_y2(array.get(zigzaglines, 1))
        highLow = array.get(zigzagdir, 1)

        lvalue = line.get_y2(array.get(zigzaglines, 2))
        lhighLow = array.get(zigzagdir, 2)

        llvalue = line.get_y2(array.get(zigzaglines, 3))
        llhighLow = array.get(zigzagdir, 3)

        risk = math.abs(value - llvalue)
        reward = math.abs(value - lvalue)
        riskPerReward = risk * 100 / (risk + reward)

        if highLow == 1 and llhighLow == 2 and lhighLow == -1 and riskPerReward < MaxRiskPerReward
            doubleTop := true
            doubleTop
        if highLow == -1 and llhighLow == -2 and lhighLow == 1 and riskPerReward < MaxRiskPerReward
            doubleBottom := true
            doubleBottom

    [doubleTop, doubleBottom]


draw_double_pattern(doubleTop, doubleBottom) =>
    if array.size(zigzagdir) >= 4
        line1 = array.get(zigzaglines, 1)
        line2 = array.get(zigzaglines, 2)

        x1 = line.get_x1(line2)
        y1 = line.get_y1(line2)
        x2 = line.get_x2(line1)
        y2 = line.get_y2(line1)

        midline = line.get_y1(line1)
        midlineIndex = line.get_x1(line1)
        risk = math.abs(y2 - y1)
        reward = math.abs(y2 - midline)
        riskPerReward = math.round(risk * 100 / (risk + reward), 2)

        base = line.new(x1=x1, y1=y1, x2=x2, y2=y2, color=doubleTop ? bearishColor : bullishColor, width=2, style=line.style_solid)

        count_index = doubleTop ? 7 : 6
        labelText = (doubleTop ? 'DT - ' : 'DB - ') + str.tostring(riskPerReward)
        array.set(patterncount, count_index, array.get(patterncount, count_index) + 1)
        baseLabel = label.new(x=x2, y=y2, text=labelText, yloc=doubleTop ? yloc.abovebar : yloc.belowbar, color=doubleTop ? bearishColor : bullishColor, style=doubleTop ? label.style_label_down : label.style_label_up, textcolor=color.black, size=size.normal)
        if line.get_x1(base) == line.get_x1(base[1])
            line.delete(base[1])
            label.delete(baseLabel[1])
            array.set(patterncount, count_index, array.get(patterncount, count_index) - 1)

        if not(doubleTop or doubleBottom)
            line.delete(base)
            label.delete(baseLabel)
            array.set(patterncount, count_index, array.get(patterncount, count_index) - 1)

calculate_wm_patterns() =>
    wm_pattern = false
    if array.size(zigzagdir) >= 5
        yxaRatio = array.get(zigzagratios, 4)
        xabRatio = array.get(zigzagratios, 3)
        abcRatio = array.get(zigzagratios, 2)
        bcdRatio = array.get(zigzagratios, 1)

        xa = array.get(zigzaglines, 4)
        ab = array.get(zigzaglines, 3)
        bc = array.get(zigzaglines, 2)
        cd = array.get(zigzaglines, 1)

        x = line.get_y1(xa)
        a = line.get_y2(xa)
        b = line.get_y2(ab)
        c = line.get_y1(cd)
        d = line.get_y2(cd)

        xadRatio = math.round(math.abs(a - d) / math.abs(x - a), 3)
        dir = a > d ? 1 : -1

        maxP1 = math.max(x, a)
        maxP2 = math.max(c, d)
        minP1 = math.min(x, a)
        minP2 = math.min(c, d)

        highPoint = math.min(maxP1, maxP2)
        lowPoint = math.max(minP1, minP2)

        if b < highPoint and b > lowPoint
            //gartley
            if gartley and xabRatio >= 0.618 * err_min and xabRatio <= 0.618 * err_max and abcRatio >= 0.382 * err_min and abcRatio <= 0.886 * err_max and (bcdRatio >= 1.272 * err_min and bcdRatio <= 1.618 * err_max or xadRatio >= 0.786 * err_min and xadRatio <= 0.786 * err_max)
                wm_pattern := true
                array.set(wmtype, 1, 0)
                array.set(wmLabels, 0, true)
            else
                array.set(wmLabels, 0, false)
            //Crab
            if crab and xabRatio >= 0.382 * err_min and xabRatio <= 0.618 * err_max and abcRatio >= 0.382 * err_min and abcRatio <= 0.886 * err_max and (bcdRatio >= 2.24 * err_min and bcdRatio <= 3.618 * err_max or xadRatio >= 1.618 * err_min and xadRatio <= 1.618 * err_max)
                wm_pattern := true
                array.set(wmtype, 1, 1)
                array.set(wmLabels, 1, true)
            else
                array.set(wmLabels, 1, false)
            //Deep Crab
            if deepCrab and xabRatio >= 0.886 * err_min and xabRatio <= 0.886 * err_max and abcRatio >= 0.382 * err_min and abcRatio <= 0.886 * err_max and (bcdRatio >= 2.00 * err_min and bcdRatio <= 3.618 * err_max or xadRatio >= 1.618 * err_min and xadRatio <= 1.618 * err_max)
                wm_pattern := true
                array.set(wmtype, 1, 2)
                array.set(wmLabels, 2, true)
            else
                array.set(wmLabels, 2, false)
            //Bat
            if bat and xabRatio >= 0.382 * err_min and xabRatio <= 0.50 * err_max and abcRatio >= 0.382 * err_min and abcRatio <= 0.886 * err_max and (bcdRatio >= 1.618 * err_min and bcdRatio <= 2.618 * err_max or xadRatio >= 0.886 * err_min and xadRatio <= 0.886 * err_max)
                wm_pattern := true
                array.set(wmtype, 1, 3)
                array.set(wmLabels, 3, true)
            else
                array.set(wmLabels, 3, false)
            //Butterfly
            if butterfly and xabRatio >= 0.786 * err_min and xabRatio <= 0.786 * err_max and abcRatio >= 0.382 * err_min and abcRatio <= 0.886 * err_max and (bcdRatio >= 1.618 * err_min and bcdRatio <= 2.618 * err_max or xadRatio >= 1.272 * err_min and xadRatio <= 1.618 * err_max)
                wm_pattern := true
                array.set(wmtype, 1, 4)
                array.set(wmLabels, 4, true)
            else
                array.set(wmLabels, 4, false)
            //Shark
            if shark and abcRatio >= 1.13 * err_min and abcRatio <= 1.618 * err_max and bcdRatio >= 1.618 * err_min and bcdRatio <= 2.24 * err_max and xadRatio >= 0.886 * err_min and xadRatio <= 1.13 * err_max
                wm_pattern := true
                array.set(wmtype, 1, 5)
                array.set(wmLabels, 5, true)
            else
                array.set(wmLabels, 5, false)
            //Cypher
            if cypher and xabRatio >= 0.382 * err_min and xabRatio <= 0.618 * err_max and abcRatio >= 1.13 * err_min and abcRatio <= 1.414 * err_max and (bcdRatio >= 1.272 * err_min and bcdRatio <= 2.00 * err_max or xadRatio >= 0.786 * err_min and xadRatio <= 0.786 * err_max)
                wm_pattern := true
                array.set(wmtype, 1, 6)
                array.set(wmLabels, 6, true)
            else
                array.set(wmLabels, 6, false)
        //3 drive
        if threeDrives and yxaRatio >= 0.618 * err_min and yxaRatio <= 0.618 * err_max and xabRatio >= 1.27 * err_min and xabRatio <= 1.618 * err_max and abcRatio >= 0.618 * err_min and abcRatio <= 0.618 * err_max and bcdRatio >= 1.27 * err_min and bcdRatio <= 1.618 * err_max
            wm_pattern := true
            array.set(wmtype, 1, 7)
            array.set(wmLabels, 7, true)
        else
            array.set(wmLabels, 7, false)
        //5-0
        if fiveZero and xabRatio >= 1.13 * err_min and xabRatio <= 1.618 * err_max and abcRatio >= 1.618 * err_min and abcRatio <= 2.24 * err_max and bcdRatio >= 0.5 * err_min and bcdRatio <= 0.5 * err_max
            wm_pattern := true
            array.set(wmtype, 1, 8)
            array.set(wmLabels, 8, true)
        else
            array.set(wmLabels, 8, false)
        if wm_pattern
            array.set(wmlines, 0, xa)
            array.set(wmlines, 1, ab)
            array.set(wmlines, 2, bc)
            array.set(wmlines, 3, cd)
            array.set(wmtype, 0, dir)
    wm_pattern

draw_wm_patterns() =>
    wm_pattern = calculate_wm_patterns()
    if array.size(wmlines) >= 4 and array.size(zigzaglines) >= 5
        xa = array.get(wmlines, 0)
        ab = array.get(wmlines, 1)
        bc = array.get(wmlines, 2)
        cd = array.get(wmlines, 3)

        dir = array.get(wmtype, 0)
        type = array.get(wmtype, 1)
        trendColor = dir > 0 ? bullishColor : bearishColor
        x = line.get_y1(xa)
        xbar = line.get_x1(xa)

        a = line.get_y2(xa)
        abar = line.get_x2(xa)

        b = line.get_y2(ab)
        bbar = line.get_x2(ab)

        c = line.get_y2(bc)
        cbar = line.get_x2(bc)

        d = line.get_y2(cd)
        dbar = line.get_x2(cd)

        line.set_color(xa, trendColor)
        line.set_color(ab, trendColor)
        line.set_color(bc, trendColor)
        line.set_color(cd, trendColor)
        ac = line.new(x1=abar, y1=a, x2=cbar, y2=c, color=trendColor, width=2, style=line.style_solid)
        xb = line.new(x1=xbar, y1=x, x2=bbar, y2=b, color=trendColor, width=2, style=line.style_solid)
        xd = line.new(x1=xbar, y1=x, x2=dbar, y2=d, color=trendColor, width=2, style=line.style_solid)
        bd = line.new(x1=bbar, y1=b, x2=dbar, y2=d, color=trendColor, width=2, style=line.style_solid)

        isGartley = array.get(wmLabels, 0)
        isCrab = array.get(wmLabels, 1)
        isDeepCrab = array.get(wmLabels, 2)
        isBat = array.get(wmLabels, 3)
        isButterfly = array.get(wmLabels, 4)
        isShark = array.get(wmLabels, 5)
        isCypher = array.get(wmLabels, 6)
        is3Drives = array.get(wmLabels, 7)
        isFiveZero = array.get(wmLabels, 8)

        labelText = isGartley ? 'Gartley' : ''
        labelText += (isCrab ? (labelText == '' ? '' : '\n') + 'Crab' : '')
        labelText += (isDeepCrab ? (labelText == '' ? '' : '\n') + 'Deep Crab' : '')
        labelText += (isBat ? (labelText == '' ? '' : '\n') + 'Bat' : '')
        labelText += (isButterfly ? (labelText == '' ? '' : '\n') + 'Butterfly' : '')
        labelText += (isShark ? (labelText == '' ? '' : '\n') + 'Shark' : '')
        labelText += (isCypher ? (labelText == '' ? '' : '\n') + 'Cypher' : '')
        labelText += (is3Drives ? (labelText == '' ? '' : '\n') + '3 Drive' : '')
        labelText += (isFiveZero ? (labelText == '' ? '' : '\n') + '5-0' : '')

        baseLabel = label.new(x=bbar, y=b, text=labelText, yloc=dir < 1 ? yloc.abovebar : yloc.belowbar, color=trendColor, style=dir < 1 ? label.style_label_down : label.style_label_up, textcolor=color.black, size=size.normal)

        xa_zg = array.get(zigzaglines, 4)
        ab_zg = array.get(zigzaglines, 3)
        bc_zg = array.get(zigzaglines, 2)
        cd_zg = array.get(zigzaglines, 1)

        for i = 0 to 8 by 1
            count_index = i * 2 + 8 + (dir > 0 ? 0 : 1)
            if array.get(wmLabels, i) and xa != xa[1] and ab != ab[1] and bc != bc[1]
                array.set(patterncount, count_index, array.get(patterncount, count_index) + 1)
            if array.get(wmLabels, i)[1] and xa == xa_zg and ab == ab_zg and bc == bc_zg
                array.set(patterncount, count_index, array.get(patterncount, count_index) - 1)

        if wm_pattern[1] and not wm_pattern and xa == xa_zg and ab == ab_zg and bc == bc_zg
            line.delete(ac[1])
            line.delete(xb[1])
            line.delete(xd[1])
            line.delete(bd[1])
            line.set_color(xa[1], transparent)
            line.set_color(ab[1], transparent)
            line.set_color(bc[1], transparent)
            line.set_color(cd[1], transparent)
            label.delete(baseLabel[1])
            // array.set(patterncount, count_index, array.get(patterncount, count_index) - 1)

        if not wm_pattern or label.get_x(baseLabel) == label.get_x(baseLabel[1]) and label.get_y(baseLabel) == label.get_y(baseLabel[1])
            line.delete(ac)
            line.delete(xb)
            line.delete(xd)
            line.delete(bd)
            label.delete(baseLabel)

zigzag(Length, DeviationThreshold)
draw_abcd()

[doubleTop, doubleBottom] = calculate_double_pattern()
draw_double_pattern(doubleTop, doubleBottom)

draw_wm_patterns()


