---
title: fp-ddd chapter 7
date: 2020-08-22 16:44:04
tags: [fp, f#, ddd]
photos:
  - https://live.staticflickr.com/65535/50235886827_c1fd30c7db_z.jpg
---

# Domain Modeling Made Functional - Chapter 5

## Modeling Workflows as Pipelines

<!--more-->

> Transformation-oriented programming

We’ll create a “pipeline” to represent the business process, which in turn will be built from a series of smaller “pipes.” Each smaller pipe will do one transformation, and then we’ll glue the smaller pipes together to make a bigger pipeline. 

- The Workflow Input

- Commands as Input

  - The command should contain everything that the workflow needs to process the request

  - Sharing common structures using Generics

## Modeling an Order as a Set of States

A much better way to model the domain is to create a new type for each state of the order. This allows us to eliminate implicit states and conditional fields.

```haskell
​ type​ Order =
​   | Unvalidated ​of​ UnvalidatedOrder
​   | Validated ​of​ ValidatedOrder
​   | Priced ​of​ PricedOrder
```

## State Machines

> we could encode that requirement directly in the function signature, using the compiler to ensure that that business rule was complied with.

A much better approach is to make each state have its own type, which stores the data that is relevant to that state (if any).

``` haskell
​​type​ Item = ...
​​type​ ActiveCartData = { UnpaidItems: Item ​list​ }
​​type​ PaidCartData = { PaidItems: Item ​list​; Payment: ​float​ }
​
​​type​ ShoppingCart =
​  | EmptyCart  ​// no data​
​  | ActiveCart ​of​ ActiveCartData
​  | PaidCart ​of​ PaidCartData
```

A command handler is then represented by a function that accepts the entire state machine (the choice type) and returns a new version of it (the updated choice type).

## Modeling Each Step in the Workflow with Types

We’ve been talking about modeling processes as functions with inputs and outputs. But how do we model these dependencies using types? Simple, we just treat them as functions, too. The type signature of the function will become the “interface” that we need to implement later.

```haskell
​type​ CheckProductCodeExists =
  ProductCode -> ​bool​
  ​// ^input      ^output
```
> We have put the dependencies first in the parameter order and the input type second to last, just before the output type. The reason for this is to make partial application easier (the functional equivalent of dependency injection). 

> Booleans are generally a bad choice in a design, though, because they are very uninformative. It would be better to use a simple Sent/NotSent choice type instead of a bool.

```haskell
​type​ SendResult = Sent | NotSent
​
​​type​ SendOrderAcknowledgment =
​    OrderAcknowledgment -> SendResult
```

In summary

```haskell
// Event
​type​ OrderAcknowledgmentSent = {
  OrderId : OrderId
  EmailAddress : EmailAddress
  }

​type​ AcknowledgeOrder =
  CreateOrderAcknowledgmentLetter     ​// dependency​
    -> SendOrderAcknowledgment        ​// dependency​
    -> PricedOrder                    ​// input​
    -> OrderAcknowledgmentSent option ​// output
```

## Creating the events to return

the workflow returns a list of events, where an event can be one of OrderPlaced, BillableOrderPlaced, or OrderAcknowledgmentSent.

```haskell
​type​ PlaceOrderEvent =
​  | OrderPlaced ​of​ OrderPlaced
​  | BillableOrderPlaced ​of​ BillableOrderPlaced
​  | AcknowledgmentSent  ​of​ OrderAcknowledgmentSent

​​type​ CreateEvents =
​   PricedOrder -> PlaceOrderEvent ​list
```