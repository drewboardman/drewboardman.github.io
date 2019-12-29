---
layout: post
title:  "Typeclasses in Scala"
date:   2019-03-18 19:52:02 -0400
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

One smart way we could implement this is to model the `Ord` type that Haskell
uses. Scala happens to have pretty much the same thing, in the `Ordering` typeclass.

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

### Creating the Typeclass

First thing to do, is create a basic outline of how we want to fight (taking
advantage of the typeclass model). We know that we are going to need to leave
room for "users" to be able to add their own new types of `Pokemon`, and all
they need to do is supply a typeclass.

```scala
def fight[A <: Pokemon, B <: Pokemon](a: A, b: B)(implicit rulesA: Rules[A], rulesB: Rules[B]) = ???
```

Ok so what should `Rules` look like? This is our typeclass that we're going to
ask users to implement for their new types. While we're at it, we should
probably make a simple ADT that expresses the "*decision*" of the fights.

```scala
trait Decision
case class Winner(a: Pokemon) extends Decision
case object Draw extends Decision
case object NotHandled extends Decision

trait Rules[A] {
  def compare(a1: A, a2: Pokemon): Decision
}
```

We need `NotHandled` here so that our implementations leave room for others to
add new `Rules` and `Pokemon` types in the future. Speaking of pokemon, we need
to make a bunch.

```scala
sealed trait Pokemon {
  val tier: Hierarchy
}

case class FirePokemon(override val tier: Hierarchy) extends Pokemon

val charmander = FirePokemon(Tier1)
val charmeleon = FirePokemon(Tier2)
val charizard = FirePokemon(Tier3)

case class WaterPokemon(override val tier: Hierarchy) extends Pokemon

val squirtle = WaterPokemon(Tier1)
val warTortle = WaterPokemon(Tier2)
val blastoise = WaterPokemon(Tier3)

case class GrassPokemon(override val tier: Hierarchy) extends Pokemon

val bulbasaur = GrassPokemon(Tier1)
val ivysaur = GrassPokemon(Tier2)
val venusaur = GrassPokemon(Tier3)
```

Now we have enough information to implement `Rules`. Here is a first pass at
`FirePokemon`.

```scala
implicit val FireRules: Rules[FirePokemon] = new Rules[FirePokemon] {
  def compare(fire: FirePokemon, opponent: Pokemon): Decision = opponent match {
    case water: WaterPokemon => Winner(water)
    case grass: GrassPokemon => Winner(fire)
    case anotherFire: FirePokemon => Draw
    case _ => NotHandled
  }
}
```

As you can see, we're using the `NotHandled` result for the case where we're
fighting a type we don't know about when we implemented `FireRules`. However you
can see that there is a problem here with this approach. Are aren't taking into
account the tier of the pokemon. A `FirePokemon` is supposed to be a
`GrassPokemon`, but what if the grass pokemon is a higher tier? We need to take
that into account.

We should also take this time to implement this same logic into `Draw` (when
they are the same type `Pokemon` but potentially different tiers).

```scala
implicit class RichTieredPokemon(a: Pokemon) {
    import TierOrdering._
    def considerTiers(b: Pokemon): Pokemon = if (a.tier < b.tier) b else a
}

private def handleSameType(same1: Pokemon, same2: Pokemon) =
  if (same1.tier == same2.tier)
    Draw
  else
    Winner(same2.considerTiers(same1))
```

The reason this is in an `implicit class` is so that we can use nice dot
notation. It's also nice because with get to finally use the `Ordering`
typeclass we implemented at the beginning.

Taking this new function, we can improve our `FireRules`. While we're at it, we
can implement the others as well.

```scala
implicit val FireRules: Rules[FirePokemon] = new Rules[FirePokemon] {
  def compare(fire: FirePokemon, opponent: Pokemon): Decision = opponent match {
    case water: WaterPokemon => Winner(water.considerTiers(fire))
    case grass: GrassPokemon => Winner(fire.considerTiers(grass))
    case anotherFire: FirePokemon => handleSameType(fire, anotherFire)
    case _ => NotHandled
  }
}

  implicit val WaterRules: Rules[WaterPokemon] = new Rules[WaterPokemon] {
  def compare(water: WaterPokemon, opponent: Pokemon): Decision = opponent match {
    case fire: FirePokemon => Winner(water.considerTiers(fire))
    case grass: GrassPokemon => Winner(grass.considerTiers(water))
    case anotherWater: WaterPokemon => handleSameType(water, anotherWater)
    case _ => NotHandled
  }
}

implicit val GrassRules: Rules[GrassPokemon] = new Rules[GrassPokemon] {
  def compare(grass: GrassPokemon, opponent: Pokemon): Decision = opponent match {
    case fire: FirePokemon => Winner(fire.considerTiers(grass))
    case water: WaterPokemon => Winner(grass.considerTiers(water))
    case anotherGrass: GrassPokemon => handleSameType(grass, anotherGrass)
    case _ => NotHandled
  }
}
```

Awesome. Now we have all the things we need to finally implement `fight`.

```scala
def fight[A <: Pokemon, B <: Pokemon](a: A, b: B)(implicit rulesA: Rules[A], rulesB: Rules[B]): Decision = {
  val originalResult = rulesA.compare(a, b)
  originalResult match {
    case NotHandled => rulesB.compare(b, a)
    case _ => originalResult
  }
}
```

First we check to see if the first pokemon has all of the information it needs
in its `Rules`, which is true in the case of the original fire, grass, and
water. If there is a new type that was added, we need to check the rules for
that new type to see if they've correctly implemented their element into the
strange pokemon taxonomy.

Let's be sure that this works correctly.

```scala
describe("PokemonBattle") {
  describe("Fire") {
    it("is a draw with same tiered Fire") {
      val newFireTier1 = FirePokemon(Tier1)
      PokemonBattle.fight(Pokemodels.charmander, newFireTier1) shouldBe Draw
    }

    it("loses to water") {
      val fight = PokemonBattle.fight(Pokemodels.charizard, Pokemodels.blastoise)
      val fightReverse = PokemonBattle.fight(Pokemodels.blastoise, Pokemodels.charizard)
      fight shouldBe Winner(Pokemodels.blastoise)
      fightReverse shouldBe Winner(Pokemodels.blastoise)
    }

    it("wins vs grass") {
      val fight = PokemonBattle.fight(Pokemodels.charizard, Pokemodels.venusaur)
      val fightReverse = PokemonBattle.fight(Pokemodels.venusaur, Pokemodels.charizard)
      fight shouldBe Winner(Pokemodels.charizard)
      fightReverse shouldBe Winner(Pokemodels.charizard)
    }

    it("considers tier correctly") {
      val grassTier3 = GrassPokemon(Tier3)
      val fight = PokemonBattle.fight(Pokemodels.charmeleon, grassTier3)
      val fightReverse = PokemonBattle.fight(grassTier3, Pokemodels.charmeleon)

      fight shouldBe Winner(grassTier3)
      fightReverse shouldBe Winner(grassTier3)
    }
  }
}
```

Running the tests with `sbt`. We see they all pass.

```
[info] PokemonTypeClassesTest:
[info] Hierarchy
[info] - should correctly compare Tiers
[info] PokemonBattle
[info]   Fire
[info]   - is a draw with same tiered Fire
[info]   - loses to water
[info]   - wins vs grass
[info]   - considers tier correctly
[info] Run completed in 129 milliseconds.
[info] Total number of tests run: 5
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 5, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
```

### Adding a new Pokemon

If we've done everything as planned, we should be able to add a new pokemon
type and fight it against the original 3 types. Let's go ahead and implement
`Air` type Pokemon. Since I have no idea how pokemon actually work, I'm just
going to make stuff up for the taxonomy. Let's just say it only wins against
water, because why not?

```scala
case class AirPokemon(override val tier: Hierarchy) extends Pokemon

val thinger = AirPokemon(Tier1)
val bigChungus = AirPokemon(Tier2)
val oleFarty = AirPokemon(Tier3)

implicit val AirRules: Rules[AirPokemon] = new Rules[AirPokemon] {
  def compare(air: AirPokemon, opponent: Pokemon): Decision = opponent match {
    case fire: FirePokemon => Winner(fire.considerTiers(air))
    case water: WaterPokemon => Winner(air.considerTiers(water))
    case grass: GrassPokemon => Winner(grass.considerTiers(air))
    case anotherAir: AirPokemon => handleSameType(air, anotherAir)
    case _ => NotHandled
  }
}

# test it!

describe("Air") {
  it("wins against water") {
    val fight = PokemonBattle.fight(Pokemodels.oleFarty, Pokemodels.blastoise)
    val fightReverse = PokemonBattle.fight(Pokemodels.blastoise, Pokemodels.oleFarty)
    fight shouldBe Winner(Pokemodels.oleFarty)
    fightReverse shouldBe Winner(Pokemodels.oleFarty)
  }

  it("loses to fire") {
    val fight = PokemonBattle.fight(Pokemodels.bigChungus, Pokemodels.charmeleon)
    val fightReverse = PokemonBattle.fight(Pokemodels.charmeleon, Pokemodels.bigChungus)
    fight shouldBe Winner(Pokemodels.charmeleon)
    fightReverse shouldBe Winner(Pokemodels.charmeleon)
  }
}
```

They pass!

```
[info] PokemonTypeClassesTest:
[info] Hierarchy
[info] - should correctly compare Tiers
[info] PokemonBattle
[info]   Fire
[info]   - is a draw with same tiered Fire
[info]   - loses to water
[info]   - wins vs grass
[info]   - considers tier correctly
[info]   Air
[info]   - wins against water
[info]   - loses to fire
[info] Run completed in 132 milliseconds.
[info] Total number of tests run: 7
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 7, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
```

### Take aways

This is a really nice illustration of why typeclasses work well for providing
*extendable* libraries. We could give the original grass/water/fire
implementation of the library to anyone. As long as they provide an instance of
the `Rules` typeclass, everything works just like it did with the original.
There is no need to change the library at all!
