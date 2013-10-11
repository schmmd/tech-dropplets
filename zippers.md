# Zippers in Scala
- Michael Schmitz
- schmmd
- 2012/03/21
- Scala
- published

The `List` data type in Scala is handy and immutable, but it has its
shortcomings. The only operation that is fast on a `List` is prepend (or cons
or push), which is O(1). Everything else (append, insert, modify, even init) is
O(n). However, there are many situations where regardless of your data type you
need to perform O(n) operations. In some of these situations, `List`s are still
awkward.

Consider a sequence of numbers `list = List(1, 4, 7, 9, 5, 6, 10)` and the
assignment to write a function `peak(list: List[Int]): Option[Int]` which
returns the first number such that its predecessor and successor are less than
it. With our sample list, `peak(list)` is 9.

At first, you might try to fall back to your imperative programming and
consider using an index into `list`. For example, you could try the following.

    def peak(list: List[Int]): Option[Int] = {
      list.indices.find { index =>
        val x = list(index)
        index > 0 && index < list.size - 1 && 
          x > list(index - 1) && x > list(index + 1)
      }.map(list(_))
    }

Not only is this hard to read between edges cases and switching between
elements of the list and indices of the list, but it's also horribly
inefficient. This algorithm is actually O(n^2) since each iteration of `find`
reindexes into the list.  Using an `Array` would reduce the efficiency back to
O(n), but the code is still ugly.  Consider the following more successful
approach.

    def peak(list: List[Int]): Option[Int] = list match {
      case x :: y :: z :: tl if y > x && y > z => Some(y)
      case x :: tl => peak(tl)
      case Nil => None
    }

This is O(n) and somewhat easier to understand.  However, we run into trouble
when we complicate the problem slightly.  Let's try to write `raisePeak(list:
List[Int]): Option[List[Int]]` which finds the peak, increments it, and returns
the list with the peak incremented.  Now we need a recursive helper function
that keeps track of the beginning of the list.

    def raisePeak(list: List[Int]): Option[List[Int]] = {
      def rec(head: List[Int], tail: List[Int]): Option[List[Int]] = tail match {
        case x :: y :: z :: tl if y > x && y > z => Some((x :: head).reverse ::: ((y + 1) :: z :: tl))
        case x :: tl => rec(x :: head, tl)
        case Nil => None
      }

      rec(List.empty, list)
    }

I think you'll agree that readability went out the window.  The problem is that
we want to phrase the algorithm in terms of a view from within the list (find
the first element with neighbors that are less than it), but all the methods we
have into the `List` consider only a single element.  In the last solution, we
started coding around this restriction by maintaining a history of where we'd
been.  We approached dangerously close to a `Zipper`.

A zipper is quite easily defined.  It consists of a current element, the
elements to the left, and the elements to the right.  In other words, it's a
perspective from inside the list at the focus.

    class Zipper[T](focus: T, left: List[T], right: List[T])

With this definition, it's easy to move left or right in constant time.

    def moveRight: Option[Zipper[T]] = right match {
      case x :: xs => Some(new Zipper[T](x, focus :: left, xs))
      case _ => None
    }

    def moveLeft: Option[Zipper[T]] = left match {
      case x :: xs => Some(new Zipper[T](x, xs, focus :: right))
      case _ => None
    }

And we can always recreate the list at any time, although this operation is O(n).

    def toList = left.reverse ::: (focus :: right)

These are my own definitions, but I'm going to switch to Scalaz's Zipper for an
implementation to raisePeak.  The biggest difference is that the Scalaz Zipper
uses a Stream as its backing--but it could just as easily be a list.  Here is
raisePeak written with a Zipper.

    def raisePeak(list: List[Int]): Option[List[Int]] = {
      for {
        zipper <- list.toZipper
        peak <- zipper.positions.findNext( z => 
          (z.previous, z.next) match {
            case (Some(p), Some(n)) => p.focus < z.focus && n.focus < z.focus
            case _ => false
          })
      } yield (peak.focus.modify(_ + 1).toStream.toList)
    }

Let's disect this.  First note that the Scalaz zipper deals heavily with
`Option` types.  When you convert a `List` to a `Zipper`, you get an
`Option[Zipper]` because the list may have been empty and there's no legal
zipper instantiation for an empty list (we need a focus).  Also, `Zipper` does
not have a find method of type `Zipper[T]=>Boolean`, which we need to check the
next and previous items, but rather `T=>Boolean`.  The way around this is to
call `positions`, which returns a `Zipper[Zipper[T]]`--a zipper of all possible
configurations of a zipper for the list.  With this, we can find the zipper
state we are looking for and modify the focus.

While this code is still complicated, I think it's the most readable solution.
It's also reasonably fast, although it does require two O(n) traversals--one to
find the target and a second to reverse the left elements of the zipper.

Are Zippers the fastest way to solve this problem?  No way.  Are they a neat
data structure allow for easily understood solutions?  Definately!
