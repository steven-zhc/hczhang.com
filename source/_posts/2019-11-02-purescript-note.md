---
title: Purescript Note
date: 2019-11-02 17:51:06
tags: [purescript, fp]
---

Notebook for *PureScript by Example*
<https://leanpub.com/purescript/read#leanpub-auto-functions-and-records>

<!--more-->

# Chapter 5

## 5.12 Algebraic Data Types

Algebraic Data Types (or ADTs)

```purescript
data Point = Point
  { x :: Number
  , y :: Number
  }

data Maybe a = Nothing | Just a

-- recursive define
data List a = Nil | Cons a (List a)

```

## newtype

Newtypes must define exactly one constructor, and that constructor must take exactly one argument.

```purescript
newtype Pixels = Pixels Number
```

# Chapter 7 Applicative Validation

```purescript
-- <$>
class Functor f where
  map :: forall a b. (a -> b) -> f a -> f b

-- <*>
class Functor f <= Apply f where
  apply :: forall a b. f (a -> b) -> f a -> f b

```

Maybe implement apply class

```purescript
instance functorMaybe :: Functor Maybe where
  map f (Just a) = Just (f a)
  map f Nothing  = Nothing

instance applyMaybe :: Apply Maybe where
  apply (Just f) (Just x) = Just (f x)
  apply _        _        = Nothing
```

## lift

```purescript
lift3 :: forall a b c d f. Apply f => 
    (a -> b -> c -> d) -> f a -> f b -> f c -> f d
lift3 f x y z = f <$> x <*> y <*> z
```

## Applicative type class

```purescript
class Apply f <= Applicative f where
    pure :: forall a. a -> f a
```

```purescript
withError Nothing  err = Left err
withError (Just a) _   = Right a

fullNameEither first middle last =
   fullName <$> (first  `withError` "First name was missing")
            <*> (middle `withError` "Middle name was missing")
            <*> (last   `withError` "Last name was missing")
```

# Chapter 8 The Eff Monad

## 8.4 Monad type class

```purescript
-- >>=
class Apply m <= Bind m where
    bind :: forall a b. m a -> (a -> m b) -> m b

class (Applicative m, Bind m) <= Monad m

instance bindArray :: Bind Array where
    bind xs f = concatMap f xs

instance bindMaybe :: Bind Maybe where
    bind Nothing  _ = Nothing
    bind (Just a) f = f a
```

## 8.5 Monad Laws

```haskell
-- Left identity
return a >>= f ≡ f a

-- Right identity
m >>= return ≡ m

-- Associativity
(m >>= f) >>= g ≡ m >>= (\x -> f x >>= g)
```

## 8.6 Folding With Monads

```purescript
foldM :: forall m a b. Monad m => (a -> b -> m a) -> a -> List b -> m a
foldM _ a Nil = pure a
foldM f a (b : bs) = do
  a' <- f a b
  foldM f a' bs
```

Example:

```purescript
import Data.List

safeDivide :: Int -> Int -> Maybe Int
safeDivide _ 0 = Nothing
safeDivide a b = Just (a / b)

-- (Just 5)
foldM safeDivide 100 (fromFoldable [5, 2, 2]) 

-- Nothing
foldM safeDivide 100 (fromFoldable [2, 0, 4])
```

## 8.7 Monads and Applicatives

> Every instance of the Monad type class is also an instance of the Applicative type class

## 8.8 Native Effects

> The Eff monad is defined in the Prelude, in the Control.Monad.Eff module. It is used to manage so-called native side-effects.

## 8.9 Side-Effects and Purity

> The answer is that PureScript does not aim to eliminate side-effects. It aims to represent side-effects in such a way that pure computations can be distinguished from computations with side-effects in the type system. In this sense, the language is still pure.

> *main* is required to be a computation in the Eff monad

## 8.10 The Eff Monad

Simple example.

This program uses do notation to combine two types of native effects provided by the Javascript runtime: random number generation and console IO.

```purescript
module Main where

import Prelude

import Control.Monad.Eff.Random (random)
import Control.Monad.Eff.Console (logShow)

main = do
  n <- random
  logShow n
```

## 8.11 Extensible Effects

```purescript
> :type main
forall eff. Eff (console :: CONSOLE, random :: RANDOM | eff) Unit
```

main is a computation with side-effects, which can be run in any environment which supports random number generation and console IO, and any other types of side effect, and which returns a value of type Unit