## elemIndex

```haskell
elemIndex :: Eq a => a -> [a] -> Maybe Int
elemIndex a xs = go 0 a xs where
    go _ _ [] = Nothing
    go i a (x:xs)
        | a == x = Just i
        | otherwise = go (i + 1) a xs
```

Given some type `t`, the annotation `Eq t` suggests that there is an implementation of the equality operator `(==) :: t -> t -> Bool`.

The need for the constraint `Eq a` is necessiated by the `(a == x)` guard in the `go` help function.

## `myFunc` : a Mystery Function

`myFunc 2 [(+1), (+2), (+3)]` is well-typed and reduces to `[3,4,5]`.

`myFunc (+1) [1,2,3]` is ill-typed.

The most general type of `myFunc` is `myFunc :: a -> [a -> b] -> [b]`.

The instance can be completed as follows:

```haskell
instance Applicative [] where
    pure :: a -> [a]
    pure x = [x]
    (<*>) :: [a -> b] -> [a] -> [b]
    fs <*> xs = [f x | f <- fs, x <- xs]
```

With these definitions, one could write `myFunc2 x fs = fs <*> pure x`. Then, `myFunc2` would have the same behavior as `myFunc`.

## Unfolding

`take 10 $ unfoldr (\x -> Just (x, x)) 'a'` produces "aaaaaaaaaa"

`take 10 $ unfoldr (\x -> Just (x + 1, x + 1)) 0` produces `[1..10]`

The following function `op1` has the desired property:

```haskell
op1 :: Int -> Int -> Maybe (Int, Int)
op1 n x
    | x > n = Nothing
    | x <= n = Just (x, x + 1)
```

The following function `op2` has the desired property:

```haskell
op2 :: (Int, Int) -> Maybe (Int, (Int, Int))
op2 (old, new) = Just (old, (new, new + old))
```

Namely, `unfoldr (op1 n) 0` evaluates to `[0,1,2..n]` and `take n $ unfoldr op2 (0, 1)` produces the first `n` fibonacci numbers.

## (Int ->) is a Functor

We can turn `Trace` into a Functor this way.

```haskell
type Trace a = Int -> a

instance Functor Trace where
  fmap f t = \i -> f (t i)
```

Fix some trace `t`. We wish to show that `id t = fmap id t`. Indeed, let `i` be an arbitrary index. Then, `(id t) i = t i`, by definition of `id`. Also, `(fmap id t) i = (\i -> id (t i)) i = id (t i) = t i`.

We also wish to show that `fmap (f . g) t = (fmap f . fmap g) t`. As before, consider an arbitrary index `i`. Then, we have

```
(fmap (f . g) t) i
= (\i -> (f . g) (t i)) i 
= (f . g) (t i) 
= f (g (t i))
```

.

And, also 

```
((fmap f . fmap g) t) i 
= (fmap f (fmap g t)) i 
= (\i -> f ((fmap g t) i)) i
= f ((fmap g t) i)
= f ((\i -> g (t i)) i)
= f (g (t i))
```

This shows `fmap (f . g) t = (fmap f . fmap g) t` as we wanted.

## Playing with IO

`myAction5` will produce an error. This is because the last line of a `do` block cannot end in a `<-` binding.

The types of the other expressions are as follows:

```haskell
mysteryAction :: String -> IO Int
mysteryAction = undefined

myAction1 :: String -> IO Int
myAction1 s = do
    getLine
    putStrLn s
    mysteryAction s

myAction2 :: IO String
myAction2 = do
    myAction1 "ABC"
    getLine

myAction3 :: IO ()
myAction3 = do
    s <- myAction2
    myAction1 s
    putStrLn s

myAction4 :: IO ()
myAction4 = do
    myAction3
    pure ()
```