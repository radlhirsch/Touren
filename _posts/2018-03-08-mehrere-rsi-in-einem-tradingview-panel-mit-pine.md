---
layout: post
title:  "Mehrere RSI-Indikatoren in Tradingview in einem Panel bündeln"
date:   2018-03-08 12:00:00
excerpt: "Die Berechnung und Darstellung der Indikatoren `RSI` und `stochastischer RSI` ähneln sich, weswegen sie gute Kandidaten für das Zusammenfassen auf einem Tradingview-Panel sind."
image:
thumb: /assets/img/thumbs/combined-rsi.jpg
tags: [bitcoin, kryptowaehrungen, tradingview, pine, rsi]
categories: [posts, cryptocurrencies]
comments: true
lang: de
ref: post-multiple-rsi-in-a-tradingview-panel-with-pine
---

## Einleitung

Das Webportal [Tradingview](https://tradingview.com) ist ein sehr ausgereiftes Werkzeug, um sich zum Beispiel Verläufe von Aktien- oder Kryptowährungskursen anzeigen zu lassen und mit Hilfe verschiedenster darstellbarer Indikatoren Analysen zum zukünftigen Verlauf zu treffen.
Die Menge darstellbarer Indikatoren ist jedoch an den Lizenztyp gebunden und außerdem durch den Platz auf dem Bildschirm limitiert. Aus beiden Gründen schien es mir sinnvoll, die wichtigen Oszillatoren "RSI" und "stochastischer RSI" zu einem Indikator zusammenzufassen. Eine Möglichkeit bietet sich mithilfe von Scripting. 

## Kennsu Pine?

TradingView stellt dazu eigene Skriptsprache `Pine Script` sowie einen integrierten Skript-Editor zur Verfügung. Dieses Scripting erlaubt es dem Benutzer, die Anzeige der technischen Indikatoren und andere Funktionen anzupassen. Nähere Informationen gibt's z.B. unter [PINE SCRIPT LANGUAGE REFERENCE MANUAL](https://www.tradingview.com/study-script-reference/) oder [Pine Script Tutorial](https://www.tradingview.com/wiki/Pine_Script_Tutorial).

## Indikatoren

Die Berechnung und Darstellung der Indikatoren `RSI` und `stochastischer RSI` ähneln sich, weswegen sie gute Kandidaten für das Zusammenfassen auf einem Panel sind.

Der `Relative-Strength-Index (RSI)` ist ein Indikator, der auf Veränderungen der Marktkurse mit Auf- und Abwärtsbewegungen reagiert und mit dem sich die Stärke des aktuellen Marktes leichter beurteilen lässt. So gilt z.B. ein Messwert von 70 oder darüber als überkauft und signalisiert einen potenziellen Kursrückgang, ein Wert von 30 oder darunter hingegen als überverkauft, was einen potenziellen Kursanstieg signalisiert. Weitere Details findet man z.B. hier: [Der Relative-Stärke-Index (RSI)](https://www.oanda.com/lang/de/forex-trading/learn/trading-tools-strategies/rsi).

![Relative-Strength-Index](/assets/img/rsi.jpg)

Beim `stochastischen RSI` liegen die Schwellwerte für obengenannte Signale bei über 80 bzw. unter 20. Näheres erfährt man z.B. hier: [Stochastik-Oszillatoren](https://www.oanda.com/lang/de/forex-trading/learn/trading-tools-strategies/stochastic).

![Stochastischer RSI](/assets/img/stoch-rsi.jpg) 

## Die Script-Lösung

Folgendes Pine-Script berechnet beide Indikatoren und fasst sie zu einer Ansicht zusammen:

``` javascript
//@version=3
study(title="(Stochastic) RSI", shorttitle="(Stoch)RSI")

// RSI
src = input(close, title="RSI Source") 
len = input(14, minval=1, title="RSI Length")
up = rma(max(change(src), 0), len)
down = rma(-min(change(src), 0), len)
rsi = down == 0 ? 100 : up == 0 ? 0 : 100 - (100 / (1 + up / down))

// Stoch
rsi1 = rsi(src, len)
length = input(14, minval=1, title="Stoch Length")
smoothK = input(3, minval=1, title="K")
smoothD = input(3, minval=1, title="D")
k = sma(stoch(rsi1, rsi1, rsi1, length), smoothK)
d = sma(k, smoothD)

// Background Stoch
h0 = hline(80)
h1 = hline(20)
fill(h0, h1, color=#663399, transp=99)

// Background RSI
h2 = hline(70)
h3 = hline(30)
fill(h2, h3, color=#6A5ACD, transp=95)

// Plot Stoch
plot(k, color=#4169E1, title="K")
plot(d, color=orange, title="D")

// Plot RSI
plot(rsi, color=fuchsia, title="RSI")
```

Es kann in der Tradingview-Chartansicht direkt in den Pine-Editor kopiert und mit `Add to Chart` der aktuellen Chartansicht hinzugefügt werden:

![Pine-Editor](/assets/img/pine-editor.jpg) 

Als Ergebnis erhält man ein neues Panel, welches wie folgt aussieht:

![Kombinierter (stochastischer) RSI](/assets/img/combined-rsi.jpg) 

 

