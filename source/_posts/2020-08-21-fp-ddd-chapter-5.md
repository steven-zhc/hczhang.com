---
title: fp-ddd chapter 5
date: 2020-08-21 22:02:25
tags: [fp, f#, ddd]
photos:
  - https://live.staticflickr.com/65535/50235018038_f4505db75a_z.jpg
---

# Domain Modeling Made Functional - Chapter 4

## Domain Modeling with Types

<!--more-->

we’re using a namespace in F# to indicate a DDD bounded context

```haskell
namespace​ OrderTaking.Domain

type WidgetCode = WidgetCode of string
type GizmoCode = GizmoCode of string
type ProductCode =
  | Widget of WidgetCode
  | Gizmo of GizmoCode

type UnitQuantikty = UnitQuantity of int
type KilogramQuantity = KilogramQuantity of decimal
type OrderQuantity =
  | Unit of UnitQuantity
  | Kilos of KilogramQuantity

type OrderId = Undefined
type OrderLineId = Undefined
type CustomerId = Undefined

type CustomerInfo = Undefined
type CustomerInfo = Undefined
type ShippingAddress = Undefined
type ShippingAddress = Undefined
type BillingAddress = Undefined
type Price = Undefined
type BillingAmount = Undefined

type Order = {
  Id: OrderId  // id for entity
  CustomerId: CustomerId  ​// customer reference
  ShippingAddress: ShippingAddress
  BillingAddress: BillingAddress
  OrderLines: OrderLInt list
  AmountToBill: BillingAmount
}

and OrderLine = {
  Id: OrderLineID // id for entity
}
```

## Workflow

```haskell
// input
type UnvalidatedOrder = {
  OrderId: string
  CustomerInfo: ...
  ShippingAddress: ...
}

// output
type PlaceOrderEvents = {
  AcknowledgmentSent: ...
  OrderPlaced: ...
  BillableOrderPlaced: ...
}

// Errors
type PlaceOrderError =
  | ValidationError of ValidationError list
  | .. // other errors

and ValidationError = {
  FieldName: string
  ErrorDescription: string
}

// The "place order" process
type PlaceOrder = 
  UnvalidatedOrder -> Result<PlaceOrderEvents, PlaceOrderError>
```
