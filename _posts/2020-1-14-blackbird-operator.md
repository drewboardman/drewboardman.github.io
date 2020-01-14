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
figured was a good topic to dive deeper into. It revolves around something
called the *blackbird* which is a [Smullyan
combinator](https://en.wikipedia.org/wiki/To_Mock_a_Mockingbird), and can be
thought of as the "composition of *compose*`(.)` and *compose*`(.)`".

Blackbird operator

```haskell
(.:) :: (c -> d) -> (a -> b -> c) -> a -> b -> d
(.:) = (.) . (.)
```

I will get to how this operator is useful, but we'll need to start at the
beginning. During the video, they decide they need to create a function that
gets the sum of the lengths in a list of lists.

```haskell
totalNumber xs = sum $ map length xs

Prelude> xs = [[1,2],[3]]
Prelude> totalNumber xs
3
```

We can refactor this into point-free style

```haskell
totalNumber = sum . (map length)
```

It would be nice to generalize this over all functions `f` instead of
`length`

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

Just looking at our function we see:

```haskell
(.) sum == (sum .) == \f -> sum . f
```

Now we can finally write this in a (really ugly) point free way

```haskell
aggregate = (sum .) . map
```

I think this is where knowledge of the blackbird's existence has to come into
play, so I'll just post the combinator here.

```haskell
\f g x y -> f (g x y)
```

You could probably just get this same combinator by refactoring `aggregate`, but
the intuition to do so seems very unlikely unless you already knew you had
blackbird as a goal. Also a *combinator* is just a lambda expression that refers
only to its arguments and nothing outside of the lambda.


How does `aggregate = (sum .) . map` become `\f g x y -> f (g x y)`? Well, let's
expand it and see:

```haskell
aggregate      = (sum .) . map
aggregate      = (.) sum . map
aggregate f    = (.) sum $ map f
aggregate f    = (.) sum (map f)
aggregate f    = sum . (map f)
aggregate f    = sum . map f
aggregate f xs = sum $ map f xs
aggregate f xs = sum (map f xs)
```

You can see that

```haskell
aggregate f xs = sum (map f xs)
               = \f xs -> sum (map f xs)
```

is of the form `\f g x y -> f (g x y)`. This means that our original point-full
implementation of `aggregate` is the blackbird combinator. You should also see
that we arrive back at our original implementation but doing this exercise:

```haskell
aggregate f    = sum . (map f)
aggregate f xs = sum (map f xs)
```

So here is that *intuition* that I referred to earlier. We, at two points in our
exercise, arrived at something that should inform you that the blackbird can be
used.

```haskell
aggregate f = sum . (map f) -- of the form \f g x y -> f (g x y)

aggregate   = (sum .) . map -- of the form (f .) . g
```

Any position you find yourself in where you have a function that can fit one of
these forms, you can intuit that blackbird can be used to create a point-free
version that is easier to read (given that you're familiar with blackbird in the
first place).

To make it easier to read, let's use that infix operator we showed at the
beginning of the post. Creating this symbol for the blackbird `(.:)`, we have
the following relationship

```haskell
(.:) = \f g x y -> f (g x y)
     = \f g     -> (f .) . g
```

However, if you remember from the beginning of this post - blackbird looks like
`(.) . (.)`. How does one get from `(f .) . g` to this final result? Consider
the following reduction.

```haskell
(.:) = \f g -> (f .) . g
(.:) = \f g -> (.) (f .) g
(.:) = \f g -> (.) (f .) $ g
(.:) = \f ->   (.) (f .)
(.:) = \f ->   (.) ((.) f)
(.:) = \f ->   (.) $ (.) f
(.:) =         (.) . (.)
```

Now we can apply this to our `aggregate` function

```haskell
aggregate f xs = sum (map f xs)
aggregate      = (sum .) . map
aggregate      = sum .: map
```

There you have it, creating a point-free version of our original function
using the infix operator form of the blackbird combinator.
