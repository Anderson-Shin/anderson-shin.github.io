---
title: "How do we procced trading?"
collection: finance
permalink: /finance/trade_method
excerpt: 'Several ways of trading in a stock market'
venue: "Finance post 4"
date: 2024-01-22
location: ""
---

# How to trade?

# How to Trade Securities

### Investing in Financial Markets

Three main type of capital market

1. Money Market - borrowing and lending large amounts of money for short periods 
자금 시장 :단기금융시장이라고도 하며, 보통 1년 만기 금융상품이 거래되는 시장.
2. Bond Markets (Debt Market, or Fixed Income Market) : 채권시장, 보통 12개월에서 30년 기간동안 대출 받은 것에 대한 이자를 냄.
3. Equity Market(Stock Market) - involve medium- to long term borrowing by issues of stocks; they can become a shareholder.

Primary market : 발행시장; 새로 발행하는 주식을 선보이는 시장

- public offering : 공모라고 하며 다수의 일반 투자자에게 주식을 발행하는경우다. 사모 발행에 비해 비용과 시간이 많이 들지만 더 많은 자금을 조달할 수 있다. 발행기업이 증권회사와 같은 인수기관에 발행물량을 넘겨주는 총액인수방식으로 진행되는 경우가 많다.
- Private placement : 사모 발행; 소수의 투자자들에게 주식을 발행하는 경우이며 금융기관, 지인등이 투자자가 됨.
- Initial Public Offering (IPO) : 상장 심사를 위한 기업공개철차이며, 회사가 상장하기 위해 심사를 받고 주주를 모집하기 위해 기업의 중요정보를 공개하는 절차임.
- Seasoned equity issue : 자본 증자라고 하며 이미 상장된 회사에서 발행한 새로운 주식임. 계절 상품에는 주주가 판매한 주식, 신주 또는 둘 모두 포함될 수 있음.

Secondary market : 발행시장에서 새로 발행한 주식을 거래하는 시장

- New York Stock Exchange (NYSE)
- American Stock Exchange
- HKEX

## Types of orders

- Market orders - Buy or sell orders that are to be executed immediately at current market price
- Price-contingent Orders
    - Limit orders → Limit-Buy Order = Buy at the price below the decided limit price(정해놓은 리밑 아래로 사는것), Limit-Sell Order = Sell at the price above the decided limit(정해놓은 리밑 위로 파는 것 )
    - Stop orders → Stop-loss order = Sell at the price below the decided limited price (정해놓은 리밑 아래로 파는 것), Stop-buy order = buy at the price above the decided limited price (정해놓은 리밑 위로 사는 것)

## Short sale

Short sale is strategies to speculate downward movement of security price.

1. The broker borrows securities from their clients who own them and lends them to a person who wants to engage in a short sale.
2. The person who borrows the securities sells them in the market at the current market price.
3. The proceeds from selling the borrowed securities are held by the broker.
4. The person who initiated the short sale will then wait for the market price of the securities to decline.
5. If the market price decreases as anticipated, the person can buy back the securities at a lower price.
6. The person returns the securities to the broker, effectively closing the short position.
7. The difference between the initial selling price and the lower buying price represents the profit made by the person who engaged in the short sale.

It's important to note that short selling carries certain risks, as the potential loss is theoretically unlimited if the price of the security increases significantly instead of decreasing. 

To mitigate this risk, margin calls can be introduced. If the person engaging in the short sale reaches a certain proportion of the original borrowed value, the broker may issue a margin call, which requires the person to provide additional funds or securities to cover the potential losses.

## Buying on margin

- 신용매수; 중개인에게 담보를 제공하여 증권을 구매하기 위한 자금을 빌림. 추가 주식을 구매하거나 포트폴리오의 포지션에 대한 거래 이익의 일부를 사용하는 것을 의미함.
- Bank provide extra money with money call rate needed for broker → broker send this amount to client →client buys extra portion of stocks needed and pays broker with call rate + service charge
- Initial Margin requirement > maintenance margin requirement.
- maintenance margin requirement is set up to make sure margin account value doesn’t drop below zero. If the equity value is lesser than maintenance margin, investor receive margin call from the broker and is expected to top up the margin account
- Initial margin = $\frac{equity}{(equity + liability)}$ ,  Actual margin = $\frac{Equity_{new}}{Asset_{new}}$
- If Actual margin < Initial margin it is bearable but 
if Actual margin < Maintenance margin, investor receives margin call.
- Conclusion; Initial margin requirement > 0.5 
if initial margin requirement(0.5)> Actual margin > maintenance margin (0.3) → it is okay to keep invest without margin call.
But if, Actual margin < Maintenance margin (0.3); investor should top up his/her margin account. Broker has right to sell securities to achieve Actual margin greater than Maintenance margin.

![how_to_trade_1](https://github.com/Anderson-Shin/Anderson-Shin.github.io/blob/master/images/how%20to%20trade1.png?raw=true)

a.

from above we can obtain that 

$$
initial\space margin = 34\times 2000 - 25000=45000
$$

$$
1year\space interest=0.08\times25000=2000
$$

$$
P_{break-point}=\frac{45000+25000+2000}{2000}=\$36
$$

b.

$$
\Delta return=\frac{2000(40-35)-2000}{45000}\approx17.78\%
$$

c.

$$
\Delta return=\frac{2000(30-35)-2000}{45000}\approx-26.67\%
$$

di)

for part b, if there was no borrowing of money;

$$
\Delta return=\frac{2000(40-35)}{70000}\approx14.29\%
$$

for part c, if there was no borrowing of money:

$$
\Delta return=\frac{2000(30-35)}{70000}\approx-14.29\%
$$

dii)

From above we can see that buying on margin return rate and simple equity return rate differs. If we compare the positive return rate which is the profit, we can see trading on margin amplifies the return rate compare to simple equity return rate. And if we look at negative return rate which is the loss, we can see trading on margin shows amplified loss. Buying on margin involves getting a loan from the brokerage and using the money from the loan to invest in more securities than what we can buy with our own available budget. Through margin buying, we can amplify returns but we have to bear in mind that trading on margin is a right option if the investments outperform the cost of the loan.