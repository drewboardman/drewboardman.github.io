---
layout: post
title:  "BlackBird Operator"
date:   2020-1-14 9:01:02 -0400
categories: jekyll update
---

![image](/assets/images/blackbird.png)

I recently watched the [point-free or
die](https://www.youtube.com/watch?v=seVSlKazsNk) video that was posted on the
Haskell Discord channel. There was an interesting refactor that was done that I
figured was a good topic to dive deeper into.

During the video, they decide they need to create a function that gets the sum
of the lengths in a list of lists.

```haskell
totalNumber xs = sum $ map length xs

Prelude> xs = [[1,2],[3]]
Prelude> totalNumber xs
3
```

Then moving this to point-free you get

```haskell
totalNumber = sum . (map length)
```

Which can then be generalized to

```haskell
aggregate f = sum . (map f)
```

However, that generalization breaks the point-free style we got with
`totalNumber`. We can play around and refactor this (like in the video) to get
it a little closer to something that looks like point-free.

```haskell
aggregate f =     sum . (map f)
aggregate f = (.) sum   (map f)
aggregate f = (.) sum $  map f
aggregate   = (.) sum .  map
aggregate   = ???
```

To refactor this, you can get familiar with [operator
sections](https://wiki.haskell.org/Section_of_an_infix_operator). A basic
example is:

```haskell
(1+) == (+) 1 == \x -> 1 + x
```

Now just looking at our function we see:

```haskell
(.) sum == (sum .) == \f -> sum . f
```

So now we can finally write this in a (really ugly) point free way

```haskell
aggregate = (sum .) . map
```

So now I think is where knowledge of the blackbird's existence has to come into
play, so I'll just post the combinator here. Also a *combinator* is just a
lambda expression that refers only to its arguments and nothing outside of the
lambda.

```haskell
\x y -> f(g x y)
```

So how does `aggregate = (sum .) . map` become `\x y -> f(g x y)`? Well, let's
expand it and see:

```haskell
aggregate      = (sum .) . map
aggregate      = (.) sum . map
aggregate f    = (.) sum $ map f
aggregate f    = sum . map f
aggregate f xs = sum $ map f xs
aggregate f xs = sum (map f xs)
```

Now you can see that `aggregate f xs = sum (map f xs) == \f xs -> sum (map f
xs)` is of the form `\f g x y -> f (g x y)`. This means that our original point-full
implementation of `aggregate` is the blackbird combinator.

So if we create a symbol for the blackbird `(.:)`, we have the following
relationship

```haskell
(.:) = \f g x y -> f (g x y) = \f g -> (f .) . g
```

Now we can make this point-free and get to our final, easier to understand,
result

```haskell
(.:) = \f g -> (f .) . g
(.:) = \f g -> (.) (f .) g
(.:) = \f g -> (.) (f .) $ g
(.:) = \f ->   (.) (f .)
(.:) = \f ->   (.) ((.) f)
(.:) = \f ->   (.) $ (.) f
(.:) =         (.) . (.)
```

So now we can apply this to our `aggregate` function

```haskell
aggregate f xs = sum (map f xs)
aggregate      = (sum .) . map
aggregate      = sum .: map
```

And there you have it, creating a point-free version of our original function
using the infix operator form of the blackbird combinator.
