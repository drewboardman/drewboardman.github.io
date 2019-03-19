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

First we need ADTs for all of the Moves

```scala
trait PokeMove {
  def show: String
}

trait FireMove extends PokeMove
case object Ember extends FireMove {
  override def show = "Vinewhip"
}
case object FlameThrower extends FireMove {
  override def show = "FlameThrower"
}
case object FireBlast extends FireMove {
  override def show = "Fireblast"
}

trait WaterMove extends PokeMove
case object Bubble extends WaterMove {
  override def show = "Bubble"
}
case object WaterGun extends WaterMove {
  override def show = "Watergun"
}

trait GrassMove extends PokeMove
case object VineWhip extends GrassMove {
  override def show = "Vinewhip"
}
```

Ok, easy enough. Now we'll create a basic OOP implementation of the Pokemon.

```scala
trait Pokemon {
  val name: String
  val move: PokeMove
}

case class FirePokemon(override val name: String, override val move: FireMove) extends Pokemon
val charmander = FirePokemon("Charmander", Ember)
val charmeleon = FirePokemon("Charmeleon", FlameThrower)
val charizard = FirePokemon("Charizard", FireBlast)

case class WaterPokemon(override val name: String, override val move: WaterMove) extends Pokemon
val squirtle = WaterPokemon("Squirtle", Bubble)
val wartortle = WaterPokemon("Wartortle", WaterGun)
val blastoise = WaterPokemon("Blastoise", WaterGun)

case class GrassPokemon(override val name: String, override val move: GrassMove) extends Pokemon
val bulbasaur = GrassPokemon("Bulbasaur", VineWhip)
val ivysaur = GrassPokemon("Ivysaur", VineWhip)
val venusaur = GrassPokemon("Venusaur", VineWhip)
```

And finally some function to print a battle

```scala

object Battle {
  def battle(home: Pokemon, away: Pokemon): Unit = {
    println(home.name ++ " casts " ++ home.move.show ++ "\n")
    println(away.name ++ " casts " ++ away.move.show)
  }
}
```

Cool, and when we run `Battle.battle(charizard, venusaur)` we get

```
Charizard casts Fireblast
Venusaur casts Vinewhip
```

to be continued
