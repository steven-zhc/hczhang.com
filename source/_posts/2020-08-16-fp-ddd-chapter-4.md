---
title: fp-ddd chapter 4
date: 2020-08-16 02:04:01
tags: [fp, f#]
photos:
    - https://photos.app.goo.gl/tkutMfjVERysmrut8
---

Domain Modeling Made Functional - Chapter 4

Understand types in F#
<!--more-->

```haskell
type CheckNumber = CheckNumber of int
type CardNumber = CardNumber of string

type CardType =
  Visa | Mastercard

type CreditCardInfo = {
  CardType: CardType
  CardNumber: CardNumber
}

// -------
type PaymentMethod =
  | Cash
  | Check of CheckNumber
  | Card of CreditCardInfo

type PaymentAmount = PaymentAmount of decimal
type Currency = EUR | USD

type Payment = {
  Amount: PaymentAmount
  Currency: Currency
  Method: PaymentMethod
}
```

