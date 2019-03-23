---
layout: post
title:  "Typeclasses in Scala"
date:   2019-03-18 19:52:02 -0500
categories: jekyll update
---
I was reading the School of Haskell's [Type Families and
Pokemon](https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/type-families-and-pokemon)
post, and thought this would be a nice example to attempt to translate into
Scala. I'm fairly certain that the *type families* portion won't be something I
can implement, but this is good practice for Type Classes.

The general goal of this post is to:

 * understand how to write a Type Class
 * investigate their strengths over other implementations

### Basic Design

So `Pokemon` should have a *type* and a *tier*. For instance `Charmander` is a
`Fire` type with `Tier1`. Also, the Pokemon have strengths vs some other types,
but weaknesses against others.

We also want to give users of this library some easy requirements that will
leverage typeclasses. The basic requirements of a consumer should be something
like:
 1. Define a Pokemon Type (eg "air")
 2. Put pokemon in that type, with `Tier`
 3. Provide a typeclass instance of something that tells how they stack up against other types

So one smart way we could implement this is to model the `Eq` type that Haskell
uses. Scala happens to have that exact thing, in the `Ordering` typeclass.

```scala
  trait Hierarchy {
    val order: Int
  }

  case object Tier1 extends Hierarchy {
    override val order = 1
  }

  case object Tier2 extends Hierarchy {
    override val order = 2
  }

  case object Tier3 extends Hierarchy {
    override val order = 3
  }

  implicit val TierOrdering: Ordering[Hierarchy] = new Ordering[Hierarchy] {
    override def compare(x: Hierarchy, y: Hierarchy): Int = x.order - y.order
  }
```

Nice, now let's test that this works.

```scala
it("should correctly compare Tiers") {
      Tier1 > Tier2 shouldBe false
      Tier2 > Tier1 shouldBe true
      Tier1 < Tier2 shouldBe true
      Tier3 == Tier3 shouldBe true
    }
```

```
[info] Hierarchy
[info] - should correctly compare Tiers
[info] Run completed in 334 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
```
