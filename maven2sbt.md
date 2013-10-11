# Moving from Maven to SBT
- Michael Schmitz
- schmmd
- 2013/04/29
- Scala
- published

When I first moved into my present position, projects were compiled with
complicated ant scripts and dependencies were checked into our source control.
It quickly became clear that it would be hell to update dependencies and manage
dependency conflicts.  Compounding this problem, I work in a research
environment which means we have many modules and don't really know what
direction we're going in.

Once I discovered maven, I quickly adopted it as our build system.  Even with
maven's verbose XML syntax, build definitions became smaller and easier to
write.  I no longer had to build classpaths or manage where my dependencies lay
on disk.  How could a build system be better?

Later we moved from Java to Scala and I became aware of SBT.  I read
conflicting reviews and SBT seemed complicated.  However, the wild
experimentation calmed down and SBT started looking like a great option at
version 12.x.  We also were interested in crossbuilding our applications for
2.9.2 and 2.10, which had support in SBT.  I also eyed some candy such as an
easy way to run the main class (sbt run) and faster compilation.

After moving to SBT I've found a lot more to like:

1. A project requires less configuration--in fact with no configuration you
still have a project.  You can just `mkdir src/main/scala/`, add a source file,
and compile it with sbt.  Configuration is far easier to remember because there
is no deep, nested XML.  And while some things (like specifying rules when
creating an assembly) look complicated, in actuality the difference is that
there is a sane way to configure operations in SBT.  I have no idea how to
specify custom merging rules in a pom.xml.</li>

2. It's easier to remember the syntax--in part because you don't need to put
elements at the proper vertical position.  SBT has the drawback of less
programatically-modifiable build files, but it's way easier to modify a build
file without resorting to Google searches (how do you exclude a dependency
using Maven?).</li>

3. Multimodule projects are sanely specifiable.  In Maven, you need a parent
pom.xml which specifies common dependencies and a separate pom.xml that builds
the submodules.  In SBT, you can specify common build definitions that get
distributed to the child modules instead of relying on POM inheritance.

I've written a <a href="https://gist.github.com/schmmd/5050790">small script to
help migrate from Maven to SBT</a>.
