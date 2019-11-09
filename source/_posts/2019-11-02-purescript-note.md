---
title: purescript note
date: 2019-11-02 17:51:06
tags: [purescript, fp]
---

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

# Chapter 7

```purescript
-- <?>
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

<!--more-->

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
            <*> (middle `withError` "Middle name was missing")            <*> (last   `withError` "Last name was missing")
```

