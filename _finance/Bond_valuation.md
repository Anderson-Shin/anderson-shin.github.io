---
title: "Valuation of bond price at its maturity"
collection: finance
permalink: /finance/bond_valuation
excerpt: 'Outlook for bond price value method'
venue: "Finance post 3"
date: 2024-01-15
location: ""

---

# Valuation of Bond Price

## Bond price calculation

$F$ = Face value, $c$ = Coupon rate in semi-annual payment, $T$ = remaining years of maturity

let $i$ be the yield to maturity (yield rate , market interest), and $P$ be the bond price.

$P = (\sum_{t=1}^{2T}F\times\frac{c}{2}\times \frac{1}{(1+\frac{i}{2})^t}) + \frac{F}{(1+\frac{i}{2})^{2T}}$

→ $P =(F\times\frac{c}{2}) \sum_{t=1}^{2T}\frac{1}{(1+\frac{i}{2})^t}+\frac{F}{(1+\frac{i}{2})^{2T}}$ →$P =(F\times\frac{c}{2})\times \frac{1-\frac{1}{(1+\frac{i}{2})^{2T}}}{\frac{i}{2}} +F\times \frac{1}{(1+\frac{i}{2})^{2T}}$

![Q1bond price](https://github.com/Anderson-Shin/Anderson-Shin.github.io/blob/master/images/Q1bondprice.png?raw=true)

given above setting where bond is redeemed at its face value, yield rate = coupon rate. 
below is the constructed table to visualize the idea using formula.

$$
P = (\sum_{t=1}^{2T}F\times\frac{c}{2}\times \frac{1}{(1+\frac{i}{2})^t}) + \frac{F}{(1+\frac{i}{2})^{2T}}
$$

![data table](https://github.com/Anderson-Shin/Anderson-Shin.github.io/blob/master/images/table%20fof%20bond%20price.png?raw=true)

If we set the different values of yield rate given the same condition as table above(maturity of 8years, coupon rate of 4%, Face value of 1) we can achieve the trend of yield rate and value of bond as below

![data plotted](https://github.com/Anderson-Shin/Anderson-Shin.github.io/blob/master/images/plot%20of%20bond%20price.png?raw=true)

By looking at the graph we can clearly see the inverse relationship between yield rate and bond value at time(t).