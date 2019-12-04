---
title: Purescript Note
date: 2019-11-02 17:51:06
tags: [purescript, fp]
---

Notebook for *PureScript by Example*
<https://leanpub.com/purescript/read#leanpub-auto-functions-and-records>

<!--more-->

# Chapter 2 Getting Started

```purescript
module Main where

import Control.Monad.Eff.Console

main = log "Hello, World!"
```

# Chapter 3 Functions and Records

## 3.3 Simple Types

Built-in types

- Number
- String
- Boolean
- integers
- characters
- arrays
- records
- functions

```purescript
> :type [1, 2, 3]
Array Int
```

Fuction

```purescript
add :: Int -> Int -> Int
add x y = x + y

> add 10 20
30
```

## 3.4 Quantified Types

```purescript
> :type flip
forall a b c. (a -> b -> c) -> b -> a -> c
```

forall means universally quantified type

## 3.5 Notes On Indentation

PureScript code is indentation-sensitive

## 3.6 Defining Our Types

```purescript
-- a record type
type Entry =
  { firstName :: String
  , lastName  :: String
  , address   :: Address
  }
type Address =
  { street :: String
  , city   :: String
  , state  :: String
  }

-- type synonym
type AddressBook = List Entry

> address = { street: "123 Fake St.", city: "Faketown", state: "CA" }
```

## 3.7 Type Constructors and Kinds

**List** is an example of a type constructor.

Values do not have the type List directly, but rather **List a** for some type **a**. That is, **List** takes a type argument a and constructs a new type List a.

values are distinguished by their **types**, types are distinguished by their **kinds**

```purescript
> :kind Number
Type

> import Data.List
> :kind List
Type -> Type

> :kind List String
Type
```

## 3.11 Curried Functions

Functions in PureScript take exactly one argument.

## 3.12 Querying the Address Book

```purescript
findEntry firstName lastName book = head $ filter filterEntry book
  where
    filterEntry :: Entry -> Boolean
    filterEntry entry = entry.firstName == firstName && entry.lastName == lastName
```

Note that, just like for top-level declarations, it was not necessary to specify a type signature for filterEntry. However, doing so is recommended as a form of documentation.

## 3.13 Infix Function Application

> head $ filter filterEntry book

is equals

> head (filter filterEntry book)

($) is just an alias for a regular function called apply

```purescript
apply :: forall a b. (a -> b) -> a -> b
apply f x = f x

infixr 0 apply as $
```

## 3.14 Function Composition

- **<<<** backwards composition
- **>>>** forwards composition

```purescript
(head <<< filter filterEntry) book
filter filterEntry >>> head
```

# Chapter 4 Recursion, Maps And Folds

- purescript-maybe, which defines the Maybe type constructor
- purescript-arrays, which defines functions for working with arrays
- purescript-strings, which defines functions for working with Javascript strings
- purescript-foldable-traversable, which defines functions for folding arrays and other data structures
- purescript-console, which defines functions for printing to the console

## 4.4 Recursion on Array

- The null function returns true on an empty array

```purescript
import Prelude

import Data.Array (null)
import Data.Array.Partial (tail)
import Partial.Unsafe (unsafePartial)

length :: forall a. Array a -> Int
length arr =
  if null arr
    then 0
    else 1 + length (unsafePartial tail arr)
```

## 4.6 Infix Operators

```purescript
> (\n -> n + 1) `map` [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]

> (\n -> n + 1) <$> [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]

> (<$>) (\n -> n + 1) [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

## 4.8 Flattening Arrays

```purescript
> import Data.Array

> :type concat
forall a. Array (Array a) -> Array a

> concat [[1, 2, 3], [4, 5], [6]]
[1, 2, 3, 4, 5, 6]

> :type concatMap
forall a b. (a -> Array b) -> Array a -> Array b

> concatMap (\n -> [n, n * n]) (1 .. 5)
[1,1,2,4,3,9,4,16,5,25]
```

## 4.10 Do Notation

**map** and **bind** allow us to write so-called **monad comprehensions**

```purescript
factors :: Int -> Array (Array Int)
factors n = filter (\xs -> product xs == n) $ do
  i <- 1 .. n
  j <- i .. n
  pure [i, j]
```

## 4.11 Guards

```purescript
import Control.MonadZero (guard)

factors :: Int -> Array (Array Int)
factors n = do
  i <- 1 .. n
  j <- i .. n
  guard $ i * j == n
  pure [i, j]

> import Control.MonadZero

> :type guard
forall m. MonadZero m => Boolean -> m Unit

> length $ guard true
1

> length $ guard false
0
```

That is, if guard is passed an expression which evaluates to true, then it returns an array with a single element. If the expression evaluates to false, then its result is empty.

## 4.12 Folds

```purescript
> :type foldl
forall a b. (b -> a -> b) -> b -> Array a -> b

> :type foldr
forall a b. (a -> b -> b) -> b -> Array a -> b

> foldl (+) 0 (1 .. 5)
15
```

## 4.13 Tail Recursion

**purescript-free** and **purescript-tailrec** packages.

```purescript
fact :: Int -> Int -> Int
fact 0 acc = acc
fact n acc = fact (n - 1) (acc * n)
```

Notice that the recursive call to fact is the last thing that happens in this function - it is in tail position.

Prefer Folds to Explicit Recursion

# Chapter 5 Pattern Matching

- **purescript-globals**, which provides access to some common JavaScript values and functions.
- **purescript-math**, which provides access to the JavaScript Math module.

## 5.3 Simple Pattern Matching

```purescript
gcd :: Int -> Int -> Int
gcd n 0 = n
gcd 0 m = m
gcd n m = if n > m
            then gcd (n - m) m
            else gcd n (m - n)
```

- Number, String, Char and Boolean literals
- Wildcard patterns, indicated with an underscore (_), which match any argument, and which do not bind any names.

## 5.5 Guards

```purescript
gcd :: Int -> Int -> Int
gcd n 0 = n
gcd 0 n = n
gcd n m | n > m     = gcd (n - m) m
        | otherwise = gcd n (m - n)
```

## 5.6 Array Patterns

```purescript
isEmpty :: forall a. Array a -> Boolean
isEmpty [] = true
isEmpty _ = false

takeFive :: Array Int -> Int
takeFive [0, 1, a, b, _] = a * b
takeFive _ = 0
```

## 5.7 Record Patterns and Row Polymorphism

```purescript
showPerson :: { first :: String, last :: String } -> String
showPerson { first: x, last: y } = y <> ", " <> x

> :type showPerson
forall r. { first :: String, last :: String | r } -> String

> showPerson { first: "Phil", last: "Freeman" }
"Freeman, Phil"

> showPerson { first: "Phil", last: "Freeman", location: "Los Angeles" }
"Freeman, Phil"
```

Notice: what is the type variable r here?
> We can read the new type signature of showPerson as “takes any record with first and last fields which are Strings and any other fields, and returns a String”.

This function is polymorphic in the row r of record fields, hence the name row polymorphism.

## 5.8 Nested Patterns

```purescript
type Address = { street :: String, city :: String }

type Person = { name :: String, address :: Address }

livesInLA :: Person -> Boolean
livesInLA { address: { city: "Los Angeles" } } = true
livesInLA _ = false
```

## 5.9 Named Patterns

```purescript
sortPair :: Array Int -> Array Int
sortPair arr@[x, y]
  | x <= y = arr
  | otherwise = [y, x]
sortPair arr = arr
```

## 5.10 Case Expressions

```purescript
import Data.Array.Partial (tail)
import Partial.Unsafe (unsafePartial)

lzs :: Array Int -> Array Int
lzs [] = []
lzs xs = case sum xs of
           0 -> xs
           _ -> lzs (unsafePartial tail xs)
```

## 5.12 Algebraic Data Types

Algebraic Data Types (or ADTs)

```purescript
data Shape
  = Circle Point Number
  | Rectangle Point Number Number
  | Line Point Point
  | Text Point String

data Point = Point
  { x :: Number
  , y :: Number
  }

data Maybe a = Nothing | Just a

-- recursive define
data List a = Nil | Cons a (List a)
```

## 5.13 Using ADTs

```purescript
showPoint :: Point -> String
showPoint (Point { x: x, y: y }) = "(" <> show x <> ", " <> show y <> ")"

-- or we could define as below. Record Puns
-- showPoint (Point { x, y }) = ...

showShape :: Shape -> String
showShape (Circle c r)      = ...
showShape (Rectangle c w h) = ...
showShape (Line start end)  = ...
showShape (Text p text) = ...
```

## 5.15 Newtypes

There is an important special case of **algebraic data types (ADT)**, called newtypes

Newtypes must define exactly one constructor, and that constructor must take exactly one argument.

a newtype gives a new name to an existing type. In fact, the values of a newtype have the same runtime representation as the underlying type. This gives an extra layer of type safety.

```purescript
newtype Pixels = Pixels Number
```

> Newtypes will become important, since they allow us to attach different behavior to a type without changing its representation at runtime.

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