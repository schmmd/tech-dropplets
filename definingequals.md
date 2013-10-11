# Defining equals effectively
- Michael Schmitz
- schmmd
- 2011/11/30
- Scala
- published

I'm amazed how complicated seemingly simple aspects of programming can be.
Everyone who has used Java for a period of time has assuredly created a new
class that overrides equals, and many of those implementations have problems.

Properly overriding equals involves understanding intimidating words like <a
href="http://en.wikipedia.org/wiki/Transitive_relation">transitive</a>, <a
href="http://en.wikipedia.org/wiki/Reflexive_relation">reflexive</a>, and <a
href="http://en.wikipedia.org/wiki/Symmetric_relation">symmetric</a>.  Although
these words can be opaque, it's critical to conform or your class will act
unexpectedly--for example with Java Collections.  Here is an example derived
from Joshua Bloch's <a
href="http://java.sun.com/docs/books/effective/">Effective Java</a>.

    class Foo (val foo: Int) {
      override def equals(that: Any) = {
        that match {
          case that: MyClass => that.foo == this.foo
          case _ => false
        }
      }
    }

This works great if Foo is the only member in it's class hierarchy.  We can
check that it follows all of our constraints.

1. **reflexive:** `foo.equals(foo)`
2. **symmetric:** `foo.equals(bar) => bar.equals(foo)`
3. **transitive:** `foo.equals(bar) && bar.equals(baz) => foo.equals(bar)`

However, things get more complicated with subclassing.  Consider the following
class hierarchy.  There are various types of ovens so we have some subclasses.
When comparing two CobOvens we want to take into account the thickness of the
walls (an added aspect), so we override equals.

    class Oven(val volume: Int) {
      override def equals(that: Any) = {
        that match {
          case that: Oven => that.volume == this.volume
          case _ => false
        }
      }
    }

    // the thickness of the walls conveys the heat retention
    class CobOven(volume: Int, val thickness: Int) extends Oven(volume) {
      override def equals(that: Any) = {
        that match {
          case that: CobOven => that.volume == this.volume && that.thickness == this.thickness
          case _ => false
        }
      }
    }

The problem is evident when we have an `val oven: Oven = new Oven(2)` and a
`CobOven cob: CobOven = new CobOven(2)`.  Our `equals` method is no longer
symmetric.

`oven.equals(cob)` is true, but `cob.equals(oven)` is false

In describing the above situation, Joshua Block mentions, "there is simply no
way to extend an instantiable class and add an aspect while preserving the
equals contract."  In fact, the only reason we don't have this problem with
`Object` is because `Object` has the most specific definition of equals,
refrence equality.  In contrast, most superclasses naturally have more general
definitions of equals, such as in the example above.

Many people maintain symmetry by having a more specific definition of `equals`,
such as the definition provided by <a
href="href="http://commons.apache.org/lang/api-release/org/apache/commons/lang3/builder/EqualsBuilder.html">Apache
EqualsBuilder</a>, a helper class for overriding equals.

    class Oven(val volume: Int) {
      override def equals(that: Object) {
        that match {
          case that: Oven => that.getClass == this.getClass && that.volume == this.volume
          case _ => false
        }
      }
    }

    // the thickness of the walls conveys the heat retention
    class CobOven(volume: Int, val thickness: Int) extends Oven(volume) {
      override def equals(that: Object) {
        that match {
          case that: CobOven => that.getClass == this.getClass && that.volume == this.volume && that.thickness == this.thickness
          case _ => false
        }
      }
    }

However, this prevents any differing classes from being equal, which is not
always desirable.  Consider `java.util.ArrayList` and `java.util.LinkedList`.
If `AbstractList`, a common superclass, implemented the restrictive equals
method above, you can have two lists with the static type `List` that contain
the same elements in the same order but are not equal! This does not seem like
good object oriented design.  Besides, if this were the direction to go, we
should implement `Equals<T>` much like `Comparable<T>` to get the added benefit
of typechecking by the compiler (although this would complicate variance).

These problems stem from Java <a
href="http://www.cs.princeton.edu/courses/archive/fall98/cs441/mainus/node12.html">conflating
subclasses with subtypes</a>.  More precisely, in Java every subclass is a
subtype even though subclassing is a mechanism to reuse code and subtyping
means the subtype must be substitutable in any function that works on a
supertype.  Treating subclassing and subtyping the same works well in most
cases, but causes some confusing situations like the above.

Fortunately, Joshua Bloch's assertion is too heavy-handed.  There is a way to
allow a class hierarchy to safely allow subclasses to choose whether they want
to be compatable with their superclass with respect to equals, but we'll need a
helper method (inspired by <a
href="http://www.artima.com/shop/programming_in_scala">Programming in Scala by
Odersky et al.</a>).

    class Oven(val volume: Int) {
      override def equals(that: Any) = {
        that match {
          case that: Oven => that.canEqual(this) && that.volume == this.volume
          case _ => false
        }
      }
      def canEqual(that: Any) = that.isInstanceOf[Oven]
    }

    // the thickness of the walls conveys the heat retention
    class CobOven(volume: Int, val thickness: Int) extends Oven(volume) {
      override def equals(that: Any) = {
        that match {
          case that: CobOven => that.canEqual(this) && that.volume == this.volume && that.thickness == this.thickness
          case _ => false
        }
      }
      override def canEqual(that: Any) = that.isInstanceOf[CobOven]
    }

    class KitchenOven(volume: Int) extends Oven(volume)
    class BrickOven(volume: Int) extends Oven(volume)

In the above example, we can't compare `Oven` and `CobOven` because the
designer of `CobOven` specified so, but we can compare `Oven`, `KitchenOven`,
and `BrickOven`.
