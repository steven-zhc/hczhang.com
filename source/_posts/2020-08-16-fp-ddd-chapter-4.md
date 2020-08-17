---
title: fp-ddd chapter 4
date: 2020-08-16 02:04:01
tags: [fp, f#]
photos:
    - https://live.staticflickr.com/65535/50235671576_a3090a7059_z.jpg
---

# Domain Modeling Made Functional - Chapter 4

## Understand F# types

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

