---
title: Purescript Types
date: 2019-12-12 14:49:01
tags: [purescript, fp]
---

Some though about types in purescript

- **data Foo = Foo Int**: Foo and Int totally different.
- **newtype Foo = Foo Int**: Foo and Int different at compile time but same at runtime.
- **type Foo = Int**: Foo and Int same at compile time and runtime.