## elemIndex

* (15 points) Define the function `elemIndex :: Eq a => a -> [a] -> Maybe Int` that takes an element and a list and returns the index of the first occurrence of the element in the list, if there is any. 

Here is an example of how it should behave

```
elemIndex 'a' "abcd" = Just 0
elemIndex 'b' "abcd" = Just 1
elemIndex 1 [2, 3, 4, 1] = Just 3
elemIndex 'x' "abcd" = Nothing
```

* (5 points) Describe in brief what the annotation `Eq a` means and indicate which part of your code necessiates it.

## `myFunc` : a Mystery Function

Consider the function `myFunc` below.

```haskell
myFunc _ [] = []
myFunc x (f:fs) = f x : myFunc x fs
```

* (2 points) Which of the two following expressions is well typed?

```haskell
-- Expression 1
myFunc 2 [(+1), (+2), (+3)]
-- Expression 2
myFunc (+1) [1,2,3]
```

* (1 point) What is the normal form of the expression above which is well typed?
* (7 points) What is the most general type of the function `myFunc`?

Recall that if a type constructor `f :: * -> *` is instance of `Applicative`, then we have two functions:

```haskell
class Applicative f where
  pure :: a -> f a
  (<*>) :: f (a -> b) -> f a -> f b
```

* (15 points) We are interested in the case where `f` is the type constructor of lists. Complete the following instance:

```haskell
instance Applicative [] where
    pure :: a -> [a]
    pure = undefined
    (<*>) :: [a -> b] -> [a] -> [b]
    (<*>) = undefined
```

In order to make sure that your definitions are correct, you should make sure that the following identities hold:

```haskell
pure id <*> v = v    
pure f <*> pure x = pure (f x) 
u <*> pure y = pure (\f -> f y) <*> u
pure (.) <*> u <*> v <*> w = u <*> (v <*> w)
```

(Remember that `id :: a -> a` is defined as `id x = x` and `(.)` is function composition where `(f .g ) x` is defined as `f (g x)`.)

You don't need to give a proof of these identities (or write anything to justify that they hold). Also, unless you pick a degenerate definition of the functions above, these should almost automatically hold.

* (10 points) Now, give a short definition of `myFunc2` using `<*>` and `pure`.

## Unfolding

Carefully try to understand the following function

```haskell
unfoldr :: (b -> Maybe (a, b)) -> b -> [a]
unfoldr f b = case f b of
    Nothing -> []
    Just (x, b') -> x : unfoldr f b'
```

* (3 points) Find the normal form of the expression `take 10 $ unfoldr (\x -> Just (x, x)) 'a'`
* (3 points) Find the normal form of the expression `take 10 $ unfoldr (\x -> Just (x + 1, x + 1)) 0`
* (6 points) Find `op1 :: Int -> (Int -> Maybe (Int, Int))` such that `unfoldr (op1 n) 0` evaluates to `[0,1,2..n]`.
* (14 points) Find `op2 :: (Int, Int) -> Maybe (Int, (Int, Int))` such that `take n $ unfoldr op2 (0, 1)` produces the first `n` natural numbers. For instance, you should have that `take 10 $ unfoldr op (0, 1)` produces `[0,1,1,2,3,5,8,13,21,34]`


## (Int -> ) is a Functor

Consider `type Trace a = Int -> a`.

* (10 points) Fill in the definition of `fmap` for the `Trace` type constructor.

```haskell
instance Functor Trace where
  fmap :: (a -> b) -> Trace a -> Trace b
  fmap = undefined
```

Recall that the identity `id :: a -> a` and the composition function `(.) :: (b -> c) -> (a -> b) -> a -> c` is defined as follows.

```haskell
id :: a -> a
id x = x
(.) :: (b -> c) -> (a -> b) -> a -> c
f . g = \x -> f (g x)
```

* (4 points) Show that for a given trace `t`, `fmap id t` is the same as `id t`. You can show this by carefully considering the definitions.
    * Hint: We say that two traces `t1, t2` are the same if for all indices `i`, `t1 i = t2 i`.
* (6 points) Check that for a given trace `t :: Trace a`, and two functions `f :: a -> b` and `g :: b -> c`, `fmap (f . g) t` is the same as `(fmap f . fmap g) t`. Use the hints from the last question to structure your proof.

## Playing with IO

Recall that `getLine :: IO String` and `putStrLn :: String -> IO ()`. Suppose that `mysteryAction :: String  -> IO Int`

* (4 points) Which of the following 5 definitions will result in an error?
* (16 points) For the other 4 definitions, write down their types.

```haskell
myAction1 s = do
    getLine
    putStrLn s
    mysteryAction s

myAction2 = do
    myAction1 "ABC"
    getLine

myAction3 = do
    s <- myAction2
    myAction1 s
    putStrLn s

myAction4 = do
    myAction3
    pure ()

myAction5 = do
    s <- myAction2
    myAction1 s
    putStrLn s
    s <- myAction2
```
