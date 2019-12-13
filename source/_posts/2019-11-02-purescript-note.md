---
title: Purescript Note
date: 2019-11-02 17:51:06
tags: [purescript, fp]
---

Notebook for *PureScript by Example*
<https://leanpub.com/purescript/read#leanpub-auto-functions-and-records>

<!--more-->

# Chapter 2 Getting Started

```haskell
module Main where

import Control.Monad.Eff.Console

main = log "Hello, World!"
```

# Chapter 3 Functions and Records

## Simple Types (3.3)

Built-in types

- Number
- String
- Boolean
- integers
- characters
- arrays
- records
- functions

```haskell
> :type [1, 2, 3]
Array Int
```

Fuction

```haskell
add :: Int -> Int -> Int
add x y = x + y

> add 10 20
30
```

## Quantified Types (3.4)

```haskell
> :type flip
forall a b c. (a -> b -> c) -> b -> a -> c
```

forall means universally quantified type

## Notes On Indentation (3.5)

PureScript code is indentation-sensitive

## Defining Our Types (3.6)

```haskell
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

## Type Constructors and Kinds (3.7)

**List** is an example of a type constructor.

Values do not have the type List directly, but rather **List a** for some type **a**. That is, **List** takes a type argument a and constructs a new type List a.

values are distinguished by their **types**, types are distinguished by their **kinds**

```haskell
> :kind Number
Type

> import Data.List
> :kind List
Type -> Type

> :kind List String
Type
```

## Curried Functions (3.11)

Functions in PureScript take exactly one argument.

## Querying the Address Book (3.12)

```haskell
findEntry firstName lastName book = head $ filter filterEntry book
  where
    filterEntry :: Entry -> Boolean
    filterEntry entry = entry.firstName == firstName && entry.lastName == lastName
```

Note that, just like for top-level declarations, it was not necessary to specify a type signature for filterEntry. However, doing so is recommended as a form of documentation.

## Infix Function Application (3.13)

> head $ filter filterEntry book

is equals

> head (filter filterEntry book)

($) is just an alias for a regular function called apply

```haskell
apply :: forall a b. (a -> b) -> a -> b
apply f x = f x

infixr 0 apply as $
```

## Function Composition (3.14)

- **<<<** backwards composition
- **>>>** forwards composition

```haskell
(head <<< filter filterEntry) book
filter filterEntry >>> head
```

# Chapter 4 Recursion, Maps And Folds

- purescript-maybe, which defines the Maybe type constructor
- purescript-arrays, which defines functions for working with arrays
- purescript-strings, which defines functions for working with Javascript strings
- purescript-foldable-traversable, which defines functions for folding arrays and other data structures
- purescript-console, which defines functions for printing to the console

## Recursion on Array (4.4)

- The null function returns true on an empty array

```haskell
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

## Infix Operators (4.6)

```haskell
> (\n -> n + 1) `map` [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]

> (\n -> n + 1) <$> [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]

> (<$>) (\n -> n + 1) [1, 2, 3, 4, 5]
[2, 3, 4, 5, 6]
```

## Flattening Arrays (4.8)

```haskell
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

## Do Notation (4.10)

**map** and **bind** allow us to write so-called **monad comprehensions**

```haskell
factors :: Int -> Array (Array Int)
factors n = filter (\xs -> product xs == n) $ do
  i <- 1 .. n
  j <- i .. n
  pure [i, j]
```

## Guards (4.11)

```haskell
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

## Folds (4.12)

```haskell
> :type foldl
forall a b. (b -> a -> b) -> b -> Array a -> b

> :type foldr
forall a b. (a -> b -> b) -> b -> Array a -> b

> foldl (+) 0 (1 .. 5)
15
```

## Tail Recursion (4.13)

**purescript-free** and **purescript-tailrec** packages.

```haskell
fact :: Int -> Int -> Int
fact 0 acc = acc
fact n acc = fact (n - 1) (acc * n)
```

Notice that the recursive call to fact is the last thing that happens in this function - it is in tail position.

Prefer Folds to Explicit Recursion

# Chapter 5 Pattern Matching

- **purescript-globals**, which provides access to some common JavaScript values and functions.
- **purescript-math**, which provides access to the JavaScript Math module.

## Simple Pattern Matching (5.3)

```haskell
gcd :: Int -> Int -> Int
gcd n 0 = n
gcd 0 m = m
gcd n m = if n > m
            then gcd (n - m) m
            else gcd n (m - n)
```

- Number, String, Char and Boolean literals
- Wildcard patterns, indicated with an underscore (_), which match any argument, and which do not bind any names.

## Guards (5.5)

```haskell
gcd :: Int -> Int -> Int
gcd n 0 = n
gcd 0 n = n
gcd n m | n > m     = gcd (n - m) m
        | otherwise = gcd n (m - n)
```

## Array Patterns (5.6)

```haskell
isEmpty :: forall a. Array a -> Boolean
isEmpty [] = true
isEmpty _ = false

takeFive :: Array Int -> Int
takeFive [0, 1, a, b, _] = a * b
takeFive _ = 0
```

## Record Patterns and Row Polymorphism (5.7)

```haskell
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

## Nested Patterns (5.8)

```haskell
type Address = { street :: String, city :: String }

type Person = { name :: String, address :: Address }

livesInLA :: Person -> Boolean
livesInLA { address: { city: "Los Angeles" } } = true
livesInLA _ = false
```

## Named Patterns (5.9)

```haskell
sortPair :: Array Int -> Array Int
sortPair arr@[x, y]
  | x <= y = arr
  | otherwise = [y, x]
sortPair arr = arr
```

## Case Expressions (5.10)

```haskell
import Data.Array.Partial (tail)
import Partial.Unsafe (unsafePartial)

lzs :: Array Int -> Array Int
lzs [] = []
lzs xs = case sum xs of
           0 -> xs
           _ -> lzs (unsafePartial tail xs)
```

## Algebraic Data Types (5.12)

Algebraic Data Types (or ADTs)

```haskell
-- OR
data Shape
  = Circle Point Number
  | Rectangle Point Number Number
  | Line Point Point
  | Text Point String

-- AND
data Point = Point
  { x :: Number
  , y :: Number
  }

data Maybe a = Nothing | Just a

-- recursive define
data List a = Nil | Cons a (List a)
```

## Using ADTs (5.13)

```haskell
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

## Newtypes (5.15)

There is an important special case of **algebraic data types (ADT)**, called newtypes

Newtypes must define exactly one constructor, and that constructor must take exactly one argument.

a newtype gives a new name to an existing type. In fact, the values of a newtype have the same runtime representation as the underlying type. This gives an extra layer of type safety.

```haskell
newtype Pixels = Pixels Number
```

> Newtypes will become important, since they allow us to attach different behavior to a type without changing its representation at runtime.

# Chapter 6 Type Classes

## Show Me (6.3)

```haskell
class Show a where
  show :: a -> String

instance showBoolean :: Show Boolean where
  show true = "true"
  show false = "false"
```

We can annotate the expression with a type, using the :: operator, so that PSCi can choose the correct type class instance:

```haskell
> show (Left 10 :: Either Int String)
"(Left 10)"
```

## Common Type Classes (6.4)

```haskell
class Eq a where
  eq :: a -> a -> Boolean

-- Ord
data Ordering = LT | EQ | GT

class Eq a <= Ord a where
  compare :: a -> a -> Ordering

-- The Field type class identifies those types which support numeric operators such as addition, subtraction, multiplication and division. It is provided to abstract over those operators
class EuclideanRing a <= Field a

-- purescript-monoid
-- Semigroups and Monoids
-- <> is as an alias for append
class Semigroup a where
  append :: a -> a -> a

class Semigroup m <= Monoid m where
  mempty :: m

-- purescript-foldable-traversable
-- Foldable
class Foldable f where
  foldr :: forall a b. (a -> b -> b) -> b -> f a -> b
  foldl :: forall a b. (b -> a -> b) -> b -> f a -> b
  foldMap :: forall a m. Monoid m => (a -> m) -> f a -> m

-- Functor
-- alias <$>
class Functor f where
  map :: forall a b. (a -> b) -> f a -> f b
```

```haskell
> import Data.Monoid
> import Data.Foldable

> foldl append mempty ["Hello", " ", "World"]  
"Hello World"
```

Functor Law

- **identity law:** map id xs = xs
- **composition law:** map g (map f xs) = map (g <<< f) xs

## Type Annotations (6.5)

```haskell
threeAreEqual :: forall a. Eq a => a -> a -> a -> Boolean
threeAreEqual a1 a2 a3 = a1 == a2 && a2 == a3

showCompare :: forall a. Ord a => Show a => a -> a -> String
showCompare a1 a2 | a1 < a2 =
  show a1 <> " is less than " <> show a2
showCompare a1 a2 | a1 > a2 =
  show a1 <> " is greater than " <> show a2
showCompare a1 a2 =
  show a1 <> " is equal to " <> show a2
```

## Instance Dependencies (6.7)

```haskell
instance showEither :: (Show a, Show b) => Show (Either a b) where
  ...
```

## Multi Parameter Type Classes (6.8)

```haskell
module Stream where

import Data.Array as Array
import Data.Maybe (Maybe)
import Data.String as String

class Stream stream element where
  uncons :: stream -> Maybe { head :: element, tail :: stream }

instance streamArray :: Stream (Array a) a where
  uncons = Array.uncons

instance streamString :: Stream String Char where
  uncons = String.uncons
```

## Functional Dependencies (6.9)

```haskell
class Stream stream element | stream -> element where
  uncons :: stream -> Maybe { head :: element, tail :: stream }

genericTail xs = map _.tail (uncons xs)

> :type genericTail
forall stream element. Stream stream element => stream -> Maybe stream

> genericTail "testing"
(Just "esting")
```

## Nullary Type Classes (6.10)

```haskell
head :: forall a. Partial => Array a -> a

tail :: forall a. Partial => Array a -> Array a
```

Note that there is no instance defined for the Partial type class! Doing so would defeat its purpose: attempting to use the head function directly will result in a type error:

```haskell
secondElement :: forall a. Partial => Array a -> a
secondElement xs = head (tail xs)
```

```haskell
unsafePartial :: forall a. (Partial => a) -> a
```
Note that the Partial constraint appears inside the parentheses on the left of the function arrow, but not in the outer forall. That is, unsafePartial is a function from partial values to regular values.

# Chapter 7 Applicative Validation

## lift Arbitrary Functions (7.4)

```haskell
-- <$>
class Functor f where
  map :: forall a b. (a -> b) -> f a -> f b

-- <*>
class Functor f <= Apply f where
  apply :: forall a b. f (a -> b) -> f a -> f b

```

Maybe implement apply class

```haskell
instance functorMaybe :: Functor Maybe where
  map f (Just a) = Just (f a)
  map f Nothing  = Nothing

instance applyMaybe :: Apply Maybe where
  apply (Just f) (Just x) = Just (f x)
  apply _        _        = Nothing
```

```haskell
lift3 :: forall a b c d f. Apply f => 
    (a -> b -> c -> d) -> f a -> f b -> f c -> f d
lift3 f x y z = f <$> x <*> y <*> z
```

## Applicative type class (7.5)

```haskell
class Functor f where
  map :: forall a b. (a -> b) -> f a -> f b

class Functor f <= Apply f where
  apply :: forall a b. f (a -> b) -> f a -> f b

class Apply f <= Applicative f where
    pure :: forall a. a -> f a

instance applicativeMaybe :: Applicative Maybe where
  pure x = Just x
```

## More Effects (7.7)

```haskell
withError Nothing  err = Left err
withError (Just a) _   = Right a

fullNameEither first middle last =
   fullName <$> (first  `withError` "First name was missing")
            <*> (middle `withError` "Middle name was missing")
            <*> (last   `withError` "Last name was missing")

> :type fullNameEither
Maybe String -> Maybe String -> Maybe String -> Either String String
```

## Applicative Validation (7.9)

```haskell
address :: String -> String -> String -> Address

phoneNumber :: PhoneType -> String -> PhoneNumber

person :: String -> String -> Address -> Array PhoneNumber -> Person

data PhoneType = HomePhone | WorkPhone | CellPhone | OtherPhone

examplePerson :: Person
examplePerson =
  person "John" "Smith"
        (address "123 Fake St." "FakeTown" "CA")
        [ phoneNumber HomePhone "555-555-5555"
         , phoneNumber CellPhone "555-555-0000"
        ]
```

> purescript-validation

The **Data.AddressBook.Validation** module uses the **V (Array String) **applicative functor to validate the data structures

```haskell

nonEmpty :: String -> String -> V Errors Unit
nonEmpty field "" = invalid ["Field '" <> field <> "' cannot be empty"]
nonEmpty _     _  = pure unit

lengthIs :: String -> Number -> String -> V Errors Unit
lengthIs field len value | S.length value /= len =
  invalid ["Field '" <> field <> "' must have length " <> show len]
lengthIs _     _   _     =
  pure unit

matches :: String -> R.Regex -> String -> V Errors Unit
matches _     regex value | R.test regex value =
  pure unit
matches field _     _     =
  invalid ["Field '" <> field <> "' did not match the required format"]

validateAddress :: Address -> V Errors Address
validateAddress (Address o) =
  address <$> (nonEmpty "Street" o.street *> pure o.street)
          <*> (nonEmpty "City"   o.city   *> pure o.city)
          <*> (lengthIs "State" 2 o.state *> pure o.state)

validatePhoneNumber :: PhoneNumber -> V Errors PhoneNumber
validatePhoneNumber (PhoneNumber o) =
  phoneNumber <$> pure o."type"
              <*> (matches "Number" phoneNumberRegex o.number *> pure o.number)

```

## Traversable Functors (7.11)

```haskell
arrayNonEmpty :: forall a. String -> Array a -> V Errors Unit
arrayNonEmpty field [] =
  invalid ["Field '" <> field <> "' must contain at least one value"]
arrayNonEmpty _     _  =
  pure unit

validatePerson :: Person -> V Errors Person
validatePerson (Person o) =
  person <$> (nonEmpty "First Name" o.firstName *> pure o.firstName)
         <*> (nonEmpty "Last Name"  o.lastName  *> pure o.lastName)
	       <*> validateAddress o.address
         <*> (arrayNonEmpty "Phone Numbers" o.phones *> traverse validatePhoneNumber o.phones)

```

```haskell
class (Functor t, Foldable t) <= Traversable t where
  traverse :: forall a b f. Applicative f => (a -> f b) -> t a -> f (t b)
  sequence :: forall a f. Applicative f => t (f a) -> f (t a)
```

```haskell
traverse :: forall a b f. Applicative f => (a -> f b) -> Array a -> f (Array b)
traverse :: forall a b. (a -> V Errors b) -> Array a -> V Errors (Array b)
```

> Traversable functors capture the idea of traversing a data structure, collecting a set of effectful computations, and combining their effects. In fact, sequence and traverse are equally important to the definition of Traversable - each can be implemented in terms of each other

List implements

```haskell
-- traverse :: forall a b f. Applicative f => (a -> f b) -> List a -> f (List b)
traverse _ Nil = pure Nil
traverse f (Cons x xs) = Cons <$> f x <*> traverse f xs

```

## Applicative Functors for Parallelism (7.12)

```haskell
f <$> parallel computation1
  <*> parallel computation2
```
This computation would start computing values asynchronously using computation1 and computation2. When both results have been computed, they would be combined into a single result using the function f.

# Chapter 8 The Eff Monad

## 8.3 Monads and Do Notation

```haskell
import Prelude

import Control.Plus (empty)
import Data.Array ((..))

countThrows :: Int -> Array (Array Int)
countThrows n = do
  x <- 1 .. 6
  y <- 1 .. 6
  if x + y == n
    then pure [x, y]
    else empty
```

> In general, every line of a do notation block will contain a computation of type m a for some type a and our monad m
> The monad m must be the same on every line
> the types a can differ (i.e. individual computations can have different result types)

```haskell
userCity :: XML -> Maybe XML
userCity root = do
  prof <- child root "profile"
  addr <- child prof "address"
  city <- child addr "city"
  pure city
```

## 8.4 Monad type class

```haskell
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

```haskell
foldM :: forall m a b. Monad m => (a -> b -> m a) -> a -> List b -> m a
foldM _ a Nil = pure a
foldM f a (b : bs) = do
  a' <- f a b
  foldM f a' bs
```

Example:

```haskell
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

```haskell
module Main where

import Prelude

import Control.Monad.Eff.Random (random)
import Control.Monad.Eff.Console (logShow)

main = do
  n <- random
  logShow n
```

## 8.11 Extensible Effects

```haskell
> :type main
forall eff. Eff (console :: CONSOLE, random :: RANDOM | eff) Unit
```

main is a computation with side-effects, which can be run in any environment which supports random number generation and console IO, and any other types of side effect, and which returns a value of type Unit